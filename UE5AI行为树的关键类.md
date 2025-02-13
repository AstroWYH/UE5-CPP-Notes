创建一个简单的行为树示例，以下是关键类和步骤的清晰说明：

关键类：
BTNode: 行为树的基础节点类。所有具体的行为节点都会继承这个类。
BTCompositeNode: 复合节点类，用于包含和管理子节点。常见的子类包括Selector和Sequence。
BTActionNode: 行动节点类，用于执行具体的行为。
BTBlackboard: 用于存储行为树节点之间共享的变量或状态。

```cpp
// 基础节点类
class BTNode {
public:
    virtual bool Execute(BTBlackboard* blackboard) = 0;
};

// 复合节点类
class BTCompositeNode : public BTNode {
protected:
    std::vector<BTNode*> children;

public:
    void AddChild(BTNode* child) {
        children.push_back(child);
    }
};

// Selector节点：按顺序执行子节点，直到找到成功的节点
class BTSelector : public BTCompositeNode {
public:
    virtual bool Execute(BTBlackboard* blackboard) override {
        for (BTNode* child : children) {
            if (child->Execute(blackboard)) {
                return true;
            }
        }
        return false;
    }
};

// Sequence节点：按顺序执行子节点，直到找到失败的节点
class BTSequence : public BTCompositeNode {
public:
    virtual bool Execute(BTBlackboard* blackboard) override {
        for (BTNode* child : children) {
            if (!child->Execute(blackboard)) {
                return false;
            }
        }
        return true;
    }
};

// 行动节点类
class BTActionNode : public BTNode {
public:
    virtual bool Execute(BTBlackboard* blackboard) override {
        // 实际的行为逻辑
        return true;  // 假设行为总是成功
    }
};

// Blackboard类
class BTBlackboard {
    // 存储行为树所需的状态和变量
};

// 使用行为树
int main() {
    BTBlackboard blackboard;

    // 创建行为树
    BTSelector* root = new BTSelector();

    BTSequence* sequence = new BTSequence();
    sequence->AddChild(new BTActionNode());  // 添加行动节点
    sequence->AddChild(new BTActionNode());  // 添加行动节点

    root->AddChild(sequence);
    root->AddChild(new BTActionNode());  // 另一个行动节点

    // 执行行为树
    root->Execute(&blackboard);

    // 清理内存（简化起见，这里略去详细的内存管理）
    delete root;

    return 0;
}

```

BTNode 是基础类，所有节点类型都继承自它。
BTCompositeNode 是复合节点，包含子节点，可以是 Selector 或 Sequence。
BTSelector 和 BTSequence 是常见的复合节点类型，分别处理选择逻辑和序列逻辑。
BTActionNode 是具体的行动节点，执行某种行为。
BTBlackboard 是共享数据的存储类，用于在行为树的节点之间共享信息。

