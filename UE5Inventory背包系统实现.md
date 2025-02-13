## MixInventorySubsystem.h

```
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "Subsystems/GameInstanceSubsystem.h"
#include "UObject/NoExportTypes.h"
#include "Templates/SharedPointer.h"
#include "MixData.h"
#include "MixInventorySubsystem.generated.h"

class UMixItem;
class UMixInventoryItem;

UCLASS(BlueprintType)
class UMixItemCfg : public UObject
{
	GENERATED_BODY()

public:
	void CopyFromFV(const FMixItemData* ItemData)
	{
		TID = ItemData->TID;
		Name = ItemData->Name;
		Icon = ItemData->Icon;
	}

	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	int32 TID;

	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	FString Name;

	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	UTexture2D* Icon;

};

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

## MixInventorySubsystem.cpp

```
// Fill out your copyright notice in the Description page of Project Settings.


#include "Subsystem/Inventory/MixInventorySubsystem.h"
#include "Engine/DataTable.h"
#include "DataTable/MixData.h"
#include "MixItem.h"
#include "MixInventoryItem.h"
#include "Data/MixDataSubsystem.h"

void UMixInventorySubsystem::Initialize(FSubsystemCollectionBase& Collection)
{
    Super::Initialize(Collection);

	UMixDataSubsystem* DataSys = GetGameInstance()->GetSubsystem<UMixDataSubsystem>();
    TArray<FMixItemData*> AllFItemsCfg = DataSys->LoadDataTableFromPath<FMixItemData>(TEXT("/Script/Engine.DataTable'/Game/MixGame/UI/Inventory/Data/DT_Item.DT_Item'"), TEXT("ItemDataContext"));

	for (FMixItemData* FItemData : AllFItemsCfg)
	{
		if (!ensure(FItemData)) return;

        TObjectPtr<UMixItemCfg> UItemData = NewObject<UMixItemCfg>();
        UItemData->CopyFromFV(FItemData);
		AllItemsCfg.Add(UItemData->TID, UItemData);
	}
}

void UMixInventorySubsystem::InventoryTestAddBtn()
{
    // Test
    int32 TestTID = -1;
    if (Cnt == 0)
    {
        TestTID = 100;
    }
    else if (Cnt == 1)
    {
        TestTID = 101;
    }

    TObjectPtr<UMixItem> Item = NewObject<UMixItem>();
    Item->TID = TestTID;
    if (!ensure(AllItemsCfg.Contains(TestTID))) return;
    Item->ItemCfg = AllItemsCfg[TestTID];

    // 模拟添加第1、2件物品
    AddItem(Item);
    Cnt++;
}

void UMixInventorySubsystem::InventoryTestRemoveBtn()
{
    Cnt--;

    // Test
    int32 TestTID = -1;
    if (Cnt == 0)
    {
        TestTID = 100;
    }
    else if (Cnt == 1)
    {
        TestTID = 101;
    }

    RemoveItem(TestTID);
}

void UMixInventorySubsystem::AddItem(TObjectPtr<UMixItem> Item, int32 PosIdx)
{
    // 不能超过格子总数
    if (!ensure(InventoryItems.Num() < KSlotNum)) return;

    if (!ensure(Item)) return;
    int32 TID = Item->TID;

    if (InventoryItems.Contains(TID))
    {
        if (!ensure(InventoryItems[TID])) return;
        InventoryItems[TID]->Amount++;
    }
    else
    {
        TObjectPtr<UMixInventoryItem> NewInventoryItem = NewObject<UMixInventoryItem>();
        int32 SavePosIdx = 0;

        // 自动安排升序第1个空位存放
        if (PosIdx == -1)
        {
            // 因为Remove位置的不确定性，每次重新计算SavePosIdx的位置
 
            for (int32 Idx = 0; Idx < KSlotNum; Idx++)
            {
                if (CurPosIdxes.Contains(Idx))
                {
                    SavePosIdx++;
                }
                else
                {
                    break;
                }
            }
        }
        // 指定位置存放
        else
        {
            SavePosIdx = PosIdx;
        }

		NewInventoryItem->Init(TID, 1, SavePosIdx);
		InventoryItems.Add(TID, NewInventoryItem);
        CurPosIdxes.Add(SavePosIdx);
        PosToTIDMap.Add(SavePosIdx, TID);
    }

    OnInventoryUpdated.Broadcast(true);
}

void UMixInventorySubsystem::RemoveItem(int32 TID)
{
    if (!ensure(InventoryItems.Contains(TID))) return;
    if (!ensure(InventoryItems[TID])) return;

    CurPosIdxes.Remove(InventoryItems[TID]->PosIdx);
    PosToTIDMap.Remove(InventoryItems[TID]->PosIdx);
    InventoryItems.Remove(TID);
    OnInventoryUpdated.Broadcast(false);
}

void UMixInventorySubsystem::ExchangeItem(int32 OldPosIdx, int32 NewPosIdx)
{
    if (!ensure(CurPosIdxes.Contains(OldPosIdx))) return;
    if (!ensure(CurPosIdxes.Contains(NewPosIdx))) return;
    if (!ensure(PosToTIDMap.Contains(OldPosIdx))) return;
    if (!ensure(PosToTIDMap.Contains(NewPosIdx))) return;

    int32 OldTID = PosToTIDMap[OldPosIdx];
    int32 NewTID = PosToTIDMap[NewPosIdx];

	TObjectPtr<UMixInventoryItem>& OldInventoryItem = InventoryItems.FindOrAdd(OldTID);
    if (!ensure(OldInventoryItem)) return;
    OldInventoryItem->PosIdx = NewPosIdx;

    TObjectPtr<UMixInventoryItem>& NewInventoryItem = InventoryItems.FindOrAdd(NewTID);
    if (!ensure(NewInventoryItem)) return;
    NewInventoryItem->PosIdx = OldPosIdx;

    PosToTIDMap[OldPosIdx] = NewTID;
    PosToTIDMap[NewPosIdx] = OldTID;

    OnInventoryUpdated.Broadcast(false);
}

void UMixInventorySubsystem::Deinitialize()
{
    Super::Deinitialize();

    InventoryItems.Empty();
    CurPosIdxes.Empty();
    PosToTIDMap.Empty();
    Cnt = 1;
}
```

## MixItem.h

```
#pragma once

#include "CoreMinimal.h"
#include "MixInventorySubsystem.h"
#include "MixItem.generated.h"

UCLASS(BlueprintType)
class MYGAME_API UMixItem : public UObject
{
	GENERATED_BODY()

public:
	UPROPERTY(BlueprintReadWrite, Category = "Inventory")
	int32 TID = -1;

	UPROPERTY(BlueprintReadWrite, Category = "Inventory")
	int32 XID = -1;

	UPROPERTY(BlueprintReadWrite, Category = "Inventory")
	TWeakObjectPtr<UMixItemCfg> ItemCfg = nullptr;

};
```

