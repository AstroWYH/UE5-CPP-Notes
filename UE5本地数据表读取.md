## UMixDataSubsystem

```cpp
UCLASS()
class MYGAME_API UMixDataSubsystem : public UGameInstanceSubsystem
{
	GENERATED_BODY()

public:
/*
	void Initialize(FSubsystemCollectionBase& Collection) override;

	void Deinitialize() override;*/

public:
	template <typename T>
	TArray<T*> LoadDataTableFromPath(const FString& Path, const FString& Context)
	{
		UDataTable* DataTable = LoadObject<UDataTable>(nullptr, *Path);
		if (!ensure(DataTable)) return TArray<T*>();

		TArray<T*> AllDataCfg;
		DataTable->GetAllRows<T>(Context, AllDataCfg);

		return AllDataCfg;
	}

};
```

## FMixItemData

```cpp
USTRUCT(BlueprintType)
struct FMixItemData : public FTableRowBase
{
	GENERATED_BODY()

public:
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	int32 TID;

	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	FString Name;

	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	UTexture2D* Icon;

};
```

