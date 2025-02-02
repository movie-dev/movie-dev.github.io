---
layout: post
title: "스마일게이트 사내 게임 공모전"
name: movie
link: https://github.com/movie-dev
date: 2025-02-02 20:10:00 +0900
categories: [PersonalProject]
tags: [UnrealEngine]
mermaid: true
---
## 개요
---
최근 게을러진 마음을 다 잡고자 사내 공모전에 참여 하였습니다. 회사 일과 병행해야 하기 때문에 빠듯하게 프로젝트를 진행 했습니다.  

규모가 크지 않은 게임 이었지만, 혼자 게임을 만든다는 것은 정말 어려웠습니다. 특히 아트 리소스는 마음에 드는 것을 찾기도 어려웠고 찾아서 게임에 배치하고 관리하는 것이 제일 오래 걸렸는데 아트 디자이너 분들이 정말 대단하다고 다시 한번 느끼게 됐던 계기가 됐습니다.  

이번에 프로젝트를 진행하면서 개발했던 내용들을 정리 해보려고 합니다.

## 게임 소개
---
![Platformer](/assets/img/Platformer.png){:style="border:0px solid #eaeaea; border-radius: 7px; padding: 0px;" }

<iframe
  class="embed-video youtube lazyload"
  src="https://www.youtube.com/embed/fgmw_lbyDMI?si=y6DmqwXfhtZaeTrN"
  title="YouTube video player"
  frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
  allowfullscreen
></iframe>

> 게임 장르 : 3D 플랫포머 달리기 게임  
> 사용 엔진 : 언리얼엔진 5.4.4  
> 제작 기간 : 2024-11-01 ~ 2025-01-31 (3개월)  
> 게임 방법 : 장애물들을 피해 목적지까지 도착하는 게임입니다.

## 개발 내용
---
### 로비
---
![Platformer_Lobby](/assets/img/Platformer_Lobby.png){:style="border:0px solid #eaeaea; border-radius: 7px; padding: 0px;" }

![Platformer_LobbyWidget](/assets/img/Platformer_LobbyWidget.png){:style="border:0px solid #eaeaea; border-radius: 7px; padding: 0px;" }

#### 레벨
---
이번에 제작한 게임에서는 월드 파티션을 사용하지 않고 레벨을 분리 하였습니다. 아직 월드 파티션에 대한 개념과 동작 원리를 완벽하게 이해하지 못하기도 했고, 작업 속도를 내기 위해 이런 결정을 내렸습니다.

플레이어 캐릭터도 레벨에 미리 배치해뒀기 때문에 위젯에 대한 부분만 설명하도록 하겠습니다.

#### 위젯
---
![gameframework](/assets/img/gameframework.png){:style="border:0px solid #eaeaea; border-radius: 7px; padding: 0px;" }

( 참고: [게임플레이 프레임워크](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/gameplay-framework-in-unreal-engine) )
{: style="color:gray; text-align: center;"}  


1. 레벨 로드가 완료되고 UWorld::BeginPlay 함수가 호출되면 위 이미지와 같은 사이클로 게임 진행에 필요한 액터들이 생성 됩니다. 뷰포트에 붙어있는 위젯들은 별다른 옵션을 건드리지 않는다면 레벨이 내려갈 때 뷰포트에서 떨어지게 됩니다.<br><br> 그래서 레벨이 로드가 된다면 뷰포트에 붙일 수 있는데, 저 같은 경우는 HUD 클래스가 BeginPlay 될 때 위젯의 Root가 되는 위젯을 생성 했습니다. Root 위젯은 Root 위젯 내부에서 ZOrder를 관리하려고 생성 했습니다.  
![rootwidget](/assets/img/rootwidget.png){:style="border:0px solid #eaeaea; border-radius: 7px; padding: 0px;" }  


1. 현재는 게임 내에 오직 하나만 있어야 하는 위젯에 대해서만 레이어를 관리하며 그 위젯들은 위젯 테이블 내에서 관리합니다. `(Key - C++ 위젯 클래스, Value - 위젯 블루프린트)`
![widgetconfig](/assets/img/widgetconfig.png){:style="border:0px solid #eaeaea; border-radius: 7px; padding: 0px;" }

1. 이렇게 되면 아래 코드와 같이 C++ 클래스를 템플릿 인자로 넘겨 코드 작성이 가능해져서 편리하게 사용할 수 있습니다.
```c++
	UDiaryWidgetSubsystem::Get(this)->OpenWidget<UDiaryInteractionWidget>(...);
```

