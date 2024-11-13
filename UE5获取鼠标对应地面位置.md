```c
FVector AMixHostController::GetMouseClickFloorPosition()
{
	// 这里只能用GetMousePositionOnViewport()，结果才正确
	FVector2D MousePosition = UWidgetLayoutLibrary::GetMousePositionOnViewport(GetWorld());
	float Scale = UWidgetLayoutLibrary::GetViewportScale(GetWorld());
	FVector2D ScreenMousePosition = MousePosition * Scale;

	FVector CameraWorldPosition;
	FVector CamerToMouseWorldDirection;
	UGameplayStatics::DeprojectScreenToWorld(this, ScreenMousePosition, CameraWorldPosition, CamerToMouseWorldDirection);

	TArray<AActor*> IgnoreActors;
	FHitResult HitResult;
	// DefaultEngine.ini配置了编辑器里新增的Cfg，ECC_GameTraceChannel2对应WalkFloor的ObjType
	TArray<TEnumAsByte<EObjectTypeQuery>> ObjectTypes{ UEngineTypes::ConvertToObjectType(ECollisionChannel::ECC_GameTraceChannel2) };
	UKismetSystemLibrary::LineTraceSingleForObjects(GetWorld(), CameraWorldPosition, CamerToMouseWorldDirection * 10000 + CameraWorldPosition, ObjectTypes, false, IgnoreActors, EDrawDebugTrace::None, HitResult, true, FLinearColor::Green);

	return FVector(HitResult.Location.X, HitResult.Location.Y, 100);
}
```

