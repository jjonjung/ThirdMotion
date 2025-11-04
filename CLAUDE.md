# ThirdMotion 가이드
**프로젝트**: UE5.6 Actor/Mesh/Light/Camera 편집툴 | **구조**: C++로직+BP UI

## 제약사항
- Engine 수정 금지 → 상속만✅
- 이모티콘 금지, CMD 한글, 가독성 유지
- 동기화 로직 → Network/ServerController, Network/ServerManager 에서 처리
- 빌드금지

## 구조
### UI(MVC)
```
UBaseWidget(필수상속)
├ UMainWidget
├ URightPanel  
├ UBottomBar
└ UViewportWidget
+UBaseWidgetController(로직분리)
```

### 디렉토리
```
Source/ThirdMotion/
├Public/
│├Framework/
│├UI/Widget|Panel|WidgetController/
│├Network/
│└Data/
└Private/
```

### DataFlow
Data→Controller→Widget→Panel

## 필수규칙
### BindWidget
```cpp
UPROPERTY(BlueprintReadWrite,meta=(BindWidget))
UBottomBar* BottomBar; //BP이름일치필수
UPROPERTY(BlueprintReadWrite,meta=(BindWidgetOptional))
URightPanel* RightPanel;
```

### 델리게이트
```cpp
//NativeConstruct: 바인드
BottomBar->OnSceneButtonClicked.AddDynamic(this,&UMainWidget::OnSceneButtonClicked);
//NativeDestruct: 언바인드필수!
BottomBar->OnSceneButtonClicked.RemoveDynamic(this,&UMainWidget::OnSceneButtonClicked);
```

### 네트워크체크
```cpp
IsServer()=GetAuthGameMode()!=nullptr
IsClient()=GetNetMode()==NM_Client
HasAuthority()=Role==ROLE_Authority
```

### RPC동기화
```cpp
UFUNCTION(Server,Reliable)
void ServerUpdateActor(AActor* Target,FVector NewLocation);
UFUNCTION(Client,Reliable) 
void MulticastUpdateActor(FVector NewLocation);
```
참조: `C:\Users\user\Desktop\NetTPS\Source\NetTPS\NetActor.cpp`

## 스타일
- 함수/변수: PascalCase | bool: bPrefix
- 주석: 한글 | 로그: 디버그만

## 트러블슈팅
1. **HotReload실패**: 위젯타입변경→전체리빌드
2. **크래시**: NativeDestruct 델리게이트언바인드누락
3. **BindWidget실패**: C++/BP이름불일치
4. **UI안보임**: PlayerController생성(GameMode❌)

## 최적화&에러처리
```cpp
SetComponentTickEnabled(false); //Tick최소화
SetCollisionEnabled(ECollisionEnabled::NoCollision);
if(!IsValid(Actor))return; //널체크필수
UE_LOG(LogTemp,Error,TEXT("에러:%s"),*ErrorMessage);
```
