Enhanced Input 系统相对于旧的输入系统，能够解决一些更复杂的输入需求和场景。以下是一个具体例子，展示了 Enhanced Input 的好处，特别是在处理组合键输入和动态重映射时的优势。

场景：组合键输入和动态重映射
旧的输入系统的问题
在旧的输入系统中，实现组合键（例如按下 Shift + A 才能触发某个动作）和动态重映射会相对复杂。你需要手动检测多个按键的状态，并在游戏运行时更改输入绑定，这可能需要编写大量的代码，并且不够直观。

Enhanced Input 系统的解决方案
步骤 1：定义输入动作
使用 Enhanced Input 系统，可以定义一个输入动作来表示组合键。
```cpp
// 定义一个输入动作，支持组合键
UInputAction* SprintAction = NewObject<UInputAction>();
SprintAction->InputActionName = FName("Sprint");
SprintAction->AddMapping(FKeyMapping(FKey("LeftShift"), EInputTrigger::Pressed));
SprintAction->AddMapping(FKeyMapping(FKey("A"), EInputTrigger::Pressed));
```
步骤 2：绑定输入动作
然后，将这个输入动作绑定到一个函数，这样当组合键被按下时，就会触发该函数。
```cpp
PlayerInputComponent->BindAction(SprintAction, ETriggerEvent::Triggered, this, &AMyCharacter::Sprint);

void AMyCharacter::Sprint()
{
    // 处理冲刺逻辑
    UE_LOG(LogTemp, Warning, TEXT("Sprinting!"));
}
```
步骤 3：动态重映射输入
Enhanced Input 系统允许你在游戏运行时动态地更改输入映射。例如，玩家可以在设置菜单中重新定义组合键。
```cpp
void AMyCharacter::RemapSprintKey(FKey NewKey)
{
    SprintAction->ClearMappings();
    SprintAction->AddMapping(FKeyMapping(FKey("LeftShift"), EInputTrigger::Pressed));
    SprintAction->AddMapping(FKeyMapping(NewKey, EInputTrigger::Pressed));
}

// 在游戏设置菜单中调用 RemapSprintKey 函数
RemapSprintKey(FKey("B")); // 将组合键改为 Shift + B
```
比较和总结
在旧的输入系统中，处理组合键和动态重映射需要手动检查每个按键的状态，并且需要编写复杂的逻辑来管理按键的状态和输入的变化。代码可能会变得难以维护，并且在处理复杂的输入方案时，容易出错。
```cpp
// 旧输入系统中手动检查组合键的状态
void AMyCharacter::Tick(float DeltaTime)
{
    Super::Tick(DeltaTime);

    if (PlayerInputComponent->IsKeyPressed(EKeys::LeftShift) && PlayerInputComponent->IsKeyPressed(EKeys::A))
    {
        Sprint();
    }
}

void AMyCharacter::RemapSprintKey(FKey NewKey)
{
    // 在旧系统中，需要额外逻辑来处理输入重映射
}
```
使用 Enhanced Input 系统，只需要定义输入动作并绑定相应的处理函数，极大地简化了代码，并提高了可读性和可维护性。动态重映射输入也变得非常直观和易于实现。Enhanced Input 系统提供了更强大的输入处理能力，能够轻松应对复杂的输入需求，从而提升了开发效率和用户体验。
