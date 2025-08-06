C#手写实现UE5 GAS简化版框架，包括ASC GE GA AS等核心基础模块。

```cpp
using System;
using System.Collections.Generic;
using System.Threading;

// =======================
// GameplayEffect
// =======================
public abstract class GameplayEffectBase
{
    public string Name { get; private set; }

    protected GameplayEffectBase(string name)
    {
        Name = name;
    }

    public abstract void Apply(AbilityContext context, EffectHandler handler);
}

public class DamageEffect : GameplayEffectBase
{
    private int Damage;

    public DamageEffect(string name, int damage) : base(name)
    {
        Damage = damage;
    }

    public override void Apply(AbilityContext context, EffectHandler handler)
    {
        handler.ApplyEffect(context.Source.Name, context.Target.Name, Damage);
    }
}

// =======================
// GameplayAbility
// =======================
public abstract class GameplayAbilityBase
{
    public string Name { get; private set; }
    public int CooldownSeconds { get; private set; }

    private DateTime lastUsed = DateTime.MinValue;

    public GameplayAbilityBase(string name, int cooldown)
    {
        Name = name;
        CooldownSeconds = cooldown;
    }

    public bool CanActivate()
    {
        return (DateTime.Now - lastUsed).TotalSeconds >= CooldownSeconds;
    }

    public void Activate(AbilityContext context, AbilitySystemComponent asc)
    {
        if (!CanActivate())
        {
            Console.WriteLine($"[GA] {Name} is on cooldown.");
            return;
        }

        lastUsed = DateTime.Now;
        ExecuteAbility(context, asc);
    }

    protected abstract void ExecuteAbility(AbilityContext context, AbilitySystemComponent asc);
}

public class FireballAbility : GameplayAbilityBase
{
    public FireballAbility() : base("Fireball", 3) { }

    protected override void ExecuteAbility(AbilityContext context, AbilitySystemComponent asc)
    {
        var spec = new DamageEffect("FireballExplosion", context.BaseDamage);
        asc.ApplyGameplayEffectToSelf(spec, context);
    }
}

// =======================
// ASC
// =======================
public class AbilitySystemComponent
{
    private Dictionary<string, GameplayAbilityBase> Abilities = new();
    private EffectHandler Handler;

    public AbilitySystemComponent(EffectHandler handler)
    {
        Handler = handler;
    }

    public void GrantAbility(GameplayAbilityBase ability)
    {
        Abilities[ability.Name] = ability;
    }

    public void TryActivateAbility(string abilityName, AbilityContext context)
    {
        if (Abilities.TryGetValue(abilityName, out var ability))
        {
            ability.Activate(context, this);
        }
        else
        {
            Console.WriteLine($"[ASC] Ability {abilityName} not found.");
        }
    }

    public void ApplyGameplayEffectToSelf(GameplayEffectBase effect, AbilityContext context)
    {
        effect.Apply(context, Handler);
    }
}

// =======================
// 上下文数据结构
// =======================
public class AbilityContext
{
    public Actor Source { get; private set; }
    public Target Target { get; private set; }
    public int BaseDamage { get; private set; }

    public AbilityContext(Actor source, Target target, int damage)
    {
        Source = source;
        Target = target;
        BaseDamage = damage;
    }
}

// =======================
// Actor 与 Target
// =======================
public class Actor
{
    public string Name { get; private set; }
    public AbilitySystemComponent ASC { get; private set; }

    public Actor(string name, EffectHandler handler)
    {
        Name = name;
        ASC = new AbilitySystemComponent(handler);
    }
}

public class Target
{
    public string Name { get; private set; }

    public Target(string name) => Name = name;
}

// =======================
// 效果处理器（含委托）
// =======================
public class EffectHandler
{
    public delegate void DamageEvent(string source, string target, int damage);
    public event DamageEvent OnDamage;

    public void ApplyEffect(string source, string target, int damage)
    {
        Console.WriteLine($"[Effect] {source} dealt {damage} damage to {target}");
        OnDamage?.Invoke(source, target, damage);
    }
}

// =======================
// 主函数
// =======================
class Program
{
    static void Main()
    {
        var handler = new EffectHandler();
        handler.OnDamage += (src, tgt, dmg) => Console.WriteLine($"[Log] {tgt} 被 {src} 击中，伤害 {dmg}");

        var hero = new Actor("Hero", handler);
        var goblin = new Target("Goblin");

        // 授权技能（GrantAbility）
        hero.ASC.GrantAbility(new FireballAbility());

        // 激活技能（TryActivateAbility）
        hero.ASC.TryActivateAbility("Fireball", new AbilityContext(hero, goblin, 120));
        Thread.Sleep(1000);
        hero.ASC.TryActivateAbility("Fireball", new AbilityContext(hero, goblin, 100)); // 冷却中
        Thread.Sleep(3000);
        hero.ASC.TryActivateAbility("Fireball", new AbilityContext(hero, goblin, 150));
    }
}

```

