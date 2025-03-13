在 **Unreal Engine (UE)** 中，`GetTypeHash` 是一个用于获取对象哈希值的函数。它的主要作用是为对象生成一个唯一的哈希值，以便在哈希表（如 `TMap` 或 `TSet`）中快速查找和比较对象。

### `GetTypeHash` 的作用
1. **哈希表支持**  
   `GetTypeHash` 是为自定义类型实现哈希化的关键函数。UE 的 `TMap` 和 `TSet` 等容器依赖哈希值来高效地存储和查找数据。如果自定义类型需要在哈希表中使用，必须为其实现 `GetTypeHash`。

2. **快速查找和比较**  
   哈希值用于在哈希表中快速定位对象，减少查找时间。通过 `GetTypeHash` 生成的哈希值，容器可以快速确定对象的存储位置。

3. **唯一性**  
   一个好的哈希函数应尽可能为不同的对象生成不同的哈希值，以减少哈希冲突。`GetTypeHash` 的设计目标是为不同的输入生成唯一的哈希值。

### `GetTypeHash` 的实现
`GetTypeHash` 通常针对特定类型进行重载。UE 已经为基本类型（如 `int32`、`FString` 等）提供了默认实现。对于自定义类型，开发者需要手动实现 `GetTypeHash`。

#### 示例：为自定义类型实现 `GetTypeHash`
```cpp
struct FMyCustomType
{
    int32 Id;
    FString Name;
};

// 实现 GetTypeHash
uint32 GetTypeHash(const FMyCustomType& MyType)
{
    // 使用 Id 和 Name 的哈希值组合生成自定义类型的哈希值
    return HashCombine(GetTypeHash(MyType.Id), GetTypeHash(MyType.Name));
}
```

在这个例子中：
- `GetTypeHash` 为 `FMyCustomType` 生成哈希值。
- `HashCombine` 是 UE 提供的工具函数，用于将多个哈希值组合成一个新的哈希值。

### `GetTypeHash` 的使用场景
1. **在 `TMap` 中作为键**  
   如果自定义类型用作 `TMap` 的键，必须为其实现 `GetTypeHash`。

   ```cpp
   TMap<FMyCustomType, int32> MyMap;
   FMyCustomType Key = {1, "Example"};
   MyMap.Add(Key, 100);
   ```

2. **在 `TSet` 中存储**  
   如果自定义类型存储在 `TSet` 中，也需要实现 `GetTypeHash`。

   ```cpp
   TSet<FMyCustomType> MySet;
   FMyCustomType Element = {1, "Example"};
   MySet.Add(Element);
   ```

### 总结
`GetTypeHash` 的作用是为对象生成哈希值，以便在哈希表中高效存储和查找。对于自定义类型，开发者需要为其实现 `GetTypeHash`，以便支持 UE 的哈希容器（如 `TMap` 和 `TSet`）。通过合理设计哈希函数，可以确保哈希值的唯一性和性能。

- 如果不为自定义类型实现 `GetTypeHash`，将其放入 `TMap` 或 `TSet` 中会导致编译错误，而不是仅仅查找变慢。
- `GetTypeHash` 是哈希容器的核心要求，必须为自定义类型实现该函数。
- 实现 `GetTypeHash` 后，哈希容器的查找、插入和删除操作会非常高效（时间复杂度为 **O(1)**）。

- 继承自 `UObject` 的自定义类可以直接用于 `TMap` 或 `TSet`，因为 `UObject` 已经实现了 `GetTypeHash`。
- `UObject` 的 `GetTypeHash` 通常基于对象的指针地址生成哈希值。
- 在哈希容器中，通常使用 `UObject` 的指针（如 `UMyObject*`）作为键。
- 非 `UObject` 类仍需手动实现 `GetTypeHash` 以支持哈希容器。