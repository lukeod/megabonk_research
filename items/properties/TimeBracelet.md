# TimeBracelet

## Overview
- **Item ID**: EItem.TimeBracelet
- **Constructor Address**: 0x180439450
- **Category**: Time/Temporal Damage
- **Rarity**: [Unknown from available data]

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| damagePerAmount | float | 0.08 | Damage multiplier per stack |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 32 | Time-based Damage | amount * 0.08 | Linear |

## Special Mechanics
- Simple time-based damage amplifier
- No special behaviors, procs, or conditions
- Purely passive damage enhancement through time-based damage stat

## Formulas
- **Time Damage Bonus** = `amount * damagePerAmount`
- **Final Time Damage** = `amount * 0.08`

## Implementation Details
- **Update Frequency**: Only on initialization or amount change
- **Event Subscriptions**: None
- **Stack Behavior**: Linear scaling - each stack adds 0.08 to time damage

## C# Pseudocode
```csharp
public class ItemTimeBracelet : ItemBase
{
    public float damagePerAmount = 0.08f;

    protected override void OnInitOrAmountChanged()
    {
        // Create StatModifier for time damage (EStat 32)
        var statModifier = new StatModifier();
        statModifier.type = EStatModifierType.Additive; // Type 2
        statModifier.stat = EStat.TimeDamage; // EStat 32
        statModifier.value = amount * damagePerAmount;

        SetStat(statModifier);
    }
}
```

## Technical Notes
- One of the simplest items in the game with minimal implementation
- Only modifies a single stat with no additional mechanics
- The `damagePerAmount` field is set to 0.079999998 in the constructor (stored as float precision of 0.08)
- EStat 32 (Time-based Damage) is a specialized damage type that may interact with time-manipulation mechanics
- No virtual method overrides beyond the required base methods

## Related Items
- **SpeedBoi**: Also has time-manipulation mechanics with slowdown effects
- **ZaWarudo**: Likely time-related based on name (JoJo reference to time stop)
- Other damage amplifiers: **Beer**, **GymSauce**, **TacticalGlasses**

---

*Data extracted from decompiled IL2CPP constructor at address 0x180439450 and C# class Assets.Scripts.Inventory__Items__Pickups.Items.ItemImplementations.ItemTimeBracelet*