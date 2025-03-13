将 Mass 框架与 Actor 结合起来，管理场景中的大量 Actor。这是一个非常常见的需求，因为 Mass 框架的高效性可以用来管理大量的 Actor，同时仍然保留 Actor 的可见性和交互性。

在这个示例中，我们将：
1. 创建 100 个蓝图 Actor。
2. 使用 Mass 框架管理这些 Actor 的移动逻辑。
3. 将 Mass 实体与 Actor 关联起来。

---

### 改进后的功能
1. 在场景中生成 100 个蓝图 Actor。
2. 每个 Actor 有一个 Mass 实体，用于管理其移动逻辑。
3. Actor 根据 Mass 实体的数据移动，并在碰到边界时反弹。

---

### 需要创建的类和文件
1. **FMassMovementFragment**：存储实体的速度。
2. **FMassActorFragment**：将 Mass 实体与 Actor 关联。
3. **FMassRandomMovementTag**：标记需要随机移动的实体。
4. **UMassMovementProcessor**：处理实体的移动逻辑。
5. **AMyMassActor**：蓝图 Actor 的基类。
6. **MyMassGameMode**：用于初始化 Mass 实体和 Actor。

---

### 详细步骤

#### 1. 创建 UE 工程
- 打开 Unreal Engine 5，创建一个新的 C++ 项目（例如 `MassActorDemo`）。
- 启用 Mass 插件：`Edit -> Plugins -> Mass`，搜索并启用 `MassEntity` 和 `MassGameplay`。

#### 2. 创建 Fragment 类
- **FMassMovementFragment**：存储实体的速度。
- **FMassActorFragment**：将 Mass 实体与 Actor 关联。

在 `Source/MassActorDemo/Public` 下创建 `MassMovementFragment.h`：
```cpp
#pragma once

#include "MassCommonFragments.h"
#include "MassMovementFragment.generated.h"

USTRUCT()
struct MASSACTORDEMO_API FMassMovementFragment : public FMassFragment
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere)
    FVector Velocity;
};
```

在 `Source/MassActorDemo/Public` 下创建 `MassActorFragment.h`：
```cpp
#pragma once

#include "MassCommonFragments.h"
#include "MassActorFragment.generated.h"

USTRUCT()
struct MASSACTORDEMO_API FMassActorFragment : public FMassFragment
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere)
    AActor* Actor;
};
```

#### 3. 创建 Tag 类
在 `Source/MassActorDemo/Public` 下创建 `MassRandomMovementTag.h`：
```cpp
#pragma once

#include "MassCommonFragments.h"
#include "MassRandomMovementTag.generated.h"

USTRUCT()
struct MASSACTORDEMO_API FMassRandomMovementTag : public FMassTag
{
    GENERATED_BODY()
};
```

#### 4. 创建 Processor 类
在 `Source/MassActorDemo/Public` 下创建 `MassMovementProcessor.h`：
```cpp
#pragma once

#include "MassProcessor.h"
#include "MassCommonFragments.h"
#include "MassMovementFragment.h"
#include "MassActorFragment.h"
#include "MassRandomMovementTag.h"
#include "MassMovementProcessor.generated.h"

UCLASS()
class MASSACTORDEMO_API UMassMovementProcessor : public UMassProcessor
{
    GENERATED_BODY()

public:
    UMassMovementProcessor();

protected:
    virtual void ConfigureQueries() override;
    virtual void Execute(FMassEntityManager& EntityManager, FMassExecutionContext& Context) override;

private:
    FMassEntityQuery EntityQuery;
};
```

在 `Source/MassActorDemo/Private` 下创建 `MassMovementProcessor.cpp`：
```cpp
#include "MassMovementProcessor.h"
#include "GameFramework/Actor.h"

UMassMovementProcessor::UMassMovementProcessor()
{
    bAutoRegisterWithProcessingPhases = true;
    ExecutionFlags = (int32)EProcessorExecutionFlags::All;
}

void UMassMovementProcessor::ConfigureQueries()
{
    EntityQuery.AddRequirement<FMassMovementFragment>(EMassFragmentAccess::ReadWrite);
    EntityQuery.AddRequirement<FMassActorFragment>(EMassFragmentAccess::ReadWrite);
    EntityQuery.AddRequirement<FMassRandomMovementTag>(EMassFragmentAccess::ReadOnly);
}

void UMassMovementProcessor::Execute(FMassEntityManager& EntityManager, FMassExecutionContext& Context)
{
    EntityQuery.ForEachEntityChunk(EntityManager, Context, [](FMassExecutionContext& Context)
    {
        const TArrayView<FMassMovementFragment> MovementFragments = Context.GetMutableFragmentView<FMassMovementFragment>();
        const TArrayView<FMassActorFragment> ActorFragments = Context.GetMutableFragmentView<FMassActorFragment>();

        for (int32 i = 0; i < Context.GetNumEntities(); ++i)
        {
            FVector& Velocity = MovementFragments[i].Velocity;
            AActor* Actor = ActorFragments[i].Actor;

            if (Actor)
            {
                // Update actor position
                FVector NewLocation = Actor->GetActorLocation() + Velocity * Context.GetDeltaTimeSeconds();
                Actor->SetActorLocation(NewLocation);

                // Simple bounce logic
                if (NewLocation.X > 5000 || NewLocation.X < -5000) Velocity.X *= -1;
                if (NewLocation.Y > 5000 || NewLocation.Y < -5000) Velocity.Y *= -1;
                if (NewLocation.Z > 5000 || NewLocation.Z < -5000) Velocity.Z *= -1;
            }
        }
    });
}
```

