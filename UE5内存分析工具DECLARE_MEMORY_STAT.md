## stat LLM
首先，可以直接通过stat LLM和stat LLMFULL粗略看出增长。
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/4b9b8797-7b0b-4e9c-8500-35bb8722f833)
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/091d0be1-b31d-418e-9dba-7c14f3f93d13)
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/8169899e-876c-4692-8c6e-bb127742076e)
统计数据系统支持下列类型：

| 统计数据类型         | 说明                                                       |
| :------------------- | :--------------------------------------------------------- |
| **循环计数器**       | 一种泛型循环计数器，用于统计数据对象生命周期中的循环次数。 |
| **浮点/Dword计数器** | 一种每帧都会清空的计数器。                                 |
| **浮点/Dword累加器** | 一种不会每帧清空的计数器，作为可重置的持久统计数据。       |
| **内存**             | 一种特殊类型的计数器，针对内存跟踪进行优化。               |

## 分组统计数据

每个统计数据必须归入组中，通常对应显示指定的统计数据组。例如，**stat statsystem** 将显示统计数据相关数据。

要定义统计数据组，请使用下列方法之一：

| 方法                                                         | 说明                                           |
| :----------------------------------------------------------- | :--------------------------------------------- |
| `DECLARE_STATS_GROUP(GroupDesc, GroupId, GroupCat)`          | 声明默认启用的统计数据组。                     |
| `DECLARE_STATS_GROUP_VERBOSE(GroupDesc, GroupId, GroupCat)`  | 声明默认禁用的统计数据组。                     |
| `DECLARE_STATS_GROUP_MAYBE_COMPILED_OUT(GroupDesc, GroupId, GroupCat)` | 声明默认禁用的统计数据组，编译器可将该组剥离。 |

- `GroupDesc` 是该组的文本描述。
- `GroupId` 是该组的 `独有` 标识
- `GroupCat` 保留供将来使用
- `CompileIn` 如设为true，编译器则可能将其剥离出来

```c
DECLARE_STATS_GROUP(TEXT("Threading"), STATGROUP_Threading, STATCAT_Advanced);
DECLARE_STATS_GROUP_VERBOSE(TEXT("Linker Load"), STATGROUP_LinkerLoad, STATCAT_Advanced);
```

## 声明和定义统计数据

现在可声明和定义统计数据，但在此之前请注意，统计数据可用在：

- 仅一个cpp文件
- 函数作用域
- 模块作用域
- 整个项目

### 用于单个文件（省略）

如作用域是单个文件，必须根据统计数据类型使用下列一种方法：

| `DECLARE_CYCLE_STAT(CounterName, StatId, GroupId)`  | 声明循环计数器统计数据。                                     |
| --------------------------------------------------- | ------------------------------------------------------------ |
| `DECLARE_MEMORY_STAT(CounterName, StatId, GroupId)` | 声明与dword累加器相同的内存计数器，但将使用内存特定单位显示。 |

### 用于多个文件

如想拥有可供整个项目（或范围较广的文件）访问的统计数据，需使用其外部版本。 这些方法与先前提到的方法相同，但名称末尾带有 `_EXTERN`：

```
DECLARE_CYCLE_STAT_EXTERN(CounterName, StatId, GroupId, API)`
`DECLARE_FLOAT_COUNTER_STAT_EXTERN(CounterName, StatId, GroupId, API)`
`DECLARE_DWORD_COUNTER_STAT_EXTERN(CounterName, StatId, GroupId, API)`
`DECLARE_FLOAT_ACCUMULATOR_STAT_EXTERN(CounterName, StatId, GroupId, API)`
`DECLARE_DWORD_ACCUMULATOR_STAT_EXTERN(CounterName, StatId, GroupId, API)`
`DECLARE_MEMORY_STAT_EXTERN(CounterName, StatId, GroupId, API)`
`DECLARE_MEMORY_STAT_POOL_EXTERN(CounterName, StatId, GroupId, Pool, API)
```

然后在源文件中，需使用下列项定义这些统计数据，这些项定义的以 `_EXTERN` 声明的统计数据：

其中：

- `CounterName` 是统计数据的文本描述
- `StatId` 是统计数据的 `独有` 标识
- `GroupId` 是统计数据所属的组的标识， GroupId` 来自 `DECLARE_STATS_GROUP*`
- `Pool` 是平台专属的内存池
- `API` 是模块的 `*_API`，如该统计数据仅使用在该模块中，其可为空

```c
DECLARE_MEMORY_STAT(TEXT("Total Physical"), STAT_TotalPhysical, STATGROUP_MemoryPlatform);
DECLARE_MEMORY_STAT(TEXT("Total Virtual"), STAT_TotalVirtual, STATGROUP_MemoryPlatform);
DECLARE_MEMORY_STAT(TEXT("Page Size"), STAT_PageSize, STATGROUP_MemoryPlatform);
DECLARE_MEMORY_STAT(TEXT("Total Physical GB"), STAT_TotalPhysicalGB, STATGROUP_MemoryPlatform);
```
可以自定义STAT GROUP，然后在分配处INC_MEMORY_STAT_BY(STAT_XXX, sizeof(SomeClass))，在释放出用DEC_MEMORY_STAT_BY(STAT_XXX, sizeof(SomeClass))
