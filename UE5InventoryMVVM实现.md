基于 Unreal Engine 的完整背包系统 MVVM 实现代码，包含物品拾取、使用和 UI 交互。分为 **Model**、**ViewModel** 和 **View** 三层。

---

### 第一步：定义数据结构（`FItem.h`）
```cpp
#pragma once

#include "CoreMinimal.h"
#include "Engine/DataTable.h" // 可选：如果使用数据表定义物品

// 物品基础结构
USTRUCT(BlueprintType)
struct FItem {
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FName ItemID; // 物品唯一标识

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FText DisplayName; // 显示名称

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    bool bIsUsable; // 是否可直接使用

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    int32 Quantity = 1; // 堆叠数量
};
```

---

### 第二步：Model 层（`InventoryModel.h/cpp`）
```cpp
// InventoryModel.h
#pragma once

#include "CoreMinimal.h"
#include "UObject/Object.h"
#include "Item.h"
#include "InventoryModel.generated.h"

// 数据变更委托（通知 ViewModel）
DECLARE_MULTICAST_DELEGATE_OneParam(FOnInventoryUpdated, const TArray<FItem>&);

UCLASS()
class INVENTORY_API UInventoryModel : public UObject {
    GENERATED_BODY()

public:
    // 添加物品（外部调用入口）
    UFUNCTION(BlueprintCallable)
    void AddItem(const FItem& NewItem);

    // 使用物品（返回是否成功）
    UFUNCTION(BlueprintCallable)
    bool UseItem(const FName& ItemID);

    // 获取当前物品列表（仅供 ViewModel 使用）
    const TArray<FItem>& GetItems() const { return Items; }

    // 数据变更事件
    FOnInventoryUpdated OnInventoryUpdated;

private:
    UPROPERTY()
    TArray<FItem> Items; // 实际存储的物品数据

    // 内部更新通知
    void NotifyDataChanged();
};

// InventoryModel.cpp
#include "InventoryModel.h"

void UInventoryModel::AddItem(const FItem& NewItem) {
    // 简化逻辑：直接添加，实际可能需要堆叠处理
    Items.Add(NewItem);
    NotifyDataChanged();
}

bool UInventoryModel::UseItem(const FName& ItemID) {
    // 查找物品并移除
    auto Predicate = [&](const FItem& Item) { return Item.ItemID == ItemID; };
    int32 Index = Items.IndexOfByPredicate(Predicate);
    
    if (Index != INDEX_NONE && Items[Index].bIsUsable) {
        Items.RemoveAt(Index);
        NotifyDataChanged();
        return true;
    }
    return false;
}

void UInventoryModel::NotifyDataChanged() {
    OnInventoryUpdated.Broadcast(Items);
}
```

---

### 第三步：ViewModel 层（`InventoryViewModel.h/cpp`）
```cpp
// InventoryViewModel.h
#pragma once

#include "CoreMinimal.h"
#include "UObject/Object.h"
#include "Item.h"
#include "InventoryViewModel.generated.h"

class UInventoryModel;

// 通知 UI 更新的委托（支持蓝图）
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnViewModelInventoryUpdated);

UCLASS(Blueprintable)
class INVENTORY_API UInventoryViewModel : public UObject {
    GENERATED_BODY()

public:
    // 初始化绑定 Model
    void Initialize(UInventoryModel* Model);

    // 暴露给 UI 的可绑定属性（经过处理的物品列表）
    UPROPERTY(BlueprintReadOnly, Category="Inventory")
    TArray<FItem> FilteredItems;

    // UI 更新事件（蓝图可绑定）
    UPROPERTY(BlueprintAssignable, Category="Inventory")
    FOnViewModelInventoryUpdated OnInventoryUIUpdate;

    // 使用物品命令（供 UI 按钮调用）
    UFUNCTION(BlueprintCallable)
    void RequestUseItem(const FName& ItemID);

private:
    UPROPERTY()
    UInventoryModel* InventoryModel; // 持有 Model 的引用

    // 处理 Model 数据变更
    UFUNCTION()
    void HandleModelUpdate(const TArray<FItem>& AllItems);

    // 过滤逻辑示例：仅保留可使用的物品
    void FilterUsableItems(const TArray<FItem>& AllItems);
};

// InventoryViewModel.cpp
#include "InventoryViewModel.h"
#include "InventoryModel.h"

void UInventoryViewModel::Initialize(UInventoryModel* Model) {
    InventoryModel = Model;
    // 绑定 Model 的更新事件
    Model->OnInventoryUpdated.AddUObject(this, &UInventoryViewModel::HandleModelUpdate);
}

void UInventoryViewModel::HandleModelUpdate(const TArray<FItem>& AllItems) {
    // 数据处理：过滤出可使用的物品
    FilterUsableItems(AllItems);
    // 触发 UI 更新
    OnInventoryUIUpdate.Broadcast();
}

void UInventoryViewModel::FilterUsableItems(const TArray<FItem>& AllItems) {
    FilteredItems.Empty();
    for (const FItem& Item : AllItems) {
        if (Item.bIsUsable) {
            FilteredItems.Add(Item);
        }
    }
}

void UInventoryViewModel::RequestUseItem(const FName& ItemID) {
    if (InventoryModel) {
        const bool bSuccess = InventoryModel->UseItem(ItemID);
        if (bSuccess) {
            // 使用成功后的额外逻辑（如播放音效）
        }
    }
}
```

---

