- 类似于业务逻辑的manager，不过是用UE自带的subsystem
- 最常见的是继承UGameInstanceSubsystem
- 不需要像从前的manager去创建单例，也不需要去借助gameinstance的生命周期（init, deinit）直接继承即可使用

- 关键函数：Initialize Deinitialize
- 如果需要tick，去继承FTickable相关的那个类即可

```c
UCLASS()
class MYGAME_API UMixInventorySubsystem : public UGameInstanceSubsystem
{
	GENERATED_BODY()

public:
	void Initialize(FSubsystemCollectionBase& Collection) override;

	void Deinitialize() override;

public:
	DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FMixOnInventoryUpdated, bool, bAdd);
	UPROPERTY(BlueprintAssignable)
    FMixOnInventoryUpdated OnInventoryUpdated;

public:
	UFUNCTION(BlueprintCallable, Category = "Inventory")
	void InventoryTestAddBtn();

	UFUNCTION(BlueprintCallable, Category = "Inventory")
	void InventoryTestRemoveBtn();

	void AddItem(TObjectPtr<UMixItem> Item, int32 PosIdx = -1);

	void RemoveItem(int32 TID);

	void ExchangeItem(int32 OldPosIdx, int32 NewPosIdx);

public:
	// 读表ItemData信息
	UPROPERTY(BlueprintReadWrite, EditAnywhere, Category = "Inventory")
	TMap<int32, TObjectPtr<UMixItemCfg>> AllItemsCfg;

	// 背包InventoryItem格子数据
	UPROPERTY(BlueprintReadWrite, EditAnywhere, Category = "Inventory")
	TMap<int32, TObjectPtr<UMixInventoryItem>> InventoryItems;

	// 通过Pos查询TID
	UPROPERTY(BlueprintReadWrite, EditAnywhere, Category = "Inventory")
	TMap<int32, int32> PosToTIDMap;

	// 当前占格位
	UPROPERTY(BlueprintReadWrite, EditAnywhere, Category = "Inventory")
	TSet<int32> CurPosIdxes;

	// 格子数量
	const int32 KSlotNum = 6;

	// Test
	int32 Cnt = 0;
};
```

