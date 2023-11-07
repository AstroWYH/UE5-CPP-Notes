- Avoid passing data to functions as `TSharedRef` or `TSharedPtr` parameters where possible, as these will incur overhead by dereferencing and reference-counting. Instead, pass the referenced object, preferably as a `const &`.
  尽可能**避免**将数据作为 **`TSharedRef` 或 `TSharedPtr` 参数传递给函数**，因为这些会因取消引用和引用计数而**产生开销**。相反，传递引用的对象，最好作为 `const &` 。
- You can forward-declare Shared Pointers to incomplete types.
  您可以将共享指针 **前向声明** 为不完整类型。
- Shared Pointers are not compatible with Unreal objects (`UObject` and its derived classes). The Engine has a separate memory management system (see [Object Handling](https://docs.unrealengine.com/5.3/en-US/unreal-object-handling-in-unreal-engine) documentation) for `UObject` management, and the two systems have no overlap with each other.
  **共享指针与虚幻对象（ `UObject` 及其派生类）不兼容。引擎有一个单独的内存管理系统（请参阅对象处理文档）用于 `UObject` 管理，并且两个系统彼此没有重叠。**

