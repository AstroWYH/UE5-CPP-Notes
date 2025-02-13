StartGame() 方法：
初始化游戏数据：激活地图中的道具，使其可被玩家交互。
初始化玩家状态：重置玩家的生命值、弹药等状态，将玩家传送到游戏开始位置，并开启玩家控制。
开始游戏计时器：设置一个计时器，在游戏时长结束后检查游戏是否结束。
EndGame() 方法：
停止计时器：停止游戏中所有的计时器。
禁用玩家控制：禁用玩家的输入控制，显示鼠标光标。
显示游戏结束界面：创建并显示游戏结束的 UMG 界面。
处理游戏结算数据：统计玩家得分、排名等信息。
返回大厅：在一段时间后调用 TransitionToPhase(EGamePhase::Lobby) 方法返回大厅阶段。
CheckGameEndCondition() 方法：
检查存活玩家的数量，如果存活玩家数量小于等于 1，则调用 TransitionToPhase(EGamePhase::GameOver) 方法进入游戏结束阶段。
通过以上代码，你可以在 UE5 中实现一个简单的分阶段游戏流程。

```cpp
#include "CoreMinimal.h"
#include "GameFramework/GameModeBase.h"
#include "YourGameMode.generated.h"

// 定义游戏阶段枚举
UENUM(BlueprintType)
enum class EGamePhase : uint8
{
    Lobby,
    InGame,
    GameOver
};

UCLASS()
class YOURGAME_API AYourGameMode : public AGameModeBase
{
    GENERATED_BODY()

public:
    AYourGameMode();

    // 重写 BeginPlay 函数，在游戏开始时进入大厅阶段
    virtual void BeginPlay() override;

    // 当前游戏阶段
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "GamePhase")
    EGamePhase CurrentGamePhase = EGamePhase::Lobby;

    // 计时器句柄
    FTimerHandle LobbyTimerHandle;

    // 大厅等待时间
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "GamePhase")
    float LobbyWaitTime = 30.0f;

    // 开始大厅计时
    void StartLobbyTimer();

    // 大厅计时结束回调函数
    void OnLobbyTimerEnd();

    // 状态转换函数
    void TransitionToPhase(EGamePhase NewPhase);

    // 开始游戏函数
    void StartGame();

    // 结束游戏函数
    void EndGame();

    // 检查游戏是否结束
    void CheckGameEndCondition();

    // 定义一个事件委托，用于广播游戏阶段变化
    DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnGamePhaseChanged, EGamePhase, NewPhase);
    UPROPERTY(BlueprintAssignable, Category = "GamePhase")
    FOnGamePhaseChanged OnGamePhaseChanged;
};

// 构造函数
AYourGameMode::AYourGameMode()
{
    // 默认构造函数，可进行一些初始化操作
}

// 重写 BeginPlay 函数，在游戏开始时进入大厅阶段
void AYourGameMode::BeginPlay()
{
    Super::BeginPlay();
    // 游戏开始时进入大厅阶段
    TransitionToPhase(EGamePhase::Lobby);
}

// 开始大厅计时
void AYourGameMode::StartLobbyTimer()
{
    GetWorldTimerManager().SetTimer(LobbyTimerHandle, this, &AYourGameMode::OnLobbyTimerEnd, LobbyWaitTime, false);
}

// 大厅计时结束回调函数
void AYourGameMode::OnLobbyTimerEnd()
{
    // 检查玩家人数等条件，这里简单假设直接进入游戏
    TransitionToPhase(EGamePhase::InGame);
}

// 状态转换函数
void AYourGameMode::TransitionToPhase(EGamePhase NewPhase)
{
    CurrentGamePhase = NewPhase;
    // 广播状态转换事件
    OnGamePhaseChanged.Broadcast(NewPhase);
    switch (NewPhase)
    {
    case EGamePhase::Lobby:
        // 进入大厅阶段的逻辑
        StartLobbyTimer();
        break;
    case EGamePhase::InGame:
        // 进入游戏阶段的逻辑
        StartGame();
        break;
    case EGamePhase::GameOver:
        // 进入游戏结束阶段的逻辑
        EndGame();
        break;
    }
}

// 开始游戏函数
void AYourGameMode::StartGame()
{
    // 1. 初始化游戏相关数据
    // 例如，初始化地图中的道具、武器等
    TArray<AActor*> Props;
    UGameplayStatics::GetAllActorsOfClass(GetWorld(), AYourPropClass::StaticClass(), Props);
    for (AActor* Prop : Props)
    {
        // 激活道具，使其可被玩家交互
        AYourPropClass* YourProp = Cast<AYourPropClass>(Prop);
        if (YourProp)
        {
            YourProp->ActivateProp();
        }
    }

    // 2. 初始化玩家状态
    TArray<APlayerController*> PlayerControllers;
    UGameplayStatics::GetAllPlayerControllers(GetWorld(), PlayerControllers);
    for (APlayerController* PC : PlayerControllers)
    {
        AYourCharacter* PlayerCharacter = Cast<AYourCharacter>(PC->GetPawn());
        if (PlayerCharacter)
        {
            // 重置玩家生命值、弹药等状态
            PlayerCharacter->ResetPlayerStats();
            // 将玩家传送到游戏开始位置
            FVector StartLocation = FVector(0.0f, 0.0f, 0.0f); // 假设的起始位置
            PlayerCharacter->SetActorLocation(StartLocation);
            // 开启玩家控制
            PC->bShowMouseCursor = false;
            PC->SetInputMode(FInputModeGameOnly());
        }
    }

    // 3. 开始游戏计时器（如果需要）
    FTimerHandle GameTimerHandle;
    float GameDuration = 300.0f; // 假设游戏时长为 300 秒
    GetWorldTimerManager().SetTimer(GameTimerHandle, this, &AYourGameMode::CheckGameEndCondition, GameDuration, false);
}

// 结束游戏函数
void AYourGameMode::EndGame()
{
    // 1. 停止游戏中的所有计时器
    GetWorldTimerManager().ClearAllTimersForObject(this);

    // 2. 禁用玩家控制
    TArray<APlayerController*> PlayerControllers;
    UGameplayStatics::GetAllPlayerControllers(GetWorld(), PlayerControllers);
    for (APlayerController* PC : PlayerControllers)
    {
        PC->bShowMouseCursor = true;
        PC->SetInputMode(FInputModeUIOnly());
    }

    // 3. 显示游戏结束界面
    // 假设我们有一个 UMG 界面类 UYourGameOverWidget
    UYourGameOverWidget* GameOverWidget = CreateWidget<UYourGameOverWidget>(GetWorld(), UYourGameOverWidget::StaticClass());
    if (GameOverWidget)
    {
        GameOverWidget->AddToViewport();
    }

    // 4. 处理游戏结算数据
    // 例如，统计玩家得分、排名等
    TArray<AYourCharacter*> AlivePlayers;
    TArray<APlayerController*> PlayerControllersArray;
    UGameplayStatics::GetAllPlayerControllers(GetWorld(), PlayerControllersArray);
    for (APlayerController* PC : PlayerControllersArray)
    {
        AYourCharacter* PlayerCharacter = Cast<AYourCharacter>(PC->GetPawn());
        if (PlayerCharacter && PlayerCharacter->IsAlive())
        {
            AlivePlayers.Add(PlayerCharacter);
        }
    }
    // 可以根据 AlivePlayers 数组进行排名等操作

    // 5. 可以选择在一段时间后返回大厅或主菜单
    FTimerHandle ReturnTimerHandle;
    float ReturnDelay = 10.0f; // 10 秒后返回
    GetWorldTimerManager().SetTimer(ReturnTimerHandle, this, [this]() {
        TransitionToPhase(EGamePhase::Lobby);
    }, ReturnDelay, false);
}

// 检查游戏是否结束
void AYourGameMode::CheckGameEndCondition()
{
    int32 AlivePlayerCount = 0;
    TArray<APlayerController*> PlayerControllers;
    UGameplayStatics::GetAllPlayerControllers(GetWorld(), PlayerControllers);
    for (APlayerController* PC : PlayerControllers)
    {
        AYourCharacter* PlayerCharacter = Cast<AYourCharacter>(PC->GetPawn());
        if (PlayerCharacter && PlayerCharacter->IsAlive())
        {
            AlivePlayerCount++;
        }
    }

    if (AlivePlayerCount <= 1)
    {
        // 游戏结束，进入游戏结束阶段
        TransitionToPhase(EGamePhase::GameOver);
    }
}

```
