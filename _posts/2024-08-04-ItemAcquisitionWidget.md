---
layout: post
title: "아이템 획득처 구현"
name: movie
link: https://github.com/movie-dev
date: 2024-08-04 13:00:00 +0900
categories: [Portfolio]
tags: [UnrealEngine]
mermaid: true
---
## 로스트아크 아이템 획득처 (예시)
![StatePattern](/assets/img/itemacquisition_lostark.png){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }

## 구현 방식 소개
---
![StatePattern](/assets/img/state_pattern.png){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }

아이템 획득처에 표시되어야 하는 각각의 획득처들을 하나의 상태라고 구분하여 부모 클래스를 만들고 자식 클래스를 추가하는 방식으로 구현 하였습니다. 인터페이스 단위로 각 획득처의 동작이 다른 부분은 없어서 무사히 끝낼 수 있었습니다.
```c++
if (Monster)
{
	...
}
else if (Craft)
{
	...
}
```
개발 당시에 패턴에 대한건 생각하지 못했는데 구현 하면서 위와 같은 로직은 꼭 피해야겠다고 생각하다 보니 이렇게 구현이 된 것 같습니다.
	

## 구현 방식
---
1. [아이템 획득처 데이터 추출](https://movie-dev.github.io/posts/ItemAcquisition/){:target="_blank"} 에서 추출된 파일의 이름을 ItemAcquisition::ItemAcquisitionPrefix + TableName 로 설정 했습니다. 위젯 구현부의 코드도 이 규칙을 따라야 코드 수정이 없기 때문에 이름 기반으로 특정 테이블이 있는지 검사하고 Array에 저장하게 됩니다. 모든 추출 테이블은 FItemAcquisitionBase 구조체를 상속받도록 설계 했습니다. 추후에 설명할 가상함수를 사용하기 위함입니다.
	```c++
	USTRUCT()
	struct FItemAcquisitionBase : public FBaseDataStruct
	{
		GENERATED_USTRUCT_BODY()

		virtual ItemAcquisitionItemsType MakeItemAcquisitionItems() const PURE_VIRTUAL(FItemAcquisitionBase::MakeItemAcquisitionItems, return {};);
		virtual void CreateItemAcquisitionItemEntries(class UTileView* InTileView, ItemAcquisitionItemsType& InData) PURE_VIRTUAL(FItemAcquisitionBase::CreateItemAcquisitionItemEntries, return;);

		template <typename T>
		void CreateItemAcquisitionItemInternal(class UTileView* InTileView, const ItemAcquisitionItemsType& InData)
		{
			for (const auto& Data : InData)
			{
				T* Entry = NewObject<T>();
				Entry->SetAlias(Data);

				InTileView->AddItem(Entry);
			}
		}
	};  
	
	...

	TArray<FItemAcquisitionBase*> Items;
	for (const auto& TableName : *TableNames)
	{
		FItemAcquisitionBase* Data = StaticCast<FItemAcquisitionBase*>(UDataTableManager::Get()->GetDataRawPtr(*(ItemAcquisition::ItemAcquisitionPrefix + TableName), SelectItem->ItemAlias.ToString()));
		if (!Data)
		{
			continue;
		}
		Items.Emplace(Data);
	}
	```

1. 그 후 Array를 순회하면서 위젯에 전달할 데이터를 만들어 위젯에 전달합니다.
	```c++
	for (auto* Item : Items)
	{
		...
		
		ItemAcquisitionItemsType ItemEntries = Item->MakeItemAcquisitionItems();
		if (Item->IsEntryShortVersion())
		{
			Item->CreateItemAcquisitionShortItemEntries(TV_ItemAcquisition, ItemEntries);
		}
		else
		{
			Item->CreateItemAcquisitionItemEntries(TV_ItemAcquisition, ItemEntries);
		}
	}
	```
	
1. MakeItemAcquisitionItems 가상함수와 CreateItemAcquisitionItemEntries 가상함수를 설명하겠습니다.
	2. MakeItemAcquisitionItems 함수는 추출된 테이블을 순회하면서 리스트뷰에 전달할 데이터를 만듭니다. 이 과정에서 정렬 또는 걸러야 할 데이터들을 처리합니다. 예를 들면 아래 코드와 같습니다.
		```c++
		ItemAcquisitionItemsType FItemAcquisitionMonster::MakeItemAcquisitionItems() const
		{
			ItemAcquisitionItemsType Result;

			for (const auto& Data : Datas)
			{
				auto DataSharedPtr = MakeShared<xxxxxx>(Data);

				...
				
				Result.Emplace(MoveTemp(DataSharedPtr));
			}

			FItemAcquisitionSorter_Monster().Sort(Result);
			
			return Result;
		}
		```
	2. CreateItemAcquisitionItemEntries 함수는 MakeItemAcquisitionItems 함수에서 처리된 데이터를 기반으로 UObject 객체를 만들어 타일 뷰로 전달합니다. 이 부분까지 코드 제너레이터를 이용하여 자동화 시킬 수도 있을 것 같은데 조금씩 예외 상황이 있어서 구현하지는 못했습니다. 파이썬이나 C#을 좀 더 공부해서 해보는 것도 좋아 보입니다.
		```c++
		void FItemAcquisitionMonster::CreateItemAcquisitionItemEntries(class UTileView* InTileView, ItemAcquisitionItemsType& InData)
		{
			CreateItemAcquisitionItemInternal<UItemAcquisitionEntry_Monster>(InTileView, InData);
		}
		```
		
1. 타일 뷰에 전달되는 데이터는 UItemAcquisitionEntryBase를 상속받게 됩니다. 이 부모 클래스는 하위 자식들에게 인터페이스를 제공하며 자식들은 인터페이스를 구현하여 각 상황에 맞는 위젯 동작을 하도록 구현합니다. GetEntryType(), OnUpdateIconSlot(), IsGradeEnable(), GetGradeType(), GetTitleType(), OnUpdateTitle() 함수 등이 인터페이스 입니다.
	```c++
	void UItemAcquisitionEntryBase::Refresh(UItemAcquisitionListItemWidget* InItemWidget)
	{
		InItemWidget->WS_Item->SetActiveWidgetIndex(StaticCast<int32>(GetEntryType()));

		InItemWidget->Btn_Shortcut->SetVisibility(ESlateVisibility::Visible);
		OnUpdateIconSlot(InItemWidget);
		InItemWidget->WS_Type->SetVisibility(IsGradeEnable() ? ESlateVisibility::HitTestInvisible : ESlateVisibility::Collapsed);
		if (IsGradeEnable())
		{
			InItemWidget->WS_Type->SetActiveWidgetIndex(StaticCast<int32>(GetGradeType()));
		}
		InItemWidget->WS_Title->SetActiveWidgetIndex(StaticCast<int32>(GetTitleType()));

		OnUpdateTitle(InItemWidget);

		InItemWidget->VB_MainCondition->SetVisibility(IsDescriptionEnable() ? ESlateVisibility::HitTestInvisible : ESlateVisibility::Collapsed);
		if (IsDescriptionEnable())
		{
			InItemWidget->WS_MainCondition->SetActiveWidgetIndex(StaticCast<int32>(GetDescriptionType()));
			OnUpdateDescription(InItemWidget);
		}

		InItemWidget->CP_SubCondition->SetVisibility(IsSubConditionEnable() ? ESlateVisibility::HitTestInvisible : ESlateVisibility::Collapsed);
		if (IsSubConditionEnable())
		{
			InItemWidget->WS_SubCondition->SetActiveWidgetIndex(StaticCast<int32>(GetSubConditionType()));
			OnUpdateSubCondition(InItemWidget);
		}

		UpdateEntryStatus(InItemWidget);
	}
	```