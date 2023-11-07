### Casting 类型转换

You can cast Shared Pointers (and Shared References) through several support functions included in the Unreal Smart Pointer Library. Up-casting is implicit, as with C++ pointers. You can const cast with the `ConstCastSharedPtr` function, and static cast (often to downcast to derived class pointers) with `StaticCastSharedPtr`. Dynamic casting is not supported, as there is no run-type type information (RTTI); static casting should be used instead, as in the following code:
您可以通过虚幻智能指针库中包含的多个支持函数来转换共享指针（和共享引用）。**与 C++ 指针一样，向上转换是隐式的**。您可以使用 `ConstCastSharedPtr` 函数进行 const 强制转换，并使用 `StaticCastSharedPtr` 进行静态强制转换（**通常向下强制转换为派生类指针**）。**不支持动态转换**，因为没有运行类型类型信息 (RTTI)；应使用静态强制转换，如以下代码所示：

```cpp
// This assumes we validated that the FDragDropOperation is actually an FAssetDragDropOp through other means.
TSharedPtr<FDragDropOperation> Operation = DragDropEvent.GetOperation();

// We can now cast with StaticCastSharedPtr.
TSharedPtr<FAssetDragDropOp> DragDropOp = StaticCastSharedPtr<FAssetDragDropOp>(Operation);
```

