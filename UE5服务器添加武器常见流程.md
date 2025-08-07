```cpp
void UNinjaCombatWeaponManagerComponent::AddWeapon(AActor* Weapon)
{
	if (!IsValid(Weapon))
	{
		COMBAT_LOG(Warning, "Null weapon provided to the Weapon Manager!");
		return;
	}

	if (!Weapon->Implements<UCombatWeaponInterface>())
	{
		COMBAT_LOG_ARGS(Warning, "Weapon %s does not implement the Combat Weapon Interface!", *GetNameSafe(Weapon));
		return;
	}
	
	if (Weapons.Contains(Weapon))
	{
		COMBAT_LOG_ARGS(Warning, "Weapon '%s' is already registered!", *GetNameSafe(Weapon));
		return;	
	}
	
	if (OwnerHasAuthority())
	{
		const TArray<AActor*> CurrentWeapons = Weapons;
		Weapons.Add(Weapon);
		OnRep_Weapons(CurrentWeapons);
	}
	else if (OwnerIsLocallyControlled())
	{
		Server_AddWeapon(Weapon);
	}
}

/** Sends an RPC to add a weapon on the server. */
UFUNCTION(Server, Reliable, WithValidation)
void Server_AddWeapon(AActor* Weapon);

void UNinjaCombatWeaponManagerComponent::Server_AddWeapon_Implementation(AActor* Weapon)
{
	AddWeapon(Weapon);
}
```

关键点：

1. Server标记的 RPC 函数是客户端调用，服务器执行的特殊函数。
2. Server标记的 RPC，服务器能调用吗？
   能调用，但没有意义，也不推荐。Server RPC 的设计初衷是客户端向服务器发请求。如果服务器自己调用Server_AddWeapon()，本质上就是服务器调用自己的函数，不会产生任何网络传输，和直接调用普通函数没区别。但这违背了 RPC 的设计意图，可能导致逻辑混乱，所以实际开发中绝不会让服务器主动调用Server标记的 RPC。
3. 同理，Client RPC 是服务器调用，客户端执行的函数。

整个流程是**单向的**：
`本地客户端 →（RPC）→ 服务器执行AddWeapon → 同步给所有客户端`
没有循环，逻辑闭环。

### 完整流程举例（玩家 A 添加武器）

假设玩家 A 在自己的电脑（本地客户端）操作，游戏有一个服务器，还有玩家 B 的客户端：

1. 玩家 A 点击 "拾取武器"，触发`AddWeapon(新武器)`。
2. 玩家 A 的客户端是 "本地客户端"，所以调用`Server_AddWeapon(新武器)`（向服务器发请求）。
3. 服务器收到请求，执行`Server_AddWeapon_Implementation`，里面调用`AddWeapon(新武器)`。
4. 服务器执行`AddWeapon`时，`OwnerHasAuthority()`为真，所以直接把武器加入`Weapons`列表，然后通过`OnRep_Weapons`通知所有客户端（包括玩家 A 和玩家 B 的客户端）。
5. 所有客户端收到同步通知后，更新自己的`Weapons`列表，显示玩家 A 有了新武器。



#### 组件和数据在网络中的存在形式


### 1. `UNinjaCombatWeaponManagerComponent` 是"每个玩家各有一个"
- 每个玩家角色（比如玩家A、玩家B的角色）都会有自己的 `UNinjaCombatWeaponManagerComponent` 实例（组件依附于Actor，每个Actor独立拥有组件）。
- 所以：玩家A的角色有一个组件A，玩家B的角色有一个组件B，两者完全独立，分别管理A和B的武器。


### 2. `Weapons` 列表是"每个组件各有一份，且服务器和客户端各存一份"
UE的网络同步遵循"**服务器存真相，客户端存副本**"原则：
- **服务器上**：
  - 有组件A的服务器端实例，其 `Weapons` 列表记录玩家A真正拥有的武器（真相）。
  - 有组件B的服务器端实例，其 `Weapons` 列表记录玩家B真正拥有的武器（真相）。
- **玩家A的客户端上**：
  - 有组件A的客户端实例（本地角色的组件），`Weapons` 是服务器同步过来的"副本"（和服务器A的列表一致）。
  - 有组件B的客户端实例（远程角色的组件），`Weapons` 是服务器同步过来的"副本"（和服务器B的列表一致）。
- **玩家B的客户端上**：
  - 同理，有组件A和组件B的客户端实例，`Weapons` 都是服务器同步的副本。


### 3. 第4步的实际过程（针对玩家A拾取武器）
- 服务器执行 `AddWeapon` 时，是**给"服务器上的组件A"的 `Weapons` 列表添加武器**（这是唯一的"真相"）。
- 然后通过 `OnRep_Weapons`（属性复制），服务器会把"组件A的 `Weapons` 最新状态"同步给：
  - 玩家A的客户端：更新"客户端上的组件A"的 `Weapons` 副本（所以玩家A自己能看到武器）。
  - 玩家B的客户端：更新"客户端上的组件A"的 `Weapons` 副本（所以玩家B能看到玩家A有了新武器）。


### 一句话总结
- 组件和 `Weapons` 列表都是"**每个玩家独立拥有**"（A和B各管各的）。
- 同时，服务器和所有客户端都会为每个玩家保存一份 `Weapons` 数据（服务器存真相，客户端存同步过来的副本）。

这种设计既能保证每个玩家的数据独立，又能通过服务器同步让所有客户端看到一致的游戏状态。

- 服务器就像 "中央数据库"，保存着所有玩家的组件实例和Weapons数据（这是唯一的 "真相版本"）。
- 每个客户端会 "下载" 一份服务器数据的拷贝（包括自己和其他玩家的组件、Weapons列表），但客户端的拷贝只能看（或做本地表现，比如显示武器），不能直接修改 —— 所有修改必须通过服务器。

举个类比：
服务器是银行总部的数据库（记录你真正有多少钱），你的手机银行 APP（客户端）是一个拷贝，你能在 APP 上看余额、发起转账请求（就像客户端发 RPC），但真正的转账操作和余额更新只在银行总部（服务器）执行，之后再同步给你的 APP 显示新余额。

这样设计的核心目的是防止客户端作弊（比如自己改Weapons列表凭空加武器），同时保证所有玩家看到的游戏状态一致。