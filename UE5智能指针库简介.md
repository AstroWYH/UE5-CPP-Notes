### **[Shared Pointers](https://docs.unrealengine.com/5.3/en-US/shared-pointers-in-unreal-engine)** (`TSharedPtr`)

A Shared Pointer owns the object it references, indefinitely preventing deletion of that object, and ultimately handling its deletion when no Shared Pointer or Shared Reference (see below) references it. A Shared Pointer can be empty, meaning it doesn't reference any object. Any non-null Shared Pointer can produce a Shared Reference to the object it references.
共享指针拥有它引用的对象，无限期地防止删除该对象，并最终**在没有共享指针或共享引用**（见下文）**引用它时**处理其**删除**。**共享指针可以为空，这意味着它不引用任何对象**。任何非空共享指针都可以生成对其引用的对象的共享引用。

### **[Shared References](https://docs.unrealengine.com/5.3/en-US/shared-references-in-unreal-engine)** (`TSharedRef`)

A Shared Reference acts like a Shared Pointer, in the sense that it owns the object it references. They differ with regard to null objects; Shared References must always reference a non-null object. Because Shared Pointers don't have that restriction, a Shared Reference can always be converted to a Shared Pointer, and that Shared Pointer is guaranteed to reference a valid object. Use Shared References when you want a guarantee that the referenced object is non-null, or if you want to indicate shared object ownership.
共享引用的作用类似于共享指针，因为它拥有它所引用的对象。它们在空对象方面有所不同；**共享引用必须始终引用非空对象**。由于**共享指针没有该限制**，因此共享引用始终可以转换为共享指针，并且保证该共享指针引用有效的对象。当您**想要保证引用的对象不为空**，或者想要指示共享对象所有权时，请使用共享引用。

### **[Weak Pointers](https://docs.unrealengine.com/5.3/en-US/weak-pointers-in-unreal-engine)** (`TWeakPtr`)

Weak Pointers are similar to Shared Pointers, but do not own the object they reference, and therefore do not affect its lifecycle. This property can be very useful, as it breaks reference cycles, but it also means that a Weak Pointer can become null at any time, without warning. For this reason, a Weak Pointer can produce a Shared Pointer to the object it references, ensuring programmers safe access to the object on a temporary basis.
弱指针与共享指针类似，但**不拥有它们引用的对象**，因此**不会影响其生命周期**。这个属性非常有用，因为它**打破了引用循环**，但它也意味着弱指针**可以随时变为 null，而不会发出警告**。因此，**弱指针可以生成指向它引用的对象的共享指针**，确保程序员临时安全地访问该对象。

### **Unique Pointers** (`TUniquePtr`)

A Unique Pointer solely and explicitly owns the object it references. Since there can only be one Unique Pointer to a given resource, Unique Pointers can transfer ownership, but cannot share it. Any attempts to copy a Unique Pointer will result in a compile error. When a Unique Pointer goes out of scope, it will automatically delete the object it references.
唯一指针单独且显式地拥有它引用的对象。由于指向给定资源的唯一指针只能有一个，因此唯一指针可以**转移**所有权，但不能共享它。**任何复制唯一指针的尝试都会导致编译错误**。当唯一指针超出范围时，它将自动删除它引用的对象。

Making a Shared Pointer or Shared Reference to an object that a Unique Pointer references is dangerous. This will not suspend the Unique Pointer's behavior of deleting the object upon its own destruction, even though other Smart Pointers still reference it. Similarly, you should not make a Unique Pointer to an object that is referenced by a Shared Pointer or Shared Reference.
**对唯一指针引用的对象创建共享指针或共享引用是危险的**。这**不会中止唯一指针在其自身销毁时删除该对象的行为**，即使其他智能指针仍然引用它。同样，您不应将唯一指针指向由共享指针或共享引用引用的对象。

```
对`TUniquePtr`创建`TSharedPtr`或`TSharedRef`，可能导致重复释放内存。
```