1. 위와 같은 위젯을 ManagedWidget이라 명칭하였고, ManagedWidget 클래스에서 필요한 인터페이스를 제공하여 원하는 레이어에 붙을 수 있게 코드를 작성 하였습니다.
```c++
	UCLASS(Abstract)
	class MAOCORE_API UMaoManagedWidget : public UMaoWidget
	{
		GENERATED_UCLASS_BODY()

	public:
		friend class UMaoWidgetSubsystem;

		void OpenWidget();
		
		virtual EMaoWidgetLayer GetWidgetLayer() const;
		virtual bool HideOtherManagedWidget() const;
		virtual bool CloseOtherManagedWidgetOpen() const;
	}
```

1. ManagedWidget들을 관리하는 WidgetSubSystem을 만들었고, 모든 위젯을 이 클래스에서 관리합니다. 위젯 생성과 동시에 특정 파라미터를 넘겨야 되는 경우가 많아서 가변인자 템플릿 문법을 사용 했습니다. 위젯 생성에는 비동기 로드 버전과 동기 로드 버전 2가지 모두 작성 했습니다.
```c++
	UCLASS(Abstract)
	class MAOCORE_API UMaoWidgetSubsystem : public UMaoGameInstanceSubsystem
	{
		GENERATED_BODY()

	public:
		friend class UMaoManagedWidget;
		
		template <typename T = UMaoManagedWidget, typename... Args>
		void OpenWidget(Args&&... InArgs)
		{
			const auto* WidgetConfig = UMaoAssetManager::Get().GetMaoPrimaryAssetData<UMaoWidgetConfig>();
			MAO_ENSURE_IF_NOT_RET(WidgetConfig, "WidgetConfig is not Exist.");

			if (const auto* WidgetPath = WidgetConfig->ManagedWidgets.Find(T::StaticClass()))
			{
				TWeakObjectPtr<UMaoWidgetSubsystem> WeakThis(this);

				using FuncType = TFunction<void(UObject*)>;
				FuncType CompleteFunction = [WeakThis, WidgetPath, ...CaptureArgs = Forward<Args>(InArgs)](UObject* InObject) mutable
				{
					if (!WeakThis.IsValid())
					{
						return;
					}
					MAO_ENSURE_FORMATTED_IF_NOT_RET(InObject, "OpenWidget Error [{0}]", FText::FromString(WidgetPath->ToSoftObjectPath().ToString()));

					if (UMaoManagedWidget** OpenedWidget = WeakThis->WidgetMap.Find(T::StaticClass()))
					{
						Cast<T>((*OpenedWidget))->OpenWidget(Forward<Args>(CaptureArgs)...);
						return;
					}
					
					auto* Widget = CreateWidget<T>(WeakThis->GetGameInstance(), CastChecked<UWidgetBlueprintGeneratedClass>(InObject));
					check(Widget != nullptr);

					Widget->OpenWidget(Forward<Args>(CaptureArgs)...);
				};

				TSharedPtr<FStreamableHandle> StreamableHandle = UMaoAssetManager::Get().RequestAsyncLoad(WidgetPath->ToSoftObjectPath(), Forward<FuncType>(CompleteFunction));
				if (StreamableHandle.IsValid() == false)
				{
					return;
				}

				LoadingWidgetMap.Emplace(T::StaticClass(), MoveTemp(StreamableHandle));
			}
		}
	}
```

### 인게임
---
![Platformer_IngameIntro](/assets/img/Platformer_IngameIntro.png){:style="border:0px solid #eaeaea; border-radius: 7px; padding: 0px;" }

![Platformer_IngamePlay](/assets/img/Platformer_IngamePlay.png){:style="border:0px solid #eaeaea; border-radius: 7px; padding: 0px;" }

#### 데이터 초기화
---
![gamemode](/assets/img/gamemode.png){:style="border:0px solid #eaeaea; border-radius: 7px; padding: 0px;" }

( 참고: [게임 모드와 게임 스테이트](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/game-mode-and-game-state-in-unreal-engine) )
{: style="color:gray; text-align: center;"}  


