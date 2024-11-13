## UMixItemWidget

```
// Fill out your copyright notice in the Description page of Project Settings.

#include "UI/Item/MixItemWidget.h"
#include "Blueprint/DragDropOperation.h"
#include "Blueprint/WidgetBlueprintLibrary.h"
#include "Components/UniformGridPanel.h"
#include "Blueprint/WidgetTree.h"
#include "Inventory/MixInventorySubsystem.h"
#include "Inventory/MixItem.h"
#include "Components/Image.h"
#include "Framework/Application/SlateApplication.h"

void UMixItemWidget::NativeConstruct()
{
	Super::NativeConstruct();
}

void UMixItemWidget::NativeTick(const FGeometry& MyGeometry, float InDeltaTime)
{
/*
	FVector2D MousePosition = FSlateApplication::Get().GetCursorPos();
	FVector2D GridMousePosition = OwnerGrid->GetCachedGeometry().AbsoluteToLocal(MousePosition);
	FVector2D GirdSize = OwnerGrid->GetCachedGeometry().GetLocalSize();

	float CellWidth = GirdSize.X / KNumColumns;
	float CellHeight = GirdSize.Y / KNumRows;

	int32 Row = FMath::FloorToInt(GridMousePosition.Y / CellHeight);
	int32 Col = FMath::FloorToInt(GridMousePosition.X / CellWidth);
	int32 MousePosIdx = Row * KNumColumns + Col;

	if (GEngine)
	{
		GEngine->AddOnScreenDebugMessage(-1, 5.0f, FColor::Yellow,
			FString::Printf(TEXT("MousePosition: (X=%.2f, Y=%.2f), GridMousePosition: (X=%.2f, Y=%.2f), GridSize: (X=%.2f, Y=%.2f), CellWidth: %.2f, CellHeight: %.2f, Row: %d, Col: %d, MousePosIdx: %d"),
				MousePosition.X, MousePosition.Y,
				GridMousePosition.X, GridMousePosition.Y,
				GirdSize.X, GirdSize.Y,
				CellWidth, CellHeight,
				Row, Col, MousePosIdx));
	}*/
}

void UMixItemWidget::NativeOnDragDetected(const FGeometry& InGeometry, const FPointerEvent& InMouseEvent, UDragDropOperation*& OutOperation)
{
	Super::NativeOnDragDetected(InGeometry, InMouseEvent, OutOperation);

	UMixInventorySubsystem* InventorySys = GetGameInstance()->GetSubsystem<UMixInventorySubsystem>();
	if (!ensure(InventorySys->AllItemsCfg.Contains(ItemTID))) return;
	if (!ensure(InventorySys->AllItemsCfg[ItemTID])) return;

	UImage* DragImage = NewObject<UImage>(this);
	UTexture2D* DragTexture = InventorySys->AllItemsCfg[ItemTID]->Icon;
	DragImage->SetBrushFromTexture(DragTexture);

	UDragDropOperation* DragDropOpr = NewObject<UDragDropOperation>(GetTransientPackage());
	DragDropOpr->DefaultDragVisual = DragImage;
	DragDropOpr->Payload = this;
	DragDropOpr->Pivot = EDragPivot::CenterCenter;
	OutOperation = DragDropOpr;

	SetVisibility(ESlateVisibility::Hidden);
}

void UMixItemWidget::NativeOnDragEnter(const FGeometry& InGeometry, const FDragDropEvent& InDragDropEvent, UDragDropOperation* InOperation)
{
	Super::NativeOnDragEnter(InGeometry, InDragDropEvent, InOperation);
}

bool UMixItemWidget::NativeOnDragOver(const FGeometry& InGeometry, const FDragDropEvent& InDragDropEvent, UDragDropOperation* InOperation)
{
	return Super::NativeOnDragOver(InGeometry, InDragDropEvent, InOperation);
}

void UMixItemWidget::NativeOnDragLeave(const FDragDropEvent& InDragDropEvent, UDragDropOperation* InOperation)
{
	Super::NativeOnDragLeave(InDragDropEvent, InOperation);
}

bool UMixItemWidget::NativeOnDrop(const FGeometry& InGeometry, const FDragDropEvent& InDragDropEvent, UDragDropOperation* InOperation)
{
	return Super::NativeOnDrop(InGeometry, InDragDropEvent, InOperation);
}

void UMixItemWidget::NativeOnDragCancelled(const FDragDropEvent& InDragDropEvent, UDragDropOperation* InOperation)
{
	Super::NativeOnDragCancelled(InDragDropEvent, InOperation);
	if (!ensure(OwnerGrid.IsValid())) return;

	FVector2D MousePosition = InDragDropEvent.GetScreenSpacePosition();
	FVector2D GridMousePosition = OwnerGrid->GetCachedGeometry().AbsoluteToLocal(MousePosition);
	FVector2D GirdSize = OwnerGrid->GetCachedGeometry().GetLocalSize();

	float CellWidth = GirdSize.X / KNumColumns;
	float CellHeight = GirdSize.Y / KNumRows;

	int32 Row = FMath::FloorToInt(GridMousePosition.Y / CellHeight);
	int32 Col = FMath::FloorToInt(GridMousePosition.X / CellWidth);
	int32 MousePosIdx = Row * KNumColumns + Col;

	bool bCrossBorder = false;
	if (Row < 0 || Row >= KNumRows || Col < 0 || Col >= KNumColumns)
	{
		bCrossBorder = true;
	}

	UMixInventorySubsystem* InventorySys = GetGameInstance()->GetSubsystem<UMixInventorySubsystem>();
	TSet<int32> AllPosIdx{ 0,1,2,3,4,5 };
	if (!ensure(OwnerWidget.IsValid())) return;

	// 拖到原格子松手 || 拖到外部空间松手 
	if (MousePosIdx == PosIdx || !AllPosIdx.Contains(MousePosIdx) || bCrossBorder)
	{
		SetVisibility(ESlateVisibility::Visible);
	}
	// 拖到其他空格子松手
	else if (!InventorySys->CurPosIdxes.Contains(MousePosIdx))
	{
		OwnerWidget->DragToOtherEmptySlot(ItemTID, PosIdx, MousePosIdx);
	}
	// 拖到其他非空格子松手
	else if (InventorySys->CurPosIdxes.Contains(MousePosIdx))
	{
		OwnerWidget->DragToExchange(PosIdx, MousePosIdx);
	}
}

FReply UMixItemWidget::NativeOnMouseButtonDown(const FGeometry& InGeometry, const FPointerEvent& InMouseEvent)
{
	Super::NativeOnMouseButtonDown(InGeometry, InMouseEvent);

	return UWidgetBlueprintLibrary::DetectDragIfPressed(InMouseEvent, this, EKeys::LeftMouseButton).NativeReply;
}

```

## UMixInventoryWidget

```
// Fill out your copyright notice in the Description page of Project Settings.


#include "UI/Item/MixInventoryWidget.h"
#include "Inventory/MixInventorySubsystem.h"
#include "Subsystem/Inventory/MixItem.h"

void UMixInventoryWidget::DragToOtherEmptySlot(int32 TID, int32 OldPosIdx, int32 NewPosIdx)
{
	UMixInventorySubsystem* InventorySys = GetGameInstance()->GetSubsystem<UMixInventorySubsystem>();
	InventorySys->RemoveItem(TID);

    TObjectPtr<UMixItem> Item = NewObject<UMixItem>();
    Item->TID = TID;
    if (!ensure(InventorySys->AllItemsCfg.Contains(TID))) return;
    Item->ItemCfg = InventorySys->AllItemsCfg[TID];
	InventorySys->AddItem(Item, NewPosIdx);
}

void UMixInventoryWidget::DragToExchange(int32 OldPosIdx, int32 NewPosIdx)
{
    UMixInventorySubsystem* InventorySys = GetGameInstance()->GetSubsystem<UMixInventorySubsystem>();
    InventorySys->ExchangeItem(OldPosIdx, NewPosIdx);
}

```