#### 5. 创建 Actor 类
在 `Source/MassActorDemo/Public` 下创建 `MyMassActor.h`：
```cpp
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "MyMassActor.generated.h"

UCLASS()
class MASSACTORDEMO_API AMyMassActor : public AActor
{
    GENERATED_BODY()

public:
    AMyMassActor();
};
```

在 `Source/MassActorDemo/Private` 下创建 `MyMassActor.cpp`：
```cpp
#include "MyMassActor.h"
#include "Components/StaticMeshComponent.h"

AMyMassActor::AMyMassActor()
{
    PrimaryActorTick.bCanEverTick = false;

    // Add a static mesh component
    UStaticMeshComponent* MeshComponent = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("MeshComponent"));
    RootComponent = MeshComponent;
}
```

#### 6. 创建 GameMode 类
在 `Source/MassActorDemo/Public` 下创建 `MyMassGameMode.h`：
```cpp
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/GameModeBase.h"
#include "MyMassGameMode.generated.h"

UCLASS()
class MASSACTORDEMO_API AMyMassGameMode : public AGameModeBase
{
    GENERATED_BODY()

public:
    virtual void StartPlay() override;
};
```

在 `Source/MassActorDemo/Private` 下创建 `MyMassGameMode.cpp`：
```cpp
#include "MyMassGameMode.h"
#include "MassEntitySubsystem.h"
#include "MassSpawnerSubsystem.h"
#include "MassMovementFragment.h"
#include "MassActorFragment.h"
#include "MassRandomMovementTag.h"
#include "MyMassActor.h"

void AMyMassGameMode::StartPlay()
{
    Super::StartPlay();

    UWorld* World = GetWorld();
    if (!World) return;

    // Get Mass subsystems
    UMassEntitySubsystem* EntitySubsystem = World->GetSubsystem<UMassEntitySubsystem>();
    UMassSpawnerSubsystem* SpawnerSubsystem = World->GetSubsystem<UMassSpawnerSubsystem>();
    if (!EntitySubsystem || !SpawnerSubsystem) return;

    // Define archetype
    FMassEntityTemplate EntityTemplate;
    EntityTemplate.AddFragment<FMassMovementFragment>();
    EntityTemplate.AddFragment<FMassActorFragment>();
    EntityTemplate.AddTag<FMassRandomMovementTag>();

    // Spawn entities and actors
    const int32 NumEntities = 100;
    TArray<FMassEntityHandle> Entities;
    SpawnerSubsystem->SpawnEntities(EntityTemplate, NumEntities, Entities);

    for (const FMassEntityHandle& Entity : Entities)
    {
        FMassMovementFragment* MovementFragment = EntitySubsystem->GetFragmentDataPtr<FMassMovementFragment>(Entity);
        FMassActorFragment* ActorFragment = EntitySubsystem->GetFragmentDataPtr<FMassActorFragment>(Entity);

        if (MovementFragment && ActorFragment)
        {
            // Spawn an actor
            FActorSpawnParameters SpawnParams;
            SpawnParams.SpawnCollisionHandlingOverride = ESpawnActorCollisionHandlingMethod::AlwaysSpawn;
            AMyMassActor* Actor = World->SpawnActor<AMyMassActor>(AMyMassActor::StaticClass(), FVector::ZeroVector, FRotator::ZeroRotator, SpawnParams);

            // Initialize fragments
            MovementFragment->Velocity = FVector(FMath::FRandRange(-100, 100), FMath::FRandRange(-100, 100), FMath::FRandRange(-100, 100));
            ActorFragment->Actor = Actor;
        }
    }
}
```

#### 7. 设置 GameMode
- 打开 `Edit -> Project Settings -> Maps & Modes`，将 `Default GameMode` 设置为 `MyMassGameMode`。

#### 8. 创建蓝图 Actor
- 在 Content Browser 中右键 `MyMassActor`，选择 `Create Blueprint Class`，命名为 `BP_MyMassActor`。
- 打开 `BP_MyMassActor`，为其添加一个静态网格体（Static Mesh），例如一个立方体。

#### 9. 运行工程
- 编译并运行工程。你会看到场景中生成了 100 个 Actor，它们随机移动并在碰到边界时反弹。

---

### 总结
1. 如何将 Mass 实体与 Actor 关联。
2. 如何使用 Mass 框架管理大量 Actor 的逻辑。
3. 如何初始化 Mass 实体和 Actor。