1. 레벨 로드가 완료된 후 UWorld::BeginPlay 함수가 호출되면서 게임에 필요한 액터들이 생성됩니다. 액터들이 생성되고 나면 게임이 시작됐다고 알리는 AGameMode::StartPlay 함수가 호출 됩니다. 게임 시작 전에 게임을 소개하는 레벨 시퀀스를 재생하고 싶어서 게임 시작 딜레이를 걸고 모든 플레이어에게 레벨 시퀀스를 재생하도록 했습니다. 이때는 액터들의 BeginPlay 함수가 호출되지 않은 시점입니다.
```c++
	void ADiaryPlatformerGameMode::StartPlay()
	{
		bDelayedStart = true;

		for (FConstPlayerControllerIterator Iterator = GetWorld()->GetPlayerControllerIterator(); Iterator; ++Iterator)
		{
			if (ADiaryPlatformerPlayerController* PC = Cast<ADiaryPlatformerPlayerController>(Iterator->Get()))
			{
				PC->FlowStep_PlayIntroLevelSequence(0);
			}
		}
	}
```
	{% include note.html content="<br/>지금은 연출의 스텝을 각각의 함수로 분리했지만, C++20에서 나온 코루틴 문법을 쓰는 것도 좋아 보입니다." %}

1. 레벨 시퀀스 재생이 완료된 후에 AGameMode::StartPlay 함수를 호출 시켜 모든 액터의 BeginPlay 함수가 불릴 수 있도록 합니다.

1. 언리얼 엔진에서 제공하는 `ModularGameplay 플러그인`을 사용하여 플레이어 게임 시작까지의 상태를 관리 하였습니다. 해당 플러그인에서는 상태를 GameplayTag로 관리합니다. 총 4가지의 게임 시작 상태를 가지고 있습니다.
	2. InitState_Spawned
	2. InitState_DataAvailable
	2. InitState_DataInitialized
	2. InitState_GameplayReady
	
1. 저의 경우에는 시작 카운트다운 위젯의 연출이 종료된 후에 다음 스텝으로 넘어갈 수 있도록 코드를 작성 했습니다.
```c++
	bool UDiaryPlatformerPawnComponent::CanChangeInitState(UGameFrameworkComponentManager* Manager, FGameplayTag CurrentState, FGameplayTag DesiredState) const
	{
		...
		
		else if (CurrentState == InitTags.InitState_DataAvailable && DesiredState == InitTags.InitState_DataInitialized)
		{
			// Wait for player state and extension component
			ADiaryPlatformerPlayerState* DiaryPS = GetPlayerState<ADiaryPlatformerPlayerState>();

			return DiaryPS && DiaryPS->IsCountDownFinished() && Manager->HasFeatureReachedInitState(Pawn, UDiaryPawnExtensionComponent::NAME_ActorFeatureName, InitTags.InitState_DataInitialized);
		}
		
		...
		
		return false;
	}
```

1. 카운트 다운이 종료되면 플레이어의 입력을 초기화 해줍니다. 언리얼 엔진에서 제공하는 [향상된 입력 플러그인](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/enhanced-input-in-unreal-engine?application_version=5.5) 을 사용했습니다.이 플러그인의 강력함은 특정 어빌리티나 상황에 따라 입력을 추가/삭제 할 수 있다는 점 같습니다.
![enhancedinput](/assets/img/enhancedinput.png){:style="border:0px solid #eaeaea; border-radius: 7px; padding: 0px;" }

#### 레벨 오브젝트
---
![levelobject](/assets/img/levelobject.png){:style="border:0px solid #eaeaea; border-radius: 7px; padding: 0px;" }

각각의 레벨 오브젝트는 ADiaryPlatformerObstacleActor 부모 클래스를 상속 받아 기능에 맞게 자식 클래스를 만들어서 공용으로 사용 하였습니다. 액터의 종류는 총 4가지 입니다.

|클래스 이름                          |설명    |
|---------------------------------|---------------|
|ADiaryPlatformerRotateActor      |회전 하는 오브젝트|
|ADiaryHammerObstacleActor        |뼈대가 있고 망치 부분을 회전 시키는 오브젝트|
|ADiaryPlatformerOverlappedActor  |박스 컴포넌트와 오버랩되면 이벤트가 발생하는 오브젝트|
|ADiarySplineMovementObstacleActor|스플라인 컴포넌트를 기반으로 움직이는 오브젝트|

그리고 사용된 컴포넌트는 아래와 같습니다.

|클래스 이름                          |설명    |
|---------------------------------|---------------|
|UDiaryRepeatRotationComponent    |실제 액터를 회전시키는 기능을 하는 컴포넌트|
|UDiarySplineMovementComponent    |스플라인 곡선을 기반으로 액터를 움직이게 하는 컴포넌트|

각 컴포넌트에서 필요한 기능을 에디터에 노출시켜 실시간으로 테스트 해가며 게임을 완성할 수 있었습니다.

![obstacleproperty](/assets/img/obstacleproperty.png){:style="border:0px solid #eaeaea; border-radius: 7px; padding: 0px;" }