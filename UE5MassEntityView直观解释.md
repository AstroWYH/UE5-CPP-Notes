## Entity & Fragment

把每个实体Entity看作学生，每个 Fragment 看作学生的科目成绩，比如：

- **数学成绩**是一个 Fragment 类型。(注：其实数学成绩可以是一个更复杂的类，比如FPawnBaseInfo)
- **英语成绩**是另一个 Fragment 类型。

如果存储所有学生的信息，传统的方式是按学生分组：

```
学生 1：{数学: 90, 英语: 85}
学生 2：{数学: 75, 英语: 95}
学生 3：{数学: 80, 英语: 88}
```

而在 Mass Entity 系统中，它会按科目（Fragment 类型）分组存储：

```
数学成绩数组： [90, 75, 80]  （按学生的顺序排列）
英语成绩数组： [85, 95, 88]
```

这样，如果你需要所有学生的数学成绩，就可以直接访问数学成绩数组，而不需要遍历每个学生。

分组存储的核心思想是：

1. 按 **Fragment 类型** 分组存储数据，每种类型有一个自己的数组。
2. 数据的顺序对应实体的顺序，**通过索引把同一实体的不同数据关联起来**。
3. 这样设计可以**大幅提高性能**，特别是在大规模实体操作中，可以使用SIMD指令。**提高了cache命中率**，数据都在相邻内存上。



## Archetype 的角色

Archetype 就是管理一组学生的成绩（Fragment 类型集合）的容器，它确保每一组学生（每个实体）都有**相同的科目成绩**。

**Archetype 1（`数学 + 英语`）**：

- 它管理的是一组学生，他们都选择了 `数学` 和 `英语`。
- 存储的内容是：
  - 数学成绩：`[90, 75]`（对应学生 1 和 2）
  - 英语成绩：`[85, 95]`（对应学生 1 和 2）
- 这两科的成绩都存储在两个不同的数组中。

```cpp
void* FMassEntityView::GetFragmentPtrChecked(const UScriptStruct& FragmentType) const
{
	checkSlow(Archetype && EntityHandle.IsValid());
	const int32 FragmentIndex = Archetype->GetFragmentIndexChecked(&FragmentType);
	return Archetype->GetFragmentData(FragmentIndex, EntityHandle);
}

	FORCEINLINE void* GetFragmentData(const int32 FragmentIndex, const FMassRawEntityInChunkData EntityIndex) const
	{
		return FragmentConfigs[FragmentIndex].GetFragmentData(EntityIndex.ChunkRawMemory, EntityIndex.IndexWithinChunk);
	}
```



## 为什么不能缓存 `FMassEntityView`

`FMassEntityView` 是动态生成的，意味着它是根据当前的实体状态（学生选择的科目、成绩等）**实时计算**的。它并不存储实际的数据，而是提供一个对数据的访问接口。因此，缓存它意味着你可能会在未来的某一时刻访问到一个过时的视图，导致错误的数据访问。