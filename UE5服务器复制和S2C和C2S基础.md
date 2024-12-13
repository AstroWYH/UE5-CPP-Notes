## 1  属性同步

- Replicated（属性同步）: 一个变量可以标记为 Replicated，这样它的值会从服务器自动同步到客户端。
- RepNotify（通知机制）: 在属性同步时，可以绑定一个回调函数（OnRep_XXX），以处理同步到客户端后需要额外执行的逻辑。
- `OnRep_XXX` 总是在 **客户端变量更新之后** 调用。

```
UPROPERTY(ReplicatedUsing = OnRep_Health)
float Health;

UFUNCTION()
void OnRep_Health();
```

## 2 S2C

```
/** Tell client to restart the level */
UFUNCTION(Reliable, Client)
void ClientRestart(class APawn* NewPawn);


void APlayerController::ClientRestart_Implementation(APawn* NewPawn)
{
    // xxx
}
```

**`Reliable`**: 表示这是一条 **可靠的 RPC** 调用。引擎会确保它一定到达客户端，并按顺序执行（即使网络状况不好，也会重试发送）。

**`Client`**: 表示这是一个 **客户端 RPC**，只能从**服务器端调用**。调用时，逻辑会被发送到客户端执行。

**`_Implementation`**：

- Unreal 的 RPC 机制需要你定义函数的逻辑内容时，在函数名后加 `_Implementation`。
- 这是引擎约定的函数命名方式，表示 `ClientRestart` 的实际实现逻辑。
- 当服务器调用 `ClientRestart` 时，客户端会执行 `ClientRestart_Implementation` 中的逻辑。



- `_Implementation` 函数的逻辑只能在对应的客户端上执行，不能在服务器上直接调用。
- 服务器调用，客户端提供实现

## 3 C2S

```
UFUNCTION(Reliable, Server, WithValidation)
void ServerSetPlayerName(const FString& NewName);
```

**`Server`**: 表明这是一个 C2S RPC，客户端调用，服务器端执行。

**`Reliable`**: 确保 RPC 是可靠的，会按顺序到达服务器。

**`WithValidation`**: 表示需要进行验证函数（`ServerSetPlayerName_Validate`）的实现，用于确保调用是合法的（防作弊）。

服务器实现

```
void AMyPlayerController::ServerSetPlayerName_Implementation(const FString& NewName)
{
    if (APlayerState* PS = GetPlayerState<APlayerState>())
    {
        PS->SetPlayerName(NewName);
        UE_LOG(LogTemp, Log, TEXT("Server: Player name updated to %s"), *NewName);
    }
}

bool AMyPlayerController::ServerSetPlayerName_Validate(const FString& NewName)
{
    // 验证玩家名字是否合法（例如长度限制或内容过滤）
    return !NewName.IsEmpty() && NewName.Len() <= 20;
}
```

客户端调用

```
if (HasAuthority())
{
    // 如果是服务器，直接更新名字
    ServerSetPlayerName("NewPlayerName");
}
else
{
    // 如果是客户端，发送请求到服务器
    ServerSetPlayerName("NewPlayerName");
}
```