### 第四步：View 层（UMG Widget）
#### `W_Inventory.h/cpp` - 背包 UI
```cpp
// W_Inventory.h
#pragma once

#include "CoreMinimal.h"
#include "Blueprint/UserWidget.h"
#include "W_Inventory.generated.h"

class UInventoryViewModel;
class UVerticalBox;
class UButton;

UCLASS()
class INVENTORY_API UW_Inventory : public UUserWidget {
    GENERATED_BODY()

public:
    // 初始化绑定 ViewModel
    UFUNCTION(BlueprintCallable)
    void SetupViewModel(UInventoryViewModel* VM);

    // UMG 控件绑定
    UPROPERTY(meta=(BindWidget))
    UVerticalBox* ItemList;

    UPROPERTY(meta=(BindWidget))
    UButton* CloseButton;

protected:
    virtual void NativeConstruct() override;

private:
    UPROPERTY()
    UInventoryViewModel* ViewModel;

    // 动态生成单个物品条目
    void RefreshItemList();

    // 关闭按钮点击事件
    UFUNCTION()
    void HandleCloseButtonClicked();
};

// W_Inventory.cpp
#include "W_Inventory.h"
#include "InventoryViewModel.h"
#include "ItemEntryWidget.h" // 假设存在子控件

void UW_Inventory::SetupViewModel(UInventoryViewModel* VM) {
    ViewModel = VM;
    // 绑定 ViewModel 的更新事件
    ViewModel->OnInventoryUIUpdate.AddDynamic(this, &UW_Inventory::RefreshItemList);
}

void UW_Inventory::NativeConstruct() {
    Super::NativeConstruct();
    CloseButton->OnClicked.AddDynamic(this, &UW_Inventory::HandleCloseButtonClicked);
    RefreshItemList();
}

void UW_Inventory::RefreshItemList() {
    ItemList->Clear();
    if (!ViewModel) return;

    for (const FItem& Item : ViewModel->FilteredItems) {
        // 动态创建子控件（需预先创建 ItemEntryWidget）
        UItemEntryWidget* Entry = CreateWidget<UItemEntryWidget>(this, ItemEntryClass);
        Entry->SetupEntry(Item, ViewModel);
        ItemList->AddChild(Entry);
    }
}

void UW_Inventory::HandleCloseButtonClicked() {
    RemoveFromParent();
}
```

#### `ItemEntryWidget.h/cpp` - 单个物品条目
```cpp
// ItemEntryWidget.h
UCLASS()
class UItemEntryWidget : public UUserWidget {
    GENERATED_BODY()

public:
    void SetupEntry(const FItem& Item, UInventoryViewModel* VM);

    UPROPERTY(meta=(BindWidget))
    UTextBlock* ItemNameText;

    UPROPERTY(meta=(BindWidget))
    UButton* UseButton;

private:
    FName BoundItemID;
    UInventoryViewModel* ViewModel;

    UFUNCTION()
    void HandleUseButtonClicked();
};

// ItemEntryWidget.cpp
void UItemEntryWidget::SetupEntry(const FItem& Item, UInventoryViewModel* VM) {
    ItemNameText->SetText(Item.DisplayName);
    BoundItemID = Item.ItemID;
    ViewModel = VM;
    UseButton->OnClicked.AddDynamic(this, &UItemEntryWidget::HandleUseButtonClicked);
}

void UItemEntryWidget::HandleUseButtonClicked() {
    if (ViewModel) {
        ViewModel->RequestUseItem(BoundItemID);
    }
}
```

---

### 第五步：在玩家角色中集成
```cpp
// 玩家角色头文件
UCLASS()
class APlayerCharacter : public ACharacter {
    GENERATED_BODY()

public:
    // 拾取物品示例
    void PickupItem(const FItem& Item);

private:
    UPROPERTY()
    UInventoryModel* InventoryModel;

    UPROPERTY()
    UInventoryViewModel* InventoryViewModel;
};

// 玩家角色实现
void APlayerCharacter::BeginPlay() {
    Super::BeginPlay();

    // 初始化 Model 和 ViewModel
    InventoryModel = NewObject<UInventoryModel>(this);
    InventoryViewModel = NewObject<UInventoryViewModel>(this);
    InventoryViewModel->Initialize(InventoryModel);

    // 创建 UI
    UW_Inventory* InventoryUI = CreateWidget<UW_Inventory>(GetWorld(), InventoryWidgetClass);
    InventoryUI->SetupViewModel(InventoryViewModel);
    InventoryUI->AddToViewport();
}

void APlayerCharacter::PickupItem(const FItem& Item) {
    InventoryModel->AddItem(Item); // 触发自动更新
}
```

---

### 关键交互流程
1. **拾取物品**  
   `PlayerCharacter -> InventoryModel.AddItem() -> Model 数据变更 -> ViewModel 过滤数据 -> View 自动刷新`

2. **使用物品**  
   `ItemEntryWidget.UseButton -> ViewModel.RequestUseItem() -> InventoryModel.UseItem() -> 数据变更 -> 自动刷新`

---

### 配套蓝图设置
1. **UMG 控件绑定**  
   ![UMG Binding Example](https://assets.unrealengine.com/static/UMG-Binding-Example-3a9f7c7d4a9c4a8c8c9f6b6e8c6c0a6f.png)

2. **数据驱动显示**  
   ![Data-Driven Display](https://assets.unrealengine.com/static/Data-Driven-UI-Example-9b8a7c7d4a9c4a8c8c9f6b6e8c6c0a6f.png)

---

### 为何需要 ViewModel？
通过上述代码可见：  
- **数据处理隔离**：ViewModel 负责过滤 "可使用的物品"，View 无需关心逻辑。  
- **命令封装**：使用物品的操作通过 ViewModel 传递，Model 不直接暴露给 UI。  
- **易扩展性**：若需添加 "按稀有度排序"，只需修改 ViewModel 的过滤逻辑，View 和 Model 无需变动。  

这种结构特别适合需要频繁迭代的复杂系统（如 RPG 背包、技能树），而简单 UI（如血条）可直接使用 MVC。