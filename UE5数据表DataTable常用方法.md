在 Unreal Engine 中，`UDataTable` 是一个用于存储结构化数据的强大工具，通常与 `USTRUCT` 结合使用。以下是 `UDataTable` 的常用方法，特别是如何查找某一行数据：

------

### 1. **查找某一行数据**

- **方法**：`FindRow`

- **作用**：根据行名（`RowName`）查找对应的数据。

- **返回值**：如果找到，返回指向数据的指针；如果未找到，返回 `nullptr`。

- **示例**：

  cpp

  复制

  ```
  UDataTable* MyDataTable = ...; // 获取 DataTable
  FName RowName = TEXT("Row1");  // 行名
  FMixAttributeData* RowData = MyDataTable->FindRow<FMixAttributeData>(RowName, TEXT("Context"));
  if (RowData)
  {
      // 使用找到的数据
      UE_LOG(LogTemp, Warning, TEXT("Health: %d"), RowData->Health);
  }
  else
  {
      UE_LOG(LogTemp, Warning, TEXT("Row not found!"));
  }
  ```

------

### 2. **获取所有行数据**

- **方法**：`GetRowMap`

- **作用**：获取 DataTable 中所有行的映射（`TMap`）。

- **返回值**：一个 `TMap<FName, uint8*>`，其中键是行名，值是指向数据的指针。

- **示例**：

  cpp

  复制

  ```
  UDataTable* MyDataTable = ...; // 获取 DataTable
  const TMap<FName, uint8*>& RowMap = MyDataTable->GetRowMap();
  
  for (const auto& Pair : RowMap)
  {
      FName RowName = Pair.Key;
      FMixAttributeData* RowData = reinterpret_cast<FMixAttributeData*>(Pair.Value);
      UE_LOG(LogTemp, Warning, TEXT("Row Name: %s, Health: %d"), *RowName.ToString(), RowData->Health);
  }
  ```

------

### 3. **获取行名列表**

- **方法**：`GetRowNames`

- **作用**：获取 DataTable 中所有行的名称列表。

- **返回值**：一个 `TArray<FName>`，包含所有行名。

- **示例**：

  cpp

  复制

  ```
  UDataTable* MyDataTable = ...; // 获取 DataTable
  TArray<FName> RowNames = MyDataTable->GetRowNames();
  
  for (const FName& RowName : RowNames)
  {
      UE_LOG(LogTemp, Warning, TEXT("Row Name: %s"), *RowName.ToString());
  }
  ```

------

### 4. **添加或修改行数据**

- **方法**：`AddRow` 或 `RemoveRow`

- **作用**：动态添加或删除行数据。

- **示例**：

  cpp

  复制

  ```
  UDataTable* MyDataTable = ...; // 获取 DataTable
  FName RowName = TEXT("NewRow");
  FMixAttributeData NewRowData;
  NewRowData.Health = 100;
  NewRowData.Speed = 10;
  
  MyDataTable->AddRow(RowName, NewRowData); // 添加新行
  MyDataTable->RemoveRow(RowName);         // 删除行
  ```

------

### 5. **检查 DataTable 是否有效**

- **方法**：`GetRowStruct`

- **作用**：获取 DataTable 的行结构体类型。

- **返回值**：如果 DataTable 有效，返回 `UScriptStruct*`；否则返回 `nullptr`。

- **示例**：

  cpp

  复制

  ```
  UDataTable* MyDataTable = ...; // 获取 DataTable
  if (MyDataTable && MyDataTable->GetRowStruct())
  {
      UE_LOG(LogTemp, Warning, TEXT("DataTable is valid!"));
  }
  else
  {
      UE_LOG(LogTemp, Warning, TEXT("DataTable is invalid!"));
  }
  ```

------

### 6. **在蓝图中使用 DataTable**

- **查找行**：使用 `Find Data Table Row` 节点。
- **获取所有行**：使用 `Get Data Table Row Names` 节点获取行名列表，然后遍历查找每行数据。
- **示例**：
  - 在蓝图中调用 `Find Data Table Row`，传入 DataTable 和行名，获取对应的行数据。

------

### 7. **总结**

| **方法**       | **作用**             | **返回值**                             |
| :------------- | :------------------- | :------------------------------------- |
| `FindRow`      | 根据行名查找数据     | 指向数据的指针（`nullptr` 如果未找到） |
| `GetRowMap`    | 获取所有行的映射     | `TMap<FName, uint8*>`                  |
| `GetRowNames`  | 获取所有行的名称列表 | `TArray<FName>`                        |
| `AddRow`       | 添加新行数据         | 无                                     |
| `RemoveRow`    | 删除行数据           | 无                                     |
| `GetRowStruct` | 获取行结构体类型     | `UScriptStruct*`                       |

通过以上方法，你可以高效地操作 `UDataTable`，查找、遍历和修改行数据。
