UState 类
UState 代表状态机中的一个状态。在每个状态中，可以定义角色或对象在该状态下的行为。
```cpp
// State.h
#pragma once

#include "CoreMinimal.h"
#include "State.generated.h"

UCLASS()
class MYGAME_API UState : public UObject
{
    GENERATED_BODY()

public:
    // 在状态开始时调用
    virtual void OnEnter();

    // 在状态更新时调用
    virtual void OnUpdate();

    // 在状态退出时调用
    virtual void OnExit();
};
```
UTransition 类
UTransition 定义了从一个状态到另一个状态的转换条件。它通常包含一个检查转换条件的方法。
```cpp
// Transition.h
#pragma once

#include "CoreMinimal.h"
#include "Transition.generated.h"

UCLASS()
class MYGAME_API UTransition : public UObject
{
    GENERATED_BODY()

public:
    // 转换条件检查函数
    bool IsTransitionValid() const
    {
        if (TransitionCondition)
        {
            return TransitionCondition();
        }
        return false;
    }

    // 设置转换条件
    void SetTransitionCondition(TFunction<bool()> InCondition)
    {
        TransitionCondition = InCondition;
    }

private:
    // 条件函数
    TFunction<bool()> TransitionCondition;
};

```
UStateMachine 类
UStateMachine 管理状态和状态之间的转换。它负责更新状态和处理状态转换逻辑。
```cpp
// StateMachine.h
#pragma once

#include "CoreMinimal.h"
#include "State.h"
#include "Transition.h"
#include "StateMachine.generated.h"

UCLASS()
class MYGAME_API UStateMachine : public UObject
{
    GENERATED_BODY()

public:
    // 设置初始状态
    void SetInitialState(UState* InitialState);

    // 添加状态
    void AddState(UState* State);

    // 添加转换
    void AddTransition(UState* FromState, UState* ToState, UTransition* Transition);

    // 更新状态机
    void Update();

protected:
    UPROPERTY()
    UState* CurrentState;

    TMap<UState*, TMap<UState*, UTransition*>> Transitions;
};

// StateMachine.cpp
#include "StateMachine.h"

void UStateMachine::SetInitialState(UState* InitialState)
{
    CurrentState = InitialState;
    if (CurrentState)
    {
        CurrentState->OnEnter();
    }
}

void UStateMachine::AddState(UState* State)
{
    // 将状态添加到状态机中
}

void UStateMachine::AddTransition(UState* FromState, UState* ToState, UTransition* Transition)
{
    if (Transitions.Contains(FromState))
    {
        Transitions[FromState].Add(ToState, Transition);
    }
    else
    {
        TMap<UState*, UTransition*> NewTransitions;
        NewTransitions.Add(ToState, Transition);
        Transitions.Add(FromState, NewTransitions);
    }
}

void UStateMachine::Update()
{
    if (!CurrentState) return;

    for (auto& TransitionPair : Transitions.FindChecked(CurrentState))
    {
        if (TransitionPair.Value->IsTransitionValid()) // 寻找对于当前状态，什么Transition满足，如果满足则转换到目标状态
        {
            CurrentState->OnExit();
            CurrentState = TransitionPair.Key;
            CurrentState->OnEnter();
            break;
        }
    }

    if (CurrentState)
    {
        CurrentState->OnUpdate();
    }
}
```
示例代码
下面的代码展示了如何使用这些类来创建一个简单的状态机，用于角色的不同动画状态：
```cpp
// MyStateMachine.h
#pragma once

#include "CoreMinimal.h"
#include "StateMachine.h"
#include "MyStateMachine.generated.h"

UCLASS()
class MYGAME_API UMyStateMachine : public UStateMachine
{
    GENERATED_BODY()

public:
    UMyStateMachine();

protected:
    virtual void InitializeStateMachine();

private:
    UPROPERTY()
    UState* IdleState;

    UPROPERTY()
    UState* RunState;

    UPROPERTY()
    UState* JumpState;

    UPROPERTY()
    UTransition* IdleToRunTransition;

    UPROPERTY()
    UTransition* RunToJumpTransition;

    UPROPERTY()
    UTransition* JumpToIdleTransition;
};

// MyStateMachine.cpp
#include "MyStateMachine.h"

UMyStateMachine::UMyStateMachine()
{
    // 创建状态
    IdleState = NewObject<UState>(this, UState::StaticClass());
    RunState = NewObject<UState>(this, UState::StaticClass());
    JumpState = NewObject<UState>(this, UState::StaticClass());

    // 创建转换
    IdleToRunTransition = NewObject<UTransition>(this, UTransition::StaticClass());
    RunToJumpTransition = NewObject<UTransition>(this, UTransition::StaticClass());
    JumpToIdleTransition = NewObject<UTransition>(this, UTransition::StaticClass());
}

void UMyStateMachine::InitializeStateMachine()
{
    // 设置初始状态
    SetInitialState(IdleState);

    // 配置状态转换
    IdleToRunTransition->SetTransitionCondition([]() {
        // 返回是否满足转换条件
        return /* 条件逻辑 */;
    });
    RunToJumpTransition->SetTransitionCondition([]() {
        // 返回是否满足转换条件
        return /* 条件逻辑 */;
    });
    JumpToIdleTransition->SetTransitionCondition([]() {
        // 返回是否满足转换条件
        return /* 条件逻辑 */;
    });

    // 添加状态和转换到状态机
    AddState(IdleState);
    AddState(RunState);
    AddState(JumpState);

    AddTransition(IdleState, RunState, IdleToRunTransition);
    AddTransition(RunState, JumpState, RunToJumpTransition);
    AddTransition(JumpState, IdleState, JumpToIdleTransition);
}
```
总结
UState：代表状态，定义状态的行为（OnEnter、OnUpdate、OnExit）。
UTransition：定义状态间的转换条件。
UStateMachine：管理状态和状态之间的转换，更新当前状态。
通过这种方式，可以管理角色的不同状态和状态之间的转换，适用于各种需要状态管理的场景。
