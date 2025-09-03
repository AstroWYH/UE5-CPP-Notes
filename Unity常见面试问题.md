1. **Unity 引擎相关**（基础 API、常见坑、性能优化）
2. **游戏开发通用**（对象池、内存管理、协程、网络同步等）
3. **计算机/图形学/逻辑**（和 UE 面试有点重叠）

我给你列一份 **高频 Unity 面试题 + 简答**，你能直接拿来准备。

---

## 🎯 一、Unity 引擎基础

### 1. Update / FixedUpdate / LateUpdate 的区别？

* **Update**：每帧调用，跟随帧率（游戏逻辑）。
* **FixedUpdate**：固定时间步长（物理计算，比如 Rigidbody）。
* **LateUpdate**：在所有 Update 后调用，常用于相机跟随。

---

### 2. Awake / Start 的区别？

* **Awake**：对象实例化后立即调用（即使没激活也会调用），一般用于初始化成员变量。
* **Start**：脚本第一次启用时调用（必须 Active 才触发），常用于依赖其他对象的初始化。

---

### 3. Destroy 和 SetActive(false) 区别？

* `Destroy(obj)`：真正销毁 GameObject，下次要用只能重新 Instantiate（开销大）。
* `obj.SetActive(false)`：只是禁用对象，仍在内存中，下次启用更快（常配合对象池）。

---

### 4. Prefab 是什么？有什么用？

* Prefab 是一种 **可复用的资源模板**，包含模型、脚本、参数等。
* 优点：统一修改、可实例化、方便管理。
* 举例：子弹、敌人、特效都用 Prefab 管理。

---

### 5. Unity 中常见优化手段？

* **对象池** → 减少 Instantiate/Destroy。
* **减少 Update** → 合并逻辑，避免在每个对象里写 Update。
* **合批 (Batching)** → Static Batching / Dynamic Batching，减少 DrawCall。
* **内存优化** → 避免频繁装箱拆箱、减少 GC。
* **贴图/模型优化** → 压缩贴图，减少顶点数。
* **异步加载** → AssetBundle/Addressables。

---

## 🎯 二、游戏开发相关

### 6. Unity 协程 (Coroutine) 是什么？和线程的区别？

* 协程是 Unity 提供的一种 **逻辑切分方式**，不是线程，依赖 Unity 的主线程调度。
* 写法：`StartCoroutine(MyRoutine())`。
* 区别：协程不会开新线程，只是把任务挂起、分帧执行，适合加载资源、等待计时。

---

### 7. MonoBehaviour 生命周期？

1. **Awake**
2. **OnEnable**
3. **Start**
4. **Update** / **FixedUpdate**
5. **LateUpdate**
6. **OnDisable**
7. **OnDestroy**

---

### 8. 如何实现一个对象池？

常见思路：

* 预先创建对象，放到 List/Queue 中并禁用。
* 需要时取出一个启用，用完再禁用放回。

👉 面试官常考你能否写出一个 **子弹池**（我刚才给你写的那个 `SimpleBulletPool` 就是标准答案）。

---

### 9. Unity 的内存分配和 GC 问题？

* C# 有垃圾回收器（GC），频繁产生临时对象会引发 GC，造成卡顿。
* 常见的 GC 来源：

  * string 拼接（频繁 new 字符串）
  * foreach 遍历 List（装箱/迭代器分配）
  * new 出来的临时 Vector3/Color 等
* 优化：

  * 用对象池代替频繁 new
  * 尽量少用字符串拼接（用 StringBuilder）
  * 使用 for 循环代替 foreach

---

### 10. Unity 的射线检测怎么做？

```csharp
Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
RaycastHit hit;
if (Physics.Raycast(ray, out hit))
{
    Debug.Log("Hit: " + hit.collider.name);
}
```

用途：点击检测、子弹碰撞检测、寻路。

---

## 🎯 三、进阶/项目经验相关

### 11. Unity 和 UE 的主要区别？

* **语言**：Unity 用 C#，UE 用 C++。
* **渲染管线**：Unity 灵活（URP/HDRP 可选），UE 内置渲染更强大。
* **编辑器生态**：Unity 插件丰富，2D/移动端更强；UE 在 3D/大作上更成熟。
* **热更新**：Unity 可以用 ILRuntime、HybridCLR；UE 热更新比较复杂。

---

### 12. 如果做一款卡牌/棋牌类游戏，Unity 里需要注意什么？

（途游很可能问这个 😎）

* UI 系统优化（UGUI 或者改造过的 UI 框架）。
* 对象池管理卡牌、特效。
* 内存优化，避免 GC。
* 断线重连/网络同步（途游棋牌类必考）。
* 局部热更新（发新牌型、特效时无需重装包）。

---

### 13. Unity 如何做热更新？

* Unity C# 编译成 IL 代码，默认不支持热更。
* 常见方案：

  * **ILRuntime**：解释执行 DLL。
  * **HybridCLR**：真 CLR 替换方案。
  * **Lua 框架（xlua, tolua）**：逻辑写 Lua，Unity 调用。
* 棋牌/休闲类游戏里非常常见，能快速上线修复 bug。

---

### 14. 如何减少 DrawCall？

* 合并静态物体 → Static Batching。
* 合并动态物体 → Dynamic Batching / GPU Instancing。
* 使用图集 (Sprite Atlas, Texture Atlas)。
* 避免 UI 元素过多层级和 Mask。

---

### 15. Unity 的序列化系统（SerializeField/ScriptableObject）？

* `[SerializeField]`：让私有字段也能在 Inspector 面板显示。
* **ScriptableObject**：一种特殊的资产类型，适合存放配置数据（比 Prefab 更轻量）。
  👉 常考点：用 ScriptableObject 做配置表，而不是写死在代码里。

---

## 🧩 建议你准备的方向

* **对象池**：能手写出来，讲清楚为什么要这样做。
* **协程 vs 线程**：一定会问。
* **GC/性能优化**：讲几个实际例子（比如 UI 大量生成、频繁字符串拼接）。
* **网络同步/断线重连**：棋牌类公司爱问。
* **热更新**：途游会考。

---