在行为树中，BTSelector 和 BTSequence 是两种不同类型的复合节点（Composite Nodes），它们用来控制子节点的执行逻辑。这两种节点的行为各自不同，但都是用来组织和管理子节点执行的顺序和条件的。
. BTSelector（选择节点）
作用: BTSelector节点会按顺序执行它的子节点，直到其中一个子节点成功为止。
执行逻辑:
它依次执行每个子节点。
如果某个子节点执行成功（返回true），则BTSelector节点也返回成功，停止执行后续子节点。
如果所有子节点都执行失败（返回false），则BTSelector节点返回失败。
使用场景: BTSelector通常用于实现“或”的逻辑，比如选择某种成功的行为，只要找到一种能成功的就停止尝试其他行为。
```cpp
BTSelector* root = new BTSelector();
root->AddChild(new SomeConditionNode()); // 如果条件成立，则返回成功
root->AddChild(new SomeOtherActionNode()); // 否则，尝试执行另一个行为

// 具体
BTSelector* attackSelector = new BTSelector();
// 添加近战攻击行为
attackSelector->AddChild(new MeleeAttackNode());  // 尝试近战攻击
// 添加远程攻击行为
attackSelector->AddChild(new RangedAttackNode()); // 如果近战失败，尝试远程攻击
```
BTSequence（顺序节点）
作用: BTSequence节点会按顺序执行它的子节点，直到其中一个子节点失败为止。
执行逻辑:
它依次执行每个子节点。
如果某个子节点执行失败（返回false），则BTSequence节点返回失败，停止执行后续子节点。
如果所有子节点都执行成功（返回true），则BTSequence节点返回成功。
使用场景: BTSequence通常用于实现“与”的逻辑，比如一系列必须按顺序执行的行为，只有所有行为都成功了，整个节点才会成功。
```cpp
BTSequence* root = new BTSequence();
root->AddChild(new MoveToTargetNode()); // 移动到目标
root->AddChild(new AttackTargetNode()); // 攻击目标

// 具体
BTSequence* attackSequence = new BTSequence();

// 添加搜寻目标行为
attackSequence->AddChild(new FindTargetNode());   // 搜索目标
// 添加移动到目标行为
attackSequence->AddChild(new MoveToTargetNode()); // 靠近目标
// 添加攻击目标行为
attackSequence->AddChild(new AttackTargetNode()); // 攻击目标

```
区别总结：
BTSelector 是“选择”的逻辑，任何一个子节点成功即可整体成功。
BTSequence 是“顺序”的逻辑，所有子节点成功，整体才成功。
这些复合节点可以用来构建更复杂的行为逻辑，通过组合和嵌套不同的节点类型，形成树状结构，管理游戏中的AI决策过程。

在行为树（Behavior Tree）中，BTNode和BTCompositeNode是两个重要的类，它们构成了行为树的基础结构。以下是它们的详细解释：

1. BTNode
BTNode是行为树中最基本的节点类。它通常是一个抽象类，定义了行为树节点的通用接口和行为。每个行为树节点都继承自BTNode，并实现具体的行为逻辑。
2. BTCompositeNode
BTCompositeNode是行为树中的复合节点类。它继承自BTNode，并且能够包含多个子节点。BTCompositeNode的主要职责是管理和控制其子节点的执行顺序和逻辑。

BTActionNode 是行为树（Behavior Tree）中的一个具体类型的节点，通常继承自 BTNode，并代表树中的一个叶子节点。这个节点执行一个特定的动作或行为，是行为树中最常见的叶子节点类型之一。

BTActionNode 的作用
执行具体动作：BTActionNode 用于执行具体的动作或行为，例如让角色攻击、移动、播放动画等。
叶子节点：因为 BTActionNode 不包含子节点，它通常是行为树中的叶子节点，代表最终执行的动作。
返回结果：BTActionNode 执行完毕后，会返回一个结果，指示动作是否成功、失败或进行中（一般为 Success、Failure 或 Running）。
与其他节点的关系
与 BTNode 的关系：BTActionNode 是 BTNode 的子类，专注于执行具体的动作，而不像 BTCompositeNode 那样管理多个子节点。
与 BTCompositeNode 的关系：BTActionNode 通常是 BTCompositeNode 的子节点，由 BTCompositeNode 控制其执行顺序。

BTBlackboard 是行为树（Behavior Tree）中的一个重要组件，它的作用是作为行为树和AI对象之间的共享数据存储区。具体来说：
数据存储：BTBlackboard 用于存储和管理行为树所需的数据，比如目标位置、AI状态、感知信息等。这些数据可以被行为树中的不同节点访问和更新。
数据共享：在行为树执行过程中，BTBlackboard 提供了一个集中位置，供行为树中的不同节点（如选择器、序列节点和自定义动作节点）读取和写入数据。这使得行为树的各个部分能够有效地共享和利用数据。
数据驱动决策：BTBlackboard 存储的数据可以用来驱动行为树的决策逻辑。例如，AI可以根据黑板上的数据来决定何时切换到不同的行为状态或执行某个特定的动作。
总之，BTBlackboard 使得行为树可以动态地管理和使用数据，从而实现复杂的AI行为和决策逻辑。
