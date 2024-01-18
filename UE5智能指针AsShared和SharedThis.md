# TSharedFromThis

Shared pointers are non-intrusive, which means the object does not know whether or not a Smart Pointer owns it. This is usually acceptable, but there may be cases in which you want to access the object as a Shared Reference or Shared Pointer. To do this, derive the object's class from `TSharedFromThis`, using the object's class as the template parameter. `TSharedFromThis` provides two functions, `AsShared` and `SharedThis`, that can convert the object to a Shared Reference (and from there, to a Shared Pointer). This can be useful with class factories that always return Shared References, or when you need to pass your object to a system that expects a Shared Reference or Shared Pointer. `AsShared` will return your class as the type originally passed as the template argument to `TSharedFromThis`, which may be a parent type to the calling object, while `SharedThis` will derive the type directly from this and return a Smart Pointer referencing an object of that type. The following example code demonstrates both functions:
共享指针是**非侵入式**的，这意味着**对象不知道智能指针是否拥有**它。这通常是可以接受的，但在**某些情况下**，您可能**希望将对象作为共享引用或共享指针来访问**。为此，**请使用对象的类作为模板参数，从 `TSharedFromThis` 派生对象的类**。 `TSharedFromThis` 提供了两个函数 **`AsShared`** 和 **`SharedThis`** ，它们**可以将对象转换为共享引用（并从那里转换为共享指针）**。这对于始终返回共享引用的类工厂很有用，或者当您需要将对象传递到需要共享引用或共享指针的系统时。 `AsShared` 将返回您的类，作为最初作为模板参数传递给 `TSharedFromThis` 的类型，它可能是调用对象的父类型，而 `SharedThis` 将派生直接从中获取类型并返回引用该类型对象的智能指针。以下示例代码演示了这两个功能：

### AsShared() 和 SharedThis(this)

```cpp
class FRegistryObject;
class FMyBaseClass: public TSharedFromThis<FMyBaseClass>
{
    virtual void RegisterAsBaseClass(FRegistryObject* RegistryObject)
    {
        // Access a shared reference to 'this'.
        // We are directly inherited from <TSharedFromThis> , so AsShared() and SharedThis(this) return the same type.
        TSharedRef<FMyBaseClass> ThisAsSharedRef = AsShared();
        // 返回父类TSharedRef<FMyBaseClass>
        // RegistryObject expects a TSharedRef<FMyBaseClass>, or a TSharedPtr<FMyBaseClass>. TSharedRef can implicitly be converted to a TSharedPtr.
        RegistryObject->Register(ThisAsSharedRef);
    }
};
class FMyDerivedClass : public FMyBaseClass
{
    virtual void Register(FRegistryObject* RegistryObject) override
    {
        // We are not directly inherited from TSharedFromThis<>, so AsShared() and SharedThis(this) return different types.
        // AsShared() will return the type originally specified in TSharedFromThis<> - TSharedRef<FMyBaseClass> in this example.
        // SharedThis(this) will return a TSharedRef with the type of 'this' - TSharedRef<FMyDerivedClass> in this example.
        // The SharedThis() function is only available in the same scope as the 'this' pointer.
        TSharedRef<FMyDerivedClass> AsSharedRef = SharedThis(this);
        // 返回子类TSharedRef<FMyDerivedClass>
        // 如果用AsShared，则返回父类TSharedRef<FMyBaseClass>
        // RegistryObject will accept a TSharedRef<FMyDerivedClass> because FMyDerivedClass is a type of FMyBaseClass.
        RegistryObject->Register(ThisAsSharedRef);
    }
};
class FRegistryObject
{
    // This function will accept a TSharedRef or TSharedPtr to FMyBaseClass or any of its children.
    void Register(TSharedRef<FMyBaseClass>);
};
```

[Smart Pointers in Unreal Engine | Unreal Engine 5.3 Documentation --- 虚幻引擎中的智能指针|虚幻引擎 5.3 文档](https://docs.unrealengine.com/5.3/en-US/smart-pointers-in-unreal-engine/)
