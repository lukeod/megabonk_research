# Oats

## Overview
- **Item ID**: EItem.Oats
- **Constructor Address**: 0x180421AC0
- **Category**: Health/Healing
- **Rarity**: Common

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| hpPerAmount | float | 25.0 | HP bonus per stack |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 0 | Max Health | amount * 25.0 | Linear |

## Special Mechanics
- Pure passive health bonus item
- No proc effects, cooldowns, or special behaviors
- Simple linear scaling with stack count

## Formulas
```
Max HP Bonus = amount * 25.0
```

Where:
- `amount` = number of item stacks owned

## Implementation Details
- **Update Frequency**: Only on amount change
- **Event Subscriptions**: None
- **Stack Behavior**: Additive (each stack adds exactly 25 HP)

## C# Pseudocode
```csharp
public class ItemOats : ItemBase
{
    private float hpPerAmount = 25.0f;

    protected override void OnInitOrAmountChanged()
    {
        // Create stat modifier for max health (EStat 0)
        var healthModifier = new StatModifier
        {
            stat = EStat.MaxHealth,  // EStat ID 0
            additive = amount * hpPerAmount,
            multiplicative = 1.0f
        };

        // Apply the stat modifier
        SetStat(healthModifier);
    }
}
```

## Technical Notes
- Oats is one of the simplest items in the game with minimal implementation
- No override methods beyond the base `OnInitOrAmountChanged`
- Uses the standard `SetStat` mechanism from `ItemBase`
- No special initialization, cleanup, tick, or combat mechanics
- Memory efficient due to its simplicity

## Related Items
- **LeechingCrystal**: Also provides Max HP (+50 per stack) but reduces HP regen
- **BeefyRing**: Provides Max HP (+10 per stack) plus dynamic damage based on current HP
- **HolyBook**: Comprehensive health item with Max HP (+100), HP regen (+50), and overheal (+0.25)
- **Beer**: Negative health item that reduces Max HP (-0.05 per stack) for damage bonus

---

*Data extracted from decompiled IL2CPP constructor analysis and C# interop code*