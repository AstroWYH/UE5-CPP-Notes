- 默认不带的，加上UCLASS(Blueprintable, BlueprintType)，可根据情况加上UAbstract
- 添加2个BP的函数，用Init和DeInit的生命周期带动，去蓝图实现
- 一般不推荐蓝图的Subsystem，因为官方不推荐，容易造成组件生命周期的问题

```c
UCLASS(Blueprintable, BlueprintType)
class MYGAME_API UMixUISubsystem : public UGameInstanceSubsystem
{
	GENERATED_BODY()

public:
    void Initialize(FSubsystemCollectionBase& Collection) override;

    void Deinitialize() override;

    UFUNCTION(BlueprintImplementableEvent, Category = "UI")
    void BpInitialize();

	UFUNCTION(BlueprintImplementableEvent, Category = "UI")
    void BpDeInitialize();

public:
/*
	UFUNCTION(BlueprintImplementableEvent, Category = "UI")
    void UpdateInventory();*/

};
```

