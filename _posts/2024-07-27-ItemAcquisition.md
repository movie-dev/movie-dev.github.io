---
layout: post
title: "아이템 획득처 데이터 추출"
name: movie
link: https://github.com/movie-dev
date: 2024-07-27 16:07:00 +0900
categories: [Portfolio]
tags: [UnrealEngine]
mermaid: true
---

## 개발 동기
---
수많은 테이블들 중 필요한 데이터만을 추출 할 수 있는 툴을 제공하여, 성능 저하를 없애고 코드 수정 없이 데이터 수정만으로 목적을 달성할 수 있도록 하기 위해 개발하게 되었습니다.

특정 아이템의 획득처를 가져오는 것은 많은 테이블들을 검색해야 할 수 있습니다. 

> Land
> > Territory
> >>Monster
> >>>MonsterReward
>>>>>Item

* 예를 들어 위와 같은 방식으로 5개의 테이블을 거쳐서 아이템을 얻어낼 수 있지만, 아이템을 기준으로 모든 획득처를 표시해야 하므로 런타임에 검색하는 것은 성능 저하를 일으킬 수 있습니다. 
* 또한 아이템 획득처를 위해 새로운 테이블을 만들어 데이터를 직접 입력할 수도 있지만 관리해야 할 테이블이 늘어나고 데이터가 변경되었을 때도 2개의 테이블 (Reward 수정 시 새로 추가한 테이블도 함께 수정) 을 수정해야 하므로 실수가 생길 여지가 많고 불편하고 귀찮아집니다.

위와 같은 이유로 기존에 사용하던 테이블을 기반으로 새로운 테이블을 프로그램으로 추출하기로 결정 하였습니다.

## 추출 테이블 구조
 ---
 특정 Land에서 나오는 아이템 획득처를 추출 한다고 했을 때 아래와 같이 작성을 하게 됩니다.

### ExtractTable

|Category|Table List                   |
|--------|-----------------------------|
|Land    |Land,Territory,Monster,MonsterReward|

<br>
그 후에 각 테이블에서 참조할 컬럼의 이름을 작성합니다.

### ColumnTable

|Table Name   |Column List    |
|-------------|---------------|
|Land         |TerritoryId    |
|Territory    |MonsterId      |
|Monster      |MonsterRewardId,MonsterRandomRewardId|
|MonsterReward|ItemId         |


## 개발 내용
 ---
Table List를 ","값으로 Parse하면 테이블의 Array를 만들 수 있는데, Array를 순회하면서 ColumnTable을 참조하면 마지막 ItemId까지 얻을 수 있는 규칙이 만들어집니다. 
구현된 내용으로 순서대로 설명 하겠습니다.

### 데이터 파싱

1. Table List를 Parse 합니다.
	```c++
	for (TDataPtr<FExtractTable> Data : TableDataIterator<FExtractTable>())
	{
		TArray<FString> ParseTableList;
		Data->TableList.ParseIntoArray(ParseTableList, TEXT(","));
		if (ParseTableList.IsEmpty())
		{
			continue;
		}
		
		...
	}
	```

1. 첫번째 테이블의 이름으로 ColumnTable을 가져옵니다. 그 후 Column List도 마찬가지로 "," 로 Parse 합니다.
	```c++
	const FString StructName = ParseTableList[0];
	ParseTableList.Remove(StructName);
	
	TDataPtr<FColumnTable> ColumnTablePtr = UDataTableManager::Get()->GetData<FColumnTable>(StructName);
	if (!ColumnTablePtr)
	{
		continue;
	}

	TArray<FString> ParseColumnList;
	ColumnTablePtr->ColumnList.ParseIntoArray(ParseColumnList, TEXT(","));
	if (ParseColumnList.IsEmpty())
	{
		continue;
	}
	
	...
	```

