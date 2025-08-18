UE5的增强输入（Enhanced Input）相比老版本的输入系统（如蓝图中的“输入轴事件”“输入动作事件”），核心优势是**更灵活、更易扩展、更贴近实际操作逻辑**。用具体场景对比会更直观：


### 1. 同一动作绑定多个输入源（比如“跳跃”）
- **老方法**：  
  要让“空格”“手柄A键”“触屏按钮”都触发跳跃，需要在项目设置里给“跳跃”动作分别绑定这三个输入，然后在蓝图中监听“跳跃”事件。  
  缺点：如果后续想临时禁用“触屏按钮”的跳跃（比如某些场景只允许键盘操作），必须去项目设置修改绑定，或在代码里加一堆判断，非常繁琐。

- **增强输入**：  
  为“跳跃”动作创建一个“输入映射上下文（Input Mapping Context）”，在其中添加三个输入源（空格、手柄A、触屏）。需要禁用某类输入时，只需在蓝图中移除该输入源对应的映射（比如调用“移除触屏映射”），其他输入源不受影响。  
  举例：角色进入UI界面时，自动移除手柄/键盘映射，只保留触屏，退出后再重新添加，无需修改底层设置。


### 2. 输入优先级与冲突处理（比如“下蹲”和“跳跃”冲突）
- **老方法**：  
  假设“左Ctrl”是下蹲，“空格”是跳跃，且规则是“下蹲时不能跳跃”。需要在蓝图中先判断角色是否处于下蹲状态，再决定是否响应跳跃输入。  
  缺点：如果后续加了“滑行”动作（也不能跳跃），必须在跳跃事件里再加一个判断条件，逻辑越堆越乱。

- **增强输入**：  
  给“跳跃”动作设置“输入触发规则（Input Trigger）”，直接在规则里定义“当下蹲或滑行状态激活时，不触发跳跃”。  
  举例：在“跳跃”的触发规则中添加两个“阻止条件”，关联下蹲和滑行的状态变量，状态变化时自动生效，无需修改跳跃事件的蓝图逻辑，规则和动作解耦。


### 3. 输入轴的精细化控制（比如“移动”或“镜头旋转”）
- **老方法**：  
  监听“移动X轴”“移动Y轴”事件，直接获取轴值（-1到1）来控制角色移动。如果想实现“按下Shift时加速”，需要在蓝图中手动判断Shift是否按下，再乘以加速系数。  
  缺点：如果再加一个“手柄LT键也能加速”，必须在代码里额外判断LT键的状态，轴值处理和按键判断混在一起。

- **增强输入**：  
  为“移动”轴动作添加“输入修饰符（Input Modifier）”，比如“按下Shift或LT时，轴值乘以2”。修饰符可以直接在输入设置中配置，无需修改移动逻辑的蓝图。  
  举例：后续想调整加速系数（比如从2倍改为1.5倍），只需修改修饰符的参数，所有使用该移动动作的角色（玩家、AI同伴）都会自动生效，不用逐个改蓝图。


### 4. 多角色/多状态下的输入切换（比如“开车”和“步行”模式）
- **老方法**：  
  步行时用WASD移动，开车时用WASD控制方向盘+空格刹车。需要在蓝图中用大量分支判断“当前是否在开车”，然后分别处理WASD的逻辑（步行时移动角色，开车时转向）。  
  缺点：如果再加一个“骑马”模式，又要加一套分支判断，代码臃肿且易出错。

- **增强输入**：  
  为“步行”“开车”“骑马”三种状态分别创建独立的“输入映射上下文”：  
  - 步行时激活“步行映射”（WASD=移动）；  
  - 开车时切换到“开车映射”（WASD=转向，空格=刹车）；  
  - 骑马时切换到“骑马映射”（WASD=前进/左右）。  
  举例：切换状态时只需调用“激活新映射+禁用旧映射”，输入逻辑和状态完全绑定，无需分支判断，新增状态时只需加一个新映射。


### 总结：核心区别
老方法更像“直接接线”——输入和逻辑硬绑定，改一处要动全身；  
增强输入像“模块化插线板”——输入源、触发规则、逻辑处理分离，可灵活组合、切换、扩展，尤其适合复杂游戏（多角色、多状态、多平台）。

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
