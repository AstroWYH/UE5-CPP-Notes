lyra动画蓝图的基础放在了ABP_Mannequin_Base，然后里面写好了一些基本的逻辑，然后根据不同的逻辑，分了很多不同的layer函数，这些layer的input/output接口，定义在了ALI_ItemAnimLayers里面，然后具体的实现是在ABP_ItemAnimLayersBase里面，然后ABP_ItemAnimLayersBase会定义很多的动画序列，但不会去填充他们，它会有几个子类，比如手枪、步枪，这些子类会提供这些动画序列。

Lyra 示例项目中的动画系统设计确实采用了这种分层和模块化的架构。以下是对这种设计模式的详细解释：

1. 核心架构
1.1 ABP_Mannequin_Base
作用：这是动画系统的核心动画蓝图，包含了角色的基础动画逻辑（如移动、跳跃、蹲下等）。

特点：

定义了角色的基础状态机（State Machine）。

通过 Anim Layer Interface 与其他动画层（Layer）进行交互。

1.2 ALI_ItemAnimLayers
作用：这是一个动画层接口（Anim Layer Interface），定义了动画层的输入和输出。

特点：

提供了一组标准的接口（如 UpdateItemAnimLayer），用于在动画层之间传递数据。

确保不同动画层之间的兼容性和一致性。

1.3 ABP_ItemAnimLayersBase
作用：这是动画层的基类，实现了 ALI_ItemAnimLayers 接口。

特点：

定义了动画层的逻辑（如武器动作、换弹、瞄准等）。

提供了动画序列的占位符（Placeholder），但不具体实现这些动画。

1.4 子类（如手枪、步枪）
作用：这些子类继承自 ABP_ItemAnimLayersBase，并提供了具体的动画序列。

特点：

每个子类对应一种武器类型（如手枪、步枪）。

子类负责填充父类中定义的动画序列（如射击动画、换弹动画）。

2. 工作流程
2.1 基础动画逻辑
ABP_Mannequin_Base 负责处理角色的基础动画逻辑（如移动、跳跃、蹲下等）。

它通过 ALI_ItemAnimLayers 接口与其他动画层进行交互。

2.2 动画层逻辑
ABP_ItemAnimLayersBase 实现了 ALI_ItemAnimLayers 接口，定义了动画层的逻辑（如武器动作、换弹、瞄准等）。

它提供了动画序列的占位符，但不具体实现这些动画。

2.3 具体动画实现
子类（如手枪、步枪）继承自 ABP_ItemAnimLayersBase，并提供了具体的动画序列。

这些子类负责填充父类中定义的动画序列（如射击动画、换弹动画）。

2.4 动画混合
在运行时，ABP_Mannequin_Base 会根据角色的状态（如是否持枪、是否射击）动态调用 ALI_ItemAnimLayers 接口，将动画层的输出与基础动画混合。

3. 示例
以下是一个简单的示例，展示这种架构的工作流程：

3.1 ABP_Mannequin_Base
定义基础动画逻辑（如移动、跳跃、蹲下）。

通过 ALI_ItemAnimLayers 接口调用动画层的逻辑。

3.2 ALI_ItemAnimLayers
定义接口（如 UpdateItemAnimLayer），用于在动画层之间传递数据。

3.3 ABP_ItemAnimLayersBase
实现 ALI_ItemAnimLayers 接口，定义动画层的逻辑。

提供动画序列的占位符（如 ShootAnim、ReloadAnim）。

3.4 子类（如手枪）
继承自 ABP_ItemAnimLayersBase。

提供具体的动画序列（如 Pistol_ShootAnim、Pistol_ReloadAnim）。

3.5 运行时
当角色持枪时，ABP_Mannequin_Base 会调用 ALI_ItemAnimLayers 接口，将手枪的动画序列与基础动画混合。

例如，角色在奔跑时射击，下半身会继续播放奔跑动画，而上半身会播放射击动画。

4. 优势
4.1 模块化设计
将动画逻辑分解为多个独立的模块（如基础动画、武器动画），使代码更加清晰和易于维护。

4.2 复用性
通过继承和接口，可以复用通用的动画逻辑（如换弹、瞄准），减少重复工作。

4.3 灵活性
通过动态调用动画层接口，可以根据角色的状态（如是否持枪、是否射击）动态调整动画逻辑。

4.4 扩展性
添加新的武器类型时，只需创建一个新的子类并实现具体的动画序列，而无需修改基础动画逻辑。

5. 总结
Lyra 示例项目中的动画系统采用了分层和模块化的设计，通过 ABP_Mannequin_Base、ALI_ItemAnimLayers、ABP_ItemAnimLayersBase 和子类的组合，实现了高效且灵活的动画逻辑管理。这种设计模式特别适用于复杂的动画系统，能够显著提高代码的可维护性和扩展性。
