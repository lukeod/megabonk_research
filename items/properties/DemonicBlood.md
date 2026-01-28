# DemonicBlood

## Overview
- **Item ID**: ItemDemonicBlood (Assets.Scripts.Inventory__Items__Pickups.Items.ItemImplementations.ItemDemonicBlood)
- **Constructor Address**: 0x1804418A0
- **Category**: Health/Scaling - Stack-based HP bonus item
- **Rarity**: Unknown (not determinable from decompiled code)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| maxStacksPerAmount | int | Function result | Calculated by sub_180331070(this, 0) |
| stacks | int | 0 (initial) | Current accumulated stacks |
| maxStacks | int | Calculated | Maximum stacks possible |
| lastUsedStacks | int | 0 (initial) | Last processed stack count |
| nextUpdateTime | float | 0.0 (initial) | Next time to update stats |
| hpPerStack | static float | Unknown value | HP bonus per stack |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 0 | Max Health | stacks * hpPerStack | Dynamic Linear |

## Special Mechanics
- **Stack Accumulation**: Gains stacks when enemies die (subscribes to A_EnemyDied event)
- **Dynamic Updates**: Updates HP stat every 1.0 second when stacks change
- **Stack Tracking**: Tracks last processed stack count to avoid redundant calculations
- **Event-Driven**: Uses Unity event system for enemy death detection

## Formulas
- **HP Bonus**: `current_stacks * hpPerStack`
- **Update Interval**: 1.0 second between stat recalculations
- **Max Stacks**: Determined by `sub_180331070(this, 0)` function per item amount

## Implementation Details
- **Update Frequency**: 1.0 second intervals (when stacks change)
- **Event Subscriptions**: Enemy.A_EnemyDied (global enemy death event)
- **Stack Behavior**: Stacks accumulate indefinitely up to maxStacks limit
- **Stat Application**: Uses EStat 0 (Max Health) with StatModifier type 2 (additive)

## C# Pseudocode
```csharp
// Constructor logic
public ItemDemonicBlood(ItemInventory itemInventoryRef) : base(itemInventoryRef)
{
    maxStacksPerAmount = CalculateMaxStacks(this, 0); // sub_180331070
    stacks = 0;
    lastUsedStacks = 0;
    nextUpdateTime = 0.0f;
}

// Initialization
public override void Init()
{
    Enemy.A_EnemyDied += OnEnemyDied;
}

// Cleanup
public override void Cleanup()
{
    Enemy.A_EnemyDied -= OnEnemyDied;
}

// Update logic (called every frame)
public override void Tick()
{
    if (MyTime.time >= nextUpdateTime && stacks > lastUsedStacks)
    {
        nextUpdateTime = MyTime.time + 1.0f;

        StatModifier hpMod = new StatModifier
        {
            stat = EStat.MaxHealth, // 0
            type = StatModifierType.Additive, // 2
            value = stacks * hpPerStack
        };

        SetStat(hpMod);
        lastUsedStacks = stacks;
    }
}

// Enemy death handler
private void OnEnemyDied(Enemy enemy, DamageContainer deathSource)
{
    if (stacks < maxStacks)
    {
        stacks++;
    }
}
```

## Technical Notes
- **Performance Optimization**: Only recalculates HP bonus when stacks actually change
- **Memory Management**: Uses IL2CPP interop for native C++ integration
- **Event Safety**: Properly subscribes/unsubscribes from global events in Init/Cleanup
- **Stat System Integration**: Uses the game's StatModifier system for applying HP bonuses
- **Stack Persistence**: Stacks are maintained throughout the game session

## Related Items
- **DemonicSoul**: Similar stacking mechanic but for attack damage instead of HP
- **BeefyRing**: Also provides HP scaling but based on current HP rather than stacks
- **SoulHarvester**: Another enemy death-triggered item with different mechanics

---

*Generated from extracted IL2CPP constructor at 0x1804418A0 and decompiled C# interop code*