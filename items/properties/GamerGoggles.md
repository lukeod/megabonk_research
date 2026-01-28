# GamerGoggles

## Overview
- **Item ID**: EItem.GamerGoggles
- **Constructor Address**: 0x1804570C0
- **Category**: Damage/Power Enhancement
- **Rarity**: Common/Uncommon (classification based on primary function)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| maxDamagePerAmount | float | 1.0 | Maximum damage bonus per stack |
| updateCooldown | float | 1.0 | Update interval in seconds |
| lastValue | float | -1.0 | Initial tracking value for change detection |
| nextUpdateTime | float | dynamic | Next scheduled update time |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 12 | Power/Damage | `(0.5 - healthPercent) * 2 * maxDamage` when healthPercent < 0.5, otherwise 0 | Dynamic/Conditional |

## Special Mechanics
- **Health-Based Damage Scaling**: Provides damage bonus only when player health is below 50%
- **Dynamic Updates**: Recalculates damage bonus every 1 second based on current health percentage
- **Change Threshold**: Only updates the stat modifier when the calculated value differs from the last value by at least 0.02 (prevents micro-updates)
- **Inverse Health Scaling**: The lower the health percentage (below 50%), the higher the damage bonus

## Formulas

### Primary Damage Formula
```
if (healthPercent < 0.5):
    damageBonus = (0.5 - healthPercent) * 2 * maxDamage
else:
    damageBonus = 0
```

Note: `maxDamage` is calculated in OnInitOrAmountChanged as `amount * maxDamagePerAmount`.

### Health Percentage Calculation
```
healthPercent = currentHP / maxHP
```

### Examples
- At 50% health: 0% damage bonus
- At 40% health: `(0.5 - 0.4) * 2 * 1.0 * stacks = 0.2 * stacks` (20% per stack)
- At 25% health: `(0.5 - 0.25) * 2 * 1.0 * stacks = 0.5 * stacks` (50% per stack)
- At 10% health: `(0.5 - 0.1) * 2 * 1.0 * stacks = 0.8 * stacks` (80% per stack)
- At 1% health: `(0.5 - 0.01) * 2 * 1.0 * stacks = 0.98 * stacks` (98% per stack)

## Implementation Details
- **Update Frequency**: Every 1.0 seconds (not every frame for performance)
- **Event Subscriptions**: Uses Tick() method for periodic updates, not event-driven
- **Stack Behavior**: Linear scaling - each additional stack increases the maximum potential damage bonus by 100%

## C# Pseudocode
```csharp
// Constructor logic
public void Constructor()
{
    maxDamagePerAmount = 1.0f;
    updateCooldown = 1.0f;
    lastValue = -1.0f;
}

// Update logic (called every frame)
public void Tick()
{
    if (Time.time >= nextUpdateTime)
    {
        nextUpdateTime = Time.time + updateCooldown;

        float currentHP = MyPlayer.Instance.inventory.playerHealth.GetCombinedHp();
        float maxHP = MyPlayer.Instance.inventory.playerHealth.GetCombinedMaxHp();
        float healthPercent = currentHP / maxHP;

        float damageBonus = 0.0f;
        if (healthPercent < 0.5f)
        {
            damageBonus = (0.5f - healthPercent) * 2.0f * maxDamage;
        }

        // Only update if change is significant enough
        if (Math.Abs(lastValue - damageBonus) >= 0.02f)
        {
            StatModifier statMod = new StatModifier();
            statMod.stat = EStat.Power; // EStat ID 12
            statMod.value = damageBonus;
            SetStat(statMod);
            lastValue = damageBonus;
        }
    }
}
```

## Technical Notes
- **Performance Optimization**: Uses a 0.02 change threshold to prevent constant stat updates for minor health fluctuations
- **Timer-Based Updates**: Updates every second rather than every frame to reduce computational overhead
- **Null Safety**: Includes multiple null checks for player instance, inventory, and health components
- **Class Initialization**: Properly initializes static type information for System.Math, MyPlayer, MyTime, and StatModifier classes

## Related Items
- **BeefyRing**: Another health-based damage scaling item (scales with current HP, not inverse)
- **Scarf**: Damage bonus based on grounded state (conditional damage like GamerGoggles)
- **TacticalGlasses**: Damage bonus against high-health enemies (>90% health, opposite condition)
- **SpeedBoi**: Another low-health triggered effect (time slowdown at <50% health)

## Synergies
- **High-Risk Builds**: Combines well with items that provide survivability at low health
- **Glass Cannon Builds**: Synergizes with items that reduce max health (like Beer) to stay in the damage bonus range
- **Healing Items**: Pairs well with controlled healing items to maintain optimal health ranges
- **Damage Amplifiers**: Stacks multiplicatively with other damage modifiers for extreme damage potential

---

*Data extracted from decompiled IL2CPP constructor at 0x1804570C0 and Tick method at 0x180456EB0*