1. 첫번째 테이블의 이름으로 ScriptStruct를 검색합니다. 실제로 테이블이 존재하는지 검사함과 동시에 언리얼엔진에서 제공하는 리플렉션에 사용됩니다. 리플렉션을 사용하여 특정 테이블의 컬럼 값을 가져올 수 있습니다.
```c++
	const UScriptStruct* CurScriptStruct = XmlHelper::GetScriptStruct(StructName);
	if (!CurScriptStruct)
	{
		continue;
	}
```
	{% include note.html content="<br/>**XmlHelper::GetScriptStruct**<br/>모든 ScriptStruct 클래스를 순회하면서 인자로 넘어온 StructName과 비교하여 값을 반환합니다." %}

1. 첫번째 (예제의 상황이라면 Land) 테이블 전체를 가져와 순회하면서 리플렉션에 필요한 데이터를 생성합니다. void* 형의 uint8* 바이너리 데이터가 필요하여 메모리를 할당 해주었습니다.
```c++
	const auto* Table = UDataTableManager::Get()->GetDataTable(*StructName);
	if (!Table)
	{
		continue;
	}
	
	for (const auto& Pair : *Table)
	{
		const TSharedPtr<const FBaseDataStruct> TableValue = Pair.Value;

		void* TableMemAllocPtr = reinterpret_cast<uint8*>(FMemory::Malloc(CurScriptStruct->GetStructureSize()));
		
		CurScriptStruct->InitializeStruct(TableMemAllocPtr);
		CurScriptStruct->CopyScriptStruct(TableMemAllocPtr, TableValue.Get());
		
		...
	}
```

1. 2번에서 Parse 했던 Column List 컬럼들을 순회하면서 각 컬럼의 값이 있는지 검사한 후 저장합니다. (예제의 상황이라면 Land 테이블 전체를 순회하면서 TerritoryId 컬럼을 검사합니다.) 컬럼 프로퍼티가 Array로 되어있는 경우가 있기 때문에 체크해야 합니다. Array가 아닐 경우에는 ArrayDim 값이 1로 반환됩니다.
```c++
	TSet<FString> ColumnIds;
	
	for (auto j = 0; j < ParseColumnList.Num(); ++j)
	{
		const int32 ArrayDim = XmlHelper::GetScriptStructPropertyArrayDim(CurScriptStruct, ParseColumnList[j]);
		for (auto k = 0; k < ArrayDim; ++k)
		{
		   FString ScriptStructPropertyValue = XmlHelper::GetScriptStructPropertyValue(TableMemAllocPtr, CurScriptStruct, ParseColumnList[j], k);
		   if (ScriptStructPropertyValue.IsEmpty())
		   {
			  continue;
		   }
		   ColumnIds.Emplace(MoveTemp(ScriptStructPropertyValue));
		}
	}
```

	{% include note.html content="<br/>**XmlHelper::GetScriptStructPropertyArrayDim**<br/>ScriptStruct 클래스 내부에는 Field 값의 Array 길이 값을 제공합니다. 그 값을 가져오는 로직을 헬퍼함수에 추가 하였습니다." %}
	{% include note.html content="<br/>**XmlHelper::GetScriptStructPropertyValue**<br/>FProperty 구조체를 상속 받는 FUInt32Property, FNumericProperty, FEnumProperty, FStrProperty, FBoolProperty 타입 캐스팅을 검사하여 값을 추출합니다." %}
	
