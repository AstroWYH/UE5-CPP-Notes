在 Godot 中获取其他场景的节点，需要通过 **场景实例化** 或 **全局引用** 实现。以下是 5 种常见方法，按使用频率排序：

---

### ⭐️ 方法 1：实例化后获取（最推荐）
```gdscript
# 1. 加载场景资源
var other_scene = preload("res://path/to/OtherScene.tscn")

# 2. 实例化并添加到当前场景
var other_scene_instance = other_scene.instantiate()
add_child(other_scene_instance)

# 3. 获取目标节点（假设节点名为 "TargetNode"）
var target_node = other_scene_instance.get_node("TargetNode")
# 或简写：
var target_node = other_scene_instance.get_node("%TargetNode")  # 如果设置了唯一名称
```

---

### 🌍 方法 2：通过全局自动加载（Autoload）
1. **创建全局脚本**  
   新建 `Global.gd`，添加到 `Project > Project Settings > Autoload`  
2. **暴露节点引用**  
   ```gdscript
   # Global.gd
   var other_scene_node: Node

   func register_node(node: Node):
       other_scene_node = node
   ```
3. **在其他场景中注册**  
   ```gdscript
   # OtherScene.gd
   func _ready():
       Global.register_node($TargetNode)
   ```
4. **任意场景获取**  
   ```gdscript
   var target_node = Global.other_scene_node
   ```

---

### 🔗 方法 3：信号通信（解耦推荐）
```gdscript
# 1. 在目标场景定义信号
# OtherScene.gd
signal node_reference_requested(node: Node)

func _ready():
    emit_signal("node_reference_requested", $TargetNode)

# 2. 在获取方连接信号
func _on_OtherScene_node_reference_requested(node):
    print("获取到节点:", node.name)
```

---

### 🏷️ 方法 4：节点组（Group）
1. **给目标节点添加分组**  
   ```gdscript
   # OtherScene.gd
   func _ready():
       $TargetNode.add_to_group("ImportantNodes")
   ```
2. **任意场景获取组内节点**  
   ```gdscript
   var nodes = get_tree().get_nodes_in_group("ImportantNodes")
   var target_node = nodes[0] if nodes else null
   ```

---

### 🗺️ 方法 5：绝对路径（慎用）
```gdscript
# 假设 OtherScene 是主场景的子节点
var target_node = get_node("/root/MainScene/OtherScene/TargetNode")
```
⚠️ 注意：硬编码路径在场景结构调整时会失效

---

### 📊 方法对比
| 方法       | 实时性 | 解耦性 | 适用场景                 |
|------------|--------|--------|--------------------------|
| 实例化     | ✅      | ❌      | 需要立即操作的动态场景   |
| 全局Autoload| ✅      | ✅      | 高频访问的核心节点       |
| 信号       | ❌      | ✅      | 跨场景异步通信           |
| 节点组     | ✅      | ✅      | 批量操作同类节点         |
| 绝对路径   | ✅      | ❌      | 快速原型开发（不推荐正式使用） |

---

### 💡 最佳实践建议
1. **优先用方法1**（实例化后获取）  
   - 适用于动态加载的场景（如弹窗、敌人等）

2. **长期存在的核心节点用方法2**  
   - 如玩家角色、游戏管理器等

3. **需要解耦时用方法3**  
   - 适合UI系统与游戏逻辑的交互

4. **避免方法5**  
   - 绝对路径会使代码脆弱难维护

---

### 🌰 完整示例（方法1 + 方法2结合）
```gdscript
# Global.gd (Autoload)
var player: Node2D

# PlayerScene.gd
func _ready():
    Global.player = self

# 任意其他脚本中
func _on_button_pressed():
    if Global.player:
        Global.player.heal(10)  # 调用玩家方法
    else:
        var player_scene = preload("res://Player.tscn").instantiate()
        get_tree().root.add_child(player_scene)
```
