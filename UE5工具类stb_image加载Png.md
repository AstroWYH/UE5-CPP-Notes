- Avoid passing data to functions as `TSharedRef` or `TSharedPtr` parameters where possible, as these will incur overhead by dereferencing and reference-counting. Instead, pass the referenced object, preferably as a `const &`.
  尽可能**避免**将数据作为 **`TSharedRef` 或 `TSharedPtr` 参数传递给函数**，因为这些会因取消引用和引用计数而**产生开销**。相反，传递引用的对象，最好作为 `const &` 。
- You can forward-declare Shared Pointers to incomplete types.
  您可以将共享指针 **前向声明** 为不完整类型。
- Shared Pointers are not compatible with Unreal objects (`UObject` and its derived classes). The Engine has a separate memory management system (see [Object Handling](https://docs.unrealengine.com/5.3/en-US/unreal-object-handling-in-unreal-engine) documentation) for `UObject` management, and the two systems have no overlap with each other.
  **共享指针与虚幻对象（ `UObject` 及其派生类）不兼容。引擎有一个单独的内存管理系统（请参阅对象处理文档）用于 `UObject` 管理，并且两个系统彼此没有重叠。**

| Benefit                                   | Description 描述                                             |
| :---------------------------------------- | :----------------------------------------------------------- |
| **Prevents memory leaks 防止内存泄漏**    | Smart Pointers (other than Weak Pointers) automatically delete objects when there are no more shared references. **当不再有共享引用时，智能指针（弱指针除外）会自动删除对象**。 |
| **Weak referencing 弱引用**               | Weak Pointers break reference cycles and prevent dangling pointers. **弱指针会破坏引用循环并防止悬空指针**。 |
| **Optional Thread safety 可选的线程安全** | The Unreal Smart Pointer Library includes thread-safe code that manages reference counting across multiple threads. **Thread safety** can be traded out for better performance if it isn't needed. 虚幻智能指针库包含管理跨多个线程的引用计数的**线程安全代码**（后面看下其实现）。如果不需要，可以用线程安全来换取更好的性能。 |
| **Runtime safety 运行时安全**             | Shared References are never null and can always be dereferenced. **共享引用永远不会为空**，并且**始终可以取消引用（可以取消？？？）。** |
| **Confers intent 赋予意图**               | You can easily tell an object owner from an observer. 您可以轻松区分对象所有者和观察者。 |
| **Memory 内存**                           | Smart Pointers are only twice the size of a C++ pointer in 64-bit (plus a shared 16-byte reference controller). The exception to this is Unique Pointers, which are the same size as C++ pointers. **智能指针的大小仅为 64 位 C++ 指针的两倍**（加上共享的 16 字节参考控制器）。**唯一指针**是个例外，它的**大小与 C++ 指针相同**。 |

By default, Smart Pointers are only safe to access on a single thread. If you need multiple threads to have access, use the thread-safe versions of Smart Pointer classes:
默认情况下，智能指针只能在单个线程上安全访问。**如果您需要多个线程进行访问**，请使用智能指针类的线程安全版本：

- **`TSharedPtr<T, ESPMode::ThreadSafe>`**
- **`TSharedRef<T, ESPMode::ThreadSafe>`**
- **`TWeakPtr<T, ESPMode::ThreadSafe>`**
- **`TSharedFromThis<T, ESPMode::ThreadSafe>`**

由于原子引用计数，这些**线程安全版本比默认版本慢一些**，但它们的行为与常规 C++ 指针一致：

- Reads and copies are always thread-safe.
  读取和复制始终是线程安全的。
- Writes and resets must be synchronized to be safe.
  **写入和重置必须同步才能安全**。

If you know your pointer will never be accessed by more than one thread, you can get better performance by avoiding the thread-safe versions.
如果您知道您的指针永远不会被多个线程访问，则可以**通过避免线程安全版本来获得更好的性能**。

[Smart Pointers in Unreal Engine | Unreal Engine 5.3 Documentation --- 虚幻引擎中的智能指针|虚幻引擎 5.3 文档](https://docs.unrealengine.com/5.3/en-US/smart-pointers-in-unreal-engine/)

### Casting 类型转换

You can cast Shared Pointers (and Shared References) through several support functions included in the Unreal Smart Pointer Library. Up-casting is implicit, as with C++ pointers. You can const cast with the `ConstCastSharedPtr` function, and static cast (often to downcast to derived class pointers) with `StaticCastSharedPtr`. Dynamic casting is not supported, as there is no run-type type information (RTTI); static casting should be used instead, as in the following code:
您可以通过虚幻智能指针库中包含的多个支持函数来转换共享指针（和共享引用）。**与 C++ 指针一样，向上转换是隐式的**。您可以使用 `ConstCastSharedPtr` 函数进行 const 强制转换，并使用 `StaticCastSharedPtr` 进行静态强制转换（**通常向下强制转换为派生类指针**）。**不支持动态转换**，因为没有运行类型类型信息 (RTTI)；应使用静态强制转换，如以下代码所示：

```cpp
// This assumes we validated that the FDragDropOperation is actually an FAssetDragDropOp through other means.
TSharedPtr<FDragDropOperation> Operation = DragDropEvent.GetOperation();

// We can now cast with StaticCastSharedPtr.
TSharedPtr<FAssetDragDropOp> DragDropOp = StaticCastSharedPtr<FAssetDragDropOp>(Operation);
```

| Helper                                                       | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| **Classes 类**                                               |                                                              |
| `TSharedFromThis`                                            | Deriving your class from `TSharedFromThis` adds the `AsShared` or `SharedThis` functions. These functions enable you to acquire a `TSharedRef` to your object. **从 `TSharedFromThis` 派生类**会添加 **`AsShared`** 或 **`SharedThis`** 函数。这些函数使您**能够获取对象的 `TSharedRef`** 。 |
| **Functions 函数**                                           |                                                              |
| `MakeShared` and `MakeShareable` `MakeShared` 和 `MakeShareable` | Creates a Shared Pointer from a regular C++ pointer. `MakeShared` allocates a new object instance and the reference controller in a single memory block, but requires the object to offer a public constructor. `MakeShareable` is less efficient, but works even if the object's constructor is private, enables you to take ownership of an object you didn't create, and supports customized behavior when deleting the object. 从常规 C++ 指针创建共享指针。 `MakeShared` 在单个内存块中分配一个新的对象实例和引用控制器，但要求该对象提供一个公共构造函数。 **`MakeShareable` 效率较低**，但即使对象的**构造函数是私有的，也可以工作**，使您能够获得未创建的对象的所有权，并支持删除对象时的自定义行为。 |
| `StaticCastSharedRef` and `StaticCastSharedPtr` `StaticCastSharedRef` 和 `StaticCastSharedPtr` | Static cast utility function, typically used to downcast to a derived type. 静态转换实用函数，**通常用于向下转换为派生类型。** |
| `ConstCastSharedRef` and `ConstCastSharedPtr` `ConstCastSharedRef` 和 `ConstCastSharedPtr` | Converts a `const` Smart Reference or Smart Pointer to a `mutable` Smart Reference or Smart Pointer, respectively. **将 `const`** 智能引用或智能指针分别**转换为 `mutable**` 智能引用或智能指针。 |
