# BeefyRing

## Overview
- **Item ID**: EItem.BeefyRing
- **Constructor Address**: 0x180439C30
- **Category**: Health-based Damage Scaling
- **Rarity**: Common/Standard

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| maxHpPerStack | int | 10 | HP gained per item stack |
| powerPerHpPerAmount | float | 0.002 | Power multiplier per HP per stack |
| lastStoredMaxHp | int | 0 | Cached max HP value for optimization |
| nextUpdateTime | float | time + 1.0 | Next update timestamp |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 0 | Max Health | amount * 10 | Linear per stack |
| 12 | Power/Damage | currentHP * 0.002 * amount | Dynamic scaling |

## Special Mechanics
- **Dynamic Power Scaling**: Power increases based on current HP, not max HP
- **Regular Updates**: Recalculates power every 1 second via Tick() method
- **HP Tracking**: Monitors changes in max HP to trigger power recalculation
- **Optimization**: Only recalculates when max HP actually changes

## Formulas
### Health Bonus
```
Health Bonus = Stack Count × 10
```

### Power Bonus
```
Power Bonus = Current HP × 0.002 × Stack Count
```

### Example Calculations
- With 3 stacks: +30 max HP
- At full HP (100 base + 30 bonus = 130): +0.78 power (130 × 0.002 × 3)
- At 50% HP (65): +0.39 power (65 × 0.002 × 3)

## Implementation Details
- **Update Frequency**: Every 1.0 seconds
- **Event Subscriptions**: None (passive item)
- **Stack Behavior**: Linear scaling for both HP and power multiplier
- **Performance**: Optimized with caching - only updates when max HP changes

## C# Pseudocode
```csharp
// Constructor - sets base values
public ItemBeefyRing(ItemInventory itemInventoryRef) : base(itemInventoryRef)
{
    maxHpPerStack = 10;
    powerPerHpPerAmount = 0.002f;
}

// Called when item is added or stack count changes
protected override void OnInitOrAmountChanged()
{
    // Set max HP stat modifier
    var hpModifier = new StatModifier()
    {
        stat = EStat.MaxHealth,
        type = StatModifierType.Additive,
        value = amount * maxHpPerStack
    };
    SetStat(hpModifier);

    // Schedule next update in 1 second
    nextUpdateTime = MyTime.time + 1.0f;
}

// Called every frame to check for updates
public override void Tick()
{
    if (MyTime.time >= nextUpdateTime)
    {
        nextUpdateTime = MyTime.time + 1.0f;

        int currentMaxHp = PlayerStats.GetStat(EStat.MaxHealth);
        if (currentMaxHp != lastStoredMaxHp)
        {
            // Max HP changed, update power
            var powerModifier = new StatModifier()
            {
                stat = EStat.Power,
                type = StatModifierType.Additive,
                value = currentMaxHp * powerPerHpPerAmount * amount
            };
            SetStat(powerModifier);
            lastStoredMaxHp = currentMaxHp;
        }
    }
}
```

## Technical Notes
- **Memory Optimization**: Uses caching to avoid unnecessary recalculations
- **Performance**: Tick method only performs work when update timer expires
- **Stat System**: Uses the game's StatModifier system for clean stat management
- **IL2CPP Compatibility**: Native implementation maintains performance in IL2CPP builds

## Related Items
### Synergistic Items
- **HolyBook**: Provides massive HP bonus (+100 per stack) for power scaling
- **Oats**: Provides moderate HP bonus (+25 per stack) for power scaling
- **LeechingCrystal**: Provides HP bonus (+50 per stack) despite HP regen penalty
- **Chonkplate**: Provides overheal which may factor into power calculations

### Anti-Synergistic Items
- **Beer**: Reduces max HP (-5% per stack), decreasing power potential
- **GlovesBlood**: May reduce HP through damage taken

### Similar Mechanics
- **GamerGoggles**: Also provides dynamic damage based on HP percentage (inverse relationship)
- **PhantomShroud**: Conditional stat bonuses triggered by game state

---
*Data extracted from decompiled IL2CPP constructors and C# bindings via IDA Pro MCP analysis*