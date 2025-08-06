
## 核心类职责说明

### 1. `Character` 类
- **角色基类**：代表游戏中持有技能系统的实体（如玩家、NPC）
- **核心职责**：
  - 持有技能系统核心组件 `AbilitySystemComponent`
  - 存储实体名称等基础信息
- **关键属性**：
  - `ASC`：技能系统组件，负责技能的管理与执行
  - `Name`：实体名称

### 2. `Target` 类
- **目标实体类**：代表技能作用的目标（如被攻击的敌人）
- **核心职责**：
  - 存储目标基础信息（如名称）
  - 作为技能效果的作用对象

### 3. `AbilitySystemComponent`（ASC）
- **技能系统核心组件**：类比 UE 中的 UAbilitySystemComponent
- **核心职责**：
  - 管理技能的授权（Grant）与存储
  - 处理技能激活请求（TryActivateAbility）
  - 应用技能效果（ApplyGameplayEffectToSelf）
- **关键方法**：
  - `GrantAbility`：向实体授予技能
  - `TryActivateAbility`：尝试激活指定技能
  - `ApplyGameplayEffectToSelf`：应用技能效果

### 4. `GameplayAbility` 类
- **技能基类**：定义技能的通用行为
- **核心职责**：
  - 管理技能冷却（Cooldown）
  - 提供技能激活检查（CanActivate）
  - 定义技能执行入口（Activate）
- **关键属性**：
  - `Name`：技能名称
  - `CooldownSeconds`：冷却时间（秒）
- **关键方法**：
  - `CanActivate`：检查技能是否可激活（冷却是否结束）
  - `Activate`：激活技能（执行冷却检查并调用具体执行逻辑）
  - `ExecuteAbility`：抽象方法，由子类实现具体技能逻辑

### 5. `GameAbility_Fireball` 类
- **具体技能实现**：火球术技能，继承自 `GameplayAbility`
- **核心职责**：
  - 实现火球术的具体执行逻辑
  - 定义火球术的名称和冷却时间（3秒）
- **实现细节**：激活时创建伤害效果并通过 ASC 应用

### 6. `GameplayEffect` 类
- **技能效果基类**：定义技能产生的效果（如伤害、治疗、Buff等）
- **核心职责**：
  - 提供效果应用的抽象接口（Apply）
- **关键方法**：
  - `Apply`：抽象方法，由子类实现具体效果逻辑

### 7. `GameplayEffect_Damage` 类
- **具体效果实现**：伤害效果，继承自 `GameplayEffect`
- **核心职责**：
  - 实现伤害效果的应用逻辑
  - 存储伤害数值信息

### 8. `AbilityContext` 类
- **技能上下文类**：传递技能执行过程中的关键信息
- **核心职责**：
  - 存储技能来源（Source）、目标（Target）
  - 传递技能参数（如基础伤害值）

### 9. `EffectHandler` 类
- **效果处理器**：处理效果的实际表现逻辑
- **核心职责**：
  - 执行效果的具体表现（如打印伤害日志）
  - 通过事件（OnDamage）通知其他系统（如UI、音效系统）
- **关键机制**：使用委托事件实现低耦合的通知机制


## 执行流程示例（以火球术为例）
1. **初始化**：
   - 创建 `EffectHandler` 实例处理效果表现
   - 创建 `Actor`（英雄）和 `Target`（怪物）实例
   - 向英雄的 ASC 授予火球术技能

2. **技能激活**：
   - 调用 `ASC.TryActivateAbility("Fireball", context)` 发起激活请求
   - ASC 检查技能是否存在，调用技能的 `Activate` 方法
   - 技能检查冷却时间，若可激活则记录使用时间并执行 `ExecuteAbility`

3. **效果应用**：
   - 火球术创建 `GameplayEffect_Damage` 实例
   - 通过 ASC 调用 `ApplyGameplayEffectToSelf` 应用效果
   - 伤害效果调用 `EffectHandler` 执行实际伤害逻辑
   - `EffectHandler` 触发 `OnDamage` 事件，输出伤害日志
   
   
   
   # 代码实现

```
// Copyright (c) Night Gamer. All rights reserved.
// ==========================
// AbilityContext.cs
// 类似 UE 中的 FGameplayAbilityActorInfo 或 FGameplayEffectContextHandle
// 用于在技能执行过程中传递施法者、目标、伤害数值等上下文信息
// ==========================

public class AbilityContext
{
    public Character Source { get; private set; }
    public Target Target { get; private set; }
    public float Damage { get; set; }

    public AbilityContext(Character source, Target target, float damage = 0)
    {
        Source = source;
        Target = target;
        Damage = damage;
    }
}
// Copyright (c) Night Gamer. All rights reserved.
// ==========================
// AbilitySystemComponent.cs
// 类似UE中的UAbilitySystemComponent（ASC）
// 是技能系统的核心组件，负责技能授权、激活与效果应用
// 每个角色Actor持有一个ASC，统一管理技能系统相关逻辑
// ==========================

using System;
using System.Collections.Generic;

public class AbilitySystemComponent
{
    private Dictionary<string, GameplayAbility> Abilities = new();
    private EffectHandler Handler;
    public Character Owner { get; private set; } // 持有所有者
    public AttributeSet AttributeSet => Owner.AttributeSet;

    public AbilitySystemComponent(EffectHandler handler, Character owner)
    {
        Handler = handler;
        Owner = owner;

        // 订阅属性变化，打印关键信息（如死亡）
        AttributeSet.OnAttributeChanged += (type, old, val, _) =>
        {
            if (type == AttributeSet.AttributeType.Health && val <= 0)
            {
                Console.WriteLine($"[ASC] {Owner.Name} 生命值归0，已死亡！");
            }
        };
    }

    public void GrantAbility(GameplayAbility ability)
    {
        if (Abilities.ContainsKey(ability.Name))
        {
            Console.WriteLine($"[ASC] {Owner.Name} 已拥有技能 {ability.Name}，无需重复授予");
            return;
        }
        Abilities[ability.Name] = ability;
        Console.WriteLine($"[ASC] 授予技能 {ability.Name} 给 {Owner.Name}");
    }

    public void TryActivateAbility(string abilityName, AbilityContext context)
    {
        if (!Abilities.TryGetValue(abilityName, out var ability))
        {
            Console.WriteLine($"[ASC] {Owner.Name} 未拥有技能 {abilityName}，无法激活");
            return;
        }

        // 检查技能是否可激活（封装在ASC中，体现GAS的集中控制）
        if (!ability.CanActivate(Owner.AttributeSet))
        {
            Console.WriteLine($"[ASC] {Owner.Name} 的技能 {abilityName} 无法激活（冷却中或资源不足）");
            return;
        }

        Console.WriteLine($"[ASC] {Owner.Name} 开始激活技能 {abilityName}");
        ability.Activate(context, this);
    }

    public void ApplyGameplayEffectToTarget(GameplayEffect effect, AbilityContext context)
    {
        Console.WriteLine($"[ASC] {Owner.Name} 对 {context.Target.Name} 应用效果 {effect.Name}");
        effect.Apply(context, Handler);
    }
}
// Copyright (c) Night Gamer. All rights reserved.
// ==========================
// Hero.cs
// 代表技能释放者（角色），持有ASC和AttributeSet
// 类似UE中的ACharacter或APlayerState
// ==========================

public class Actor
{
    public string Name { get; private set; }
    public AttributeSet AttributeSet { get; private set; } // 新增属性集

    public Actor(string name)
    {
        Name = name;
        AttributeSet = new AttributeSet(this);
    }
}
// Copyright (c) Night Gamer. All rights reserved.
// ==========================
// AttributeSet.cs
// 类似UE中的UAttributeSet
// 用于管理角色的各种属性（生命值、法力值等）
// 提供属性修改和获取的基础功能
// ==========================

using System;
using System.Collections.Generic;
using System.Net;
using System.Numerics;

public class AttributeSet
{
    // 定义属性类型枚举，体现GAS的属性分类
    public enum AttributeType
    {
        Health,
        MaxHealth,
        Mana,
        MaxMana,
        AttackPower,
        Defense
    }

    private Dictionary<AttributeType, float> attributes = new();
    public event Action<AttributeType, float, float, object> OnAttributeChanged;
    public object Owner { get; private set; } // 持有所有者

    public AttributeSet(object owner)
    {
        Owner = owner;
    }

    public void InitAttributeSet(float maxHealth, float health, float maxMana, float Mana, float attackPower, float defense)
    {
        // 初始化属性并记录日志
        SetAttribute(AttributeType.MaxHealth, maxHealth, Owner);
        SetAttribute(AttributeType.Health, health, Owner);
        SetAttribute(AttributeType.MaxMana, maxMana, Owner);
        SetAttribute(AttributeType.Mana, Mana, Owner);
        SetAttribute(AttributeType.AttackPower, attackPower, Owner);
        SetAttribute(AttributeType.Defense, defense, Owner);
    }

    // 带发起者和日志的属性设置
    public void SetAttribute(AttributeType type, float value, object instigator)
    {
        if (attributes.TryGetValue(type, out float oldValue))
        {
            if (Math.Abs(oldValue - value) < 0.001f) return; // 避免浮点数精度问题

            attributes[type] = value;
            OnAttributeChanged?.Invoke(type, oldValue, value, instigator);
            if (instigator is Actor ins && Owner is Actor owner)
                Console.WriteLine($"[AS] {owner.Name} 修改属性 {type}: {oldValue:F1} → {value:F1}，发起者: {ins.Name}");
        }
        else
        {
            attributes.Add(type, value);
            OnAttributeChanged?.Invoke(type, 0, value, instigator);
            if (instigator is Actor ins && Owner is Actor owner)
                Console.WriteLine($"[AS] {owner.Name} 初始化属性 {type}: {oldValue:F1} → {value:F1}，发起者: {ins.Name}");
        }
    }

    public float GetAttribute(AttributeType type)
    {
        attributes.TryGetValue(type, out float value);
        return value;
    }

    // 带约束的属性修改（如生命值不能超过最大值）
    public void ModifyAttribute(AttributeType type, float delta, object instigator)
    {
        float current = GetAttribute(type);
        float newValue = current + delta;

        // 特殊属性约束（体现GAS的属性规则）
        if (type == AttributeType.Health)
        {
            newValue = Math.Clamp(newValue, 0, GetAttribute(AttributeType.MaxHealth));
        }
        else if (type == AttributeType.Mana)
        {
            newValue = Math.Clamp(newValue, 0, GetAttribute(AttributeType.MaxMana));
        }

        SetAttribute(type, newValue, instigator);
    }
}
// Copyright (c) Night Gamer. All rights reserved.
// ==========================
// Hero.cs
// 代表技能释放者（角色），持有ASC和AttributeSet
// 类似UE中的ACharacter或APlayerState
// ==========================

public class Character : Actor
{
    public AbilitySystemComponent ASC { get; private set; }

    public Character(string name, EffectHandler handler) : base(name)
    {
        ASC = new AbilitySystemComponent(handler, this); // 传入自身到ASC
    }
}
// Copyright (c) Night Gamer. All rights reserved.
// ==========================
// EffectHandler.cs
// 类似 UE 中的 GameplayCueManager / CueNotifyActor
// 用于处理实际的表现逻辑（如播放特效、广播伤害）
// 也包含一个简单的委托机制，模拟事件通知
// ==========================

using System;

public class EffectHandler
{
    public delegate void DamageEvent(AbilityContext context);
    public event DamageEvent OnDamage;

    public void ApplyEffect(AbilityContext context)
    {
        // 打印效果表现日志（类似UE的GameplayCue）
        Console.WriteLine($"[Effect] 特效：{context.Source.Name} 对 {context.Target.Name} 造成 {context.Damage} 点伤害（火花四溅！）");

        OnDamage?.Invoke(context);
    }
}
// Copyright (c) Night Gamer. All rights reserved.
// ==========================
// GameplayAbility.cs
// 类比于 UE 中的 UGameplayAbility，是技能的基类
// 负责控制技能冷却判断、激活流程、调用执行逻辑
// 可继承实现具体技能逻辑，例如火球术等
// ==========================

using System;

public abstract class GameplayAbility
{
    public string Name { get; private set; }
    public int CooldownSeconds { get; private set; }
    public int ManaCost { get; private set; } // 技能消耗的法力值
    private DateTime lastUsed = DateTime.MinValue;

    // 构造函数新增法力值消耗参数
    public GameplayAbility(string name, int cooldown, int manaCost)
    {
        Name = name;
        CooldownSeconds = cooldown;
        ManaCost = manaCost;
    }

    // 检查是否可激活（结合冷却和资源）
    public bool CanActivate(AttributeSet attributeSet)
    {
        var remainingCooldown = CooldownSeconds - (DateTime.Now - lastUsed).TotalSeconds;
        if (remainingCooldown > 0)
        {
            Console.WriteLine($"[GA] {Name} 冷却中，剩余 {remainingCooldown:F1} 秒");
            return false;
        }

        if (attributeSet.GetAttribute(AttributeSet.AttributeType.Mana) < ManaCost)
        {
            Console.WriteLine($"[GA] {Name} 法力值不足（当前: {attributeSet.GetAttribute(AttributeSet.AttributeType.Mana)}, 所需: {ManaCost}）");
            return false;
        }

        return true;
    }

    public void Activate(AbilityContext context, AbilitySystemComponent asc)
    {
        // 消耗法力值（体现GAS的资源系统）
        asc.AttributeSet.ModifyAttribute(AttributeSet.AttributeType.Mana, -ManaCost, asc.Owner);
        lastUsed = DateTime.Now;

        Console.WriteLine($"[GA] {asc.Owner.Name} 成功激活 {Name}，冷却 {CooldownSeconds} 秒");
        ExecuteAbility(context, asc);
    }

    protected abstract void ExecuteAbility(AbilityContext context, AbilitySystemComponent asc);
}
// Copyright (c) Night Gamer. All rights reserved.
// ==========================
// GameplayAbility_Fireball.cs
// 继承自 GameplayAbilityBase，代表一个具体技能：火球术
// 类似 UE 中具体的技能 Blueprint 类（继承自UGameplayAbility）
// ==========================

public class GameplayAbility_Fireball : GameplayAbility
{
    public GameplayAbility_Fireball(string name, int cooldown, int manaCost) : base(name, cooldown, manaCost) { }

    protected override void ExecuteAbility(AbilityContext context, AbilitySystemComponent asc)
    {
        // 传递基础伤害到效果
        var damageEffect = new GameplayEffect_Damage("GE伤害效果", 20);
        asc.ApplyGameplayEffectToTarget(damageEffect, context); // 改为对目标应用效果
    }
}
// Copyright (c) Night Gamer. All rights reserved.
// ==========================
// GameplayEffect.cs
// 类似 UE 中的 UGameplayEffect，用于描述技能造成的效果
// 例如伤害、治疗、Buff 等，这里以伤害为例
// ==========================

public abstract class GameplayEffect
{
    public string Name { get; private set; }

    protected GameplayEffect(string name)
    {
        Name = name;
        Console.WriteLine($"[GE] 创建技能效果 {name}");
    }

    public abstract void Apply(AbilityContext context, EffectHandler handler);
}
// Copyright (c) Night Gamer. All rights reserved.
// ==========================
// GameplayEffect_Damage.cs
// 一个具体的GameplayEffect实现，造成伤害
// 现在使用AttributeSet来修改目标生命值
// ==========================

public class GameplayEffect_Damage : GameplayEffect
{
    private int BaseDamage;

    public GameplayEffect_Damage(string name, int baseDamage) : base(name)
    {
        BaseDamage = baseDamage;
    }

    public override void Apply(AbilityContext context, EffectHandler handler)
    {
        // 从属性集获取攻击者攻击力和目标防御力（体现GAS的属性联动）
        float attackPower = context.Source.AttributeSet.GetAttribute(AttributeSet.AttributeType.AttackPower);
        float targetDefense = context.Target.AttributeSet.GetAttribute(AttributeSet.AttributeType.Defense);

        // 伤害公式：基础伤害 + 攻击力 - 目标防御（最低1点伤害）
        float finalDamage = Math.Max(1, BaseDamage + attackPower - targetDefense);
        context.Damage = finalDamage;
        // 应用伤害到目标生命值
        context.Target.AttributeSet.ModifyAttribute(
            AttributeSet.AttributeType.Health,
            -finalDamage,
            context.Source);

        // 通知伤害事件
        handler.ApplyEffect(context);
    }
}
// Copyright (c) Night Gamer. All rights reserved.
// ==========================
// Target.cs
// 代表技能目标，被攻击或受影响的对象
// 简化版，类似 UE 中被命中的 Actor
// ==========================

public class Target : Actor
{
    public Target(string name) : base(name)
    {
        // 目标属性可以和角色不同（例如怪物有更高防御）
        AttributeSet.SetAttribute(AttributeSet.AttributeType.Defense, 8, this);
    }
}
// Copyright (c) Night Gamer. All rights reserved.
// ==========================
// Program.cs
// 入口主程序，对应UE中 GameMode 或测试入口点
// 用于验证 GAS 系统运行效果，模拟施法、冷却等流程
// ==========================

using System;
using System.Threading;

using System;
using System.Threading;
using static AttributeSet;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== GAS系统测试开始 ===");
        var handler = new EffectHandler();
        handler.OnDamage += (context) =>
            Console.WriteLine($"[战斗日志] {context.Target.Name} 受到 {context.Source.Name} 的 {context.Damage} 点伤害，剩余生命值 {context.Target.AttributeSet.GetAttribute(AttributeType.Health)}");

        // 依赖注入EffectHandler
        var hero = new Character("英雄", handler);
        hero.AttributeSet.InitAttributeSet(100, 100, 50, 50, 20, 10);
        var goblin = new Target("哥布林");
        goblin.AttributeSet.InitAttributeSet(80, 80, 30, 30, 20, 10);

        // 授予技能 
        hero.ASC.GrantAbility(new GameplayAbility_Fireball("GA火球术", 3, 20));

        // 测试技能激活（第一次：正常激活）
        hero.ASC.TryActivateAbility("GA火球术", new AbilityContext(hero, goblin));
        Thread.Sleep(1000);

        // 测试冷却（1秒后再次激活，冷却未结束）
        hero.ASC.TryActivateAbility("GA火球术", new AbilityContext(hero, goblin));
        Thread.Sleep(3000); // 等待冷却结束

        // 测试法力值不足（多次释放后法力值耗尽）
        for (int i = 0; i < 6; i++)
        {
            hero.ASC.TryActivateAbility("GA火球术", new AbilityContext(hero, goblin));
            Thread.Sleep(1000);
        }

        Console.WriteLine("=== GAS系统测试结束 ===");
    }
}
```