1. 이제 나머지 테이블들을 순회해야 합니다. (첫번째 테이블 (예제의 경우 Land)을 따로 처리해준 것은 개발스펙에 추출되면 안되는 데이터들이 있어서 어쩔 수 없이 분리 하였습니다.) 이제는 예외가 없으므로 재귀함수를 통해 처리 하였습니다.
```c++
	if (!ColumnIds.IsEmpty())
	{
		for (auto i = 0; i < ParseTableList.Num(); ++i)
		{
		   ColumnIds = RecursiveExtractTable(ParseTableList[i], ColumnIds);
		}
	}
	
	...
```
	2. RecursiveExtractTable 함수는 첫번째 테이블을 Parse 했던 것과 방식이 동일합니다. XmlHelper::GetScriptColumnValues 함수에 그 과정을 모아 두었습니다.
		```c
			TSet<FString> Ret;

			TDataPtr<FColumnTable> ColumnTablePtr = UDataTableManager::Get()->GetData<FColumnTable>(InTableName);
			if (!ColumnTablePtr)
			{
				continue;
			}

			TArray<FString> ParseColumnList;
			ColumnTablePtr->ColumnList.ParseIntoArray(ParseColumnList, TEXT(","));
			if (ParseColumnList.IsEmpty())
			{
				continue;
			}

			return XmlHelper::GetScriptColumnValues(InTableName, InColumnIds, ParseColumnList);
		```
		
1. 마지막 테이블까지 순회가 완료되면 ColumnIds 컨테이너에 ItemId가 저장됩니다. 저의 경우에는 ItemId 안에 현재 테이블의 Id가 필요했기 때문에 Map을 사용 하였습니다. 그 후 데이터를 추출합니다.
```c++
	TMap<int32, TSet<int32>> Result;
	
	if (!ColumnIds.IsEmpty())
	{
		const int32 CurTableId = XmlHelper::GetScriptStructPropertyValue(TableMemAllocPtr, CurScriptStruct, ItemAcquisition::UID, 0);
		
		for (const auto& Id : ColumnIds)
		{
			Result.FindOrAdd(Id).Emplace(CurTableId);
		}
	}
	
	FMemory::Free(TableMemAllocPtr);
	
	...
	
	WriteXmlWrapperFile(ItemAcquisition::ItemAcquisitionPrefix, Data->TableName, Result);
```

### 데이터 추출

1. 추출해야 되는 테이블마다 데이터 구조가 조금 다른 부분이 있어서 분기를 태웠습니다. 로직은 거의 같아서 일반적인 경우만 기술 하겠습니다. 인자로 추출할 파일 앞에 붙을 Prefix와 파일명, 그리고 위에서 추출했던 Result가 넘어옵니다.
```c++
template <typename T>
void WriteXmlWrapperFile(const FString& InFilenamePrefix, const FString& InFilename, const T& InTable)
{
	if (InFilename == TEXT("xxx") || InFilename == TEXT("xxxx"))
	{
		WriteXmlWrapperFileImpl(InFilenamePrefix, InFilename, PostProcessXmlWrapperFile_xxxx(InTable));
	}
	else if (InFilename == TEXT("aaa"))
	{
		WriteXmlWrapperFileImpl(InFilenamePrefix, InFilename, PostProcessXmlWrapperFile_aaa(InTable));
	}
	else
	{
		WriteXmlWrapperFileImpl(InFilenamePrefix, InFilename, InTable);
	}
}
```

1. WriteXmlWrapperFileImpl 함수에는 Xml의 테이블 형식과 데이터를 그 형식에 맞춰 Write 해주고 있습니다. 형상관리 툴로 퍼포스를 사용하고 있었기 때문에 변하지 않은 파일은 되돌려 Revert 시켜 주었습니다.

	```c++
	template <typename T, typename U>
	void WriteXmlWrapperFileImpl(const FString& InFilenamePrefix, const FString& InFilename, const ExportTableDataTypeTemplate<T, U>& InTable)
	{
	#if WITH_EDITOR
		EditorUtilityGlobalFunction::CheckOutFile(XmlHelper::GetContentXmlFilename(InFilenamePrefix + InFilename));
	#endif

		const FString FileTemplate = TEXT("<?xml version=\"1.0\" endcoding =\"UTF-8\"?>\n<List>\n</List>");
		FXmlFile* File = new FXmlFile(FileTemplate, EConstructMethod::ConstructFromBuffer);

		FXmlNode* RootNode = File->GetRootNode();
		RootNode->AppendChildNode(TEXT("TableType"), TEXT("HorizontalTable"));
		RootNode->AppendChildNode(TEXT("KeyType"), TEXT("SingleKey"));

		for (const auto& Pair : InTable)
		{
			const auto& Key = Pair.Key;
			RootNode->AppendChildNode(TEXT("UStruct"), TEXT(""));
			FXmlNode* ChildNode = RootNode->GetChildrenNodes()[RootNode->GetChildrenNodes().Num() - 1];

			for (TFieldIterator<FProperty>It(T::StaticStruct()); It; ++It)
			{
				FProperty* CurProperty = *It;
				XmlHelper::AppenChildNode(&Key, ChildNode, CurProperty, 0);
			}

			FXmlNode* ArrayNode = RootNode->GetChildrenNodes()[RootNode->GetChildrenNodes().Num() - 1];
			ArrayNode->AppendChildNode(InFilename + ItemAcquisition::ID, TEXT(""));
			FXmlNode* ArrayChildNode = ArrayNode->GetChildrenNodes()[ArrayNode->GetChildrenNodes().Num() - 1];

			for (const auto& Item : Pair.Value)
			{
				ArrayChildNode->AppendChildNode(TEXT("UStruct"), TEXT(""));
				FXmlNode* ItemNode = ArrayChildNode->GetChildrenNodes()[ArrayChildNode->GetChildrenNodes().Num() - 1];
				for (TFieldIterator<FProperty>It(U::StaticStruct()); It; ++It)
				{
					FProperty* CurProperty = *It;
					XmlHelper::AppenChildNode(&Item, ItemNode, CurProperty, 0);
				}
			}
		}

		File->Save(XmlHelper::GetContentXmlFilename(InFilenamePrefix + InFilename));

	#if WITH_EDITOR
		EditorUtilityGlobalFunction::RevertUnchangedFiles(XmlHelper::GetContentXmlFilename(InFilenamePrefix + InFilename));
	#endif
	}
	```
	
## 기타
---
### Xml vs Binary
1. 프로젝트에서 테이블 포맷을 Xml로 사용하고 있는데 글자가 많아질수록 용량이 늘어나는 현상을 발견 했습니다. 이를 개선하고자 아이템 획득처 추출 테이블에서는 Binary 형태로도 추출이 가능하도록 구현 하였습니다.
```c++
USTRUCT()
struct FItemAcquisitionMonster : public FItemAcquisitionBase
{
   GENERATED_USTRUCT_BODY()

   ...

   friend uint32 GetTypeHash(FItemAcquisitionMonster const& Key)
   {
      return FCrc::StrCrc32(*FString::Printf(TEXT("%d_%s"), xxxx, xxxxx));
   }
   bool operator== (FItemAcquisitionMonster const& Key) const
   {
      return xxxx == Key.xxxx
         && xxxxx == Key.xxxxx;
   }
   virtual void Serialize(FArchive& Ar) override
   {
      Ar << *this;
   }
   friend FArchive& operator<<(FArchive& Ar, FItemAcquisitionMonster& Data)
   {
      Ar << Data.xxxx;
      Ar << Data.xxxxx;

      return Ar;
   }
```

1. 언리얼 엔진에서 제공하는 Serialize를 사용하여 Xml 대비 약 15배 적은 용량으로 동일한 기능을 구현할 수 있었습니다. 하지만 기획자분들이 사용하는 툴 등이 전부 Xml 기반으로 되어있었기 때문에 사용되지는 않았습니다.

### 커맨드렛
1. 데이터가 바뀔 때마다 매번 데이터 추출 버튼을 누르는 것은 귀찮을 뿐더러 깜빡할 수 있다고 생각 했습니다. 그래서 언리얼 엔진에서 제공하는 커맨드렛을 사용하여 윈도우빌드 시에 함께 동작할 수 있도록 구현 하였습니다.
1. 이것도 사용되지는 않고 있습니다.