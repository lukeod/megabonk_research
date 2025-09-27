# DemonicSoul

## Overview
- **Item ID**: EItem.DemonicSoul
- **Constructor Address**: 0x180408150
- **Category**: Damage/Power (Stack-based)
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| maxStacksPerAmount | int | sub_180338070(this, 0) | Dynamic calculation based on function |
| stacks | int | 0 | Current stack count |
| maxStacks | int | amount * maxStacksPerAmount | Maximum stacks possible |
| lastUsedStacks | int | 0 | Tracking for stat updates |
| nextUpdateTime | float | 0.0 | Next tick time for stat updates |
| attackDamagePerStack | static float | Unknown | Static field, value not determined |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 12 | Power/Damage | stacks * attackDamagePerStack | Linear per stack |

## Special Mechanics
- **Stack Acquisition**: Gains stacks when enemies die (subscribes to Enemy.A_EnemyDied event)
- **Dynamic Stat Updates**: Updates power/damage stat every 1 second if stacks increased
- **Stack Management**: Automatically caps stacks at maxStacks value
- **Persistent Scaling**: Damage bonus persists as long as stacks are maintained

## Formulas
```
maxStacks = amount * maxStacksPerAmount
damageBonus = stacks * attackDamagePerStack
```

## Implementation Details
- **Update Frequency**: 1 second intervals (nextUpdateTime = currentTime + 1.0)
- **Event Subscriptions**:
  - Init: Subscribes to Enemy.A_EnemyDied
  - Cleanup: Unsubscribes from Enemy.A_EnemyDied
- **Stack Behavior**: Stacks accumulate indefinitely up to maxStacks limit
- **Stat Update**: Only updates stat modifiers when stacks > lastUsedStacks

## C# Pseudocode
```csharp
public class ItemDemonicSoul : ItemBase
{
    private static readonly float attackDamagePerStack; // Unknown value
    private int maxStacksPerAmount;
    private int stacks;
    private int maxStacks;
    private int lastUsedStacks;
    private float nextUpdateTime;

    // Constructor
    public ItemDemonicSoul(ItemInventory itemInventoryRef)
    {
        this.maxStacksPerAmount = CalculateMaxStacksPerAmount(); // sub_180338070
        base(itemInventoryRef);
    }

    // Initialize event subscriptions
    public override void Init()
    {
        Enemy.A_EnemyDied += OnEnemyDied;
    }

    // Clean up event subscriptions
    public override void Cleanup()
    {
        Enemy.A_EnemyDied -= OnEnemyDied;
    }

    // Update stack limits when amount changes
    protected override void OnInitOrAmountChanged()
    {
        int newMaxStacks = amount * maxStacksPerAmount;
        maxStacks = newMaxStacks;
        if (stacks > newMaxStacks)
            stacks = newMaxStacks;
    }

    // Gain stack on enemy death
    private void OnEnemyDied(Enemy enemy, DamageContainer deathSource)
    {
        if (stacks < maxStacks)
            stacks++;
    }

    // Update damage stat periodically
    public override void Tick()
    {
        if (Time.time >= nextUpdateTime)
        {
            nextUpdateTime = Time.time + 1.0f;

            if (stacks > lastUsedStacks)
            {
                StatModifier damageModifier = new StatModifier
                {
                    stat = EStat.Power, // ID 12
                    addType = StatAddType.Additive, // 0
                    value = stacks * attackDamagePerStack
                };

                SetStat(damageModifier);
                lastUsedStacks = stacks;
            }
        }
    }
}
```

## Technical Notes
- **IL2CPP Interop**: Uses IL2CPP runtime for native code integration
- **Event-Driven Architecture**: Reactive design based on Unity's event system
- **Performance Optimization**: Only updates stats when necessary (stack increases)
- **Memory Management**: Proper event subscription/unsubscription lifecycle
- **Stack Overflow Protection**: Built-in maximum stack limits prevent unbounded growth

## Related Items
- **DemonicBlood**: Similar stack-based mechanics but affects HP (EStat 0) instead of damage
- **BeefyRing**: Also uses sub_180338070 for maxStacksPerAmount calculation
- **Stack-based Items**: Items that accumulate permanent bonuses through gameplay events

## Missing Information
- **attackDamagePerStack Value**: Static field value not available in decompiled sources
- **sub_180338070 Function**: Implementation details of maxStacksPerAmount calculation
- **OnEnemyDied Implementation**: Actual logic for stack gaining conditions
- **Localization Keys**: Item name and description strings

---

*Data sources: extracted_constructors/items/DemonicSoul.c, decompiled C# files, megabonk_research/items.md*
*Generated from IL2CPP decompilation and IDA Pro analysis*