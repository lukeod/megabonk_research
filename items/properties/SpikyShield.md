# SpikyShield

## Overview
- **Item ID**: EItem.SpikyShield
- **Constructor Address**: 0x180467990
- **Category**: Defensive
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| armorPerAmount | float | 0.1 | Armor bonus per stack |
| retaliationPerArmorPerAmount | float | 200.0 | Retaliation scaling per armor per stack |
| lastStoredArmor | float | Dynamic | Cached armor value for optimization |
| nextUpdateTime | float | Dynamic | Next update timestamp (1 second intervals) |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 4 | Armor | 0.1 | Linear per stack |
| 3 | Thorns/Retaliation | amount × 200.0 × current_armor | Dynamic scaling |

## Special Mechanics
- **Dynamic Retaliation Scaling**: The retaliation damage is recalculated whenever the player's armor value changes
- **Update Frequency**: Checks for armor changes every 1 second via Tick() method
- **Multiplicative Scaling**: Retaliation damage scales with both item stacks AND total armor value

## Formulas
```
Armor Bonus = amount × 0.1
Retaliation Damage = amount × 200.0 × current_armor_value

Where:
- amount = number of SpikyShield stacks
- current_armor_value = player's total armor stat (EStat 4)
```

## Implementation Details
- **Update Frequency**: Every 1.0 seconds
- **Event Subscriptions**: None (uses tick-based updates)
- **Stack Behavior**: Linear armor scaling, multiplicative retaliation scaling with total armor

## C# Pseudocode
```csharp
public class ItemSpikyShield : ItemBase
{
    private float armorPerAmount = 0.1f;
    private float retaliationPerArmorPerAmount = 200.0f;
    private float lastStoredArmor;
    private float nextUpdateTime;

    protected override void OnInitOrAmountChanged()
    {
        // Set armor stat modifier
        SetStat(new StatModifier
        {
            statType = EStat.Armor,
            modifierType = StatModifierType.Additive,
            value = amount * armorPerAmount
        });

        nextUpdateTime = MyTime.time + 1.0f;
    }

    public override void Tick()
    {
        if (MyTime.time >= nextUpdateTime)
        {
            nextUpdateTime = MyTime.time + 1.0f;

            float currentArmor = PlayerStats.GetStatRaw(EStat.Armor);
            if (lastStoredArmor != currentArmor)
            {
                // Update retaliation based on current armor
                SetStat(new StatModifier
                {
                    statType = EStat.Thorns,
                    modifierType = StatModifierType.Additive,
                    value = amount * retaliationPerArmorPerAmount * currentArmor
                });

                lastStoredArmor = currentArmor;
            }
        }
    }
}
```

## Technical Notes
- Uses IL2CPP native method calls for performance-critical operations
- Caches armor value to avoid unnecessary stat updates
- The retaliation calculation creates a positive feedback loop: more armor → more retaliation → more effective defense
- Performance optimized with 1-second update intervals rather than per-frame checks

## Related Items
- **QuinsMask**: Also provides thorns/retaliation damage (20.0 per stack)
- **Chonkplate**: Defensive item with armor-adjacent mechanics (overheal)
- **Mirror**: Reflects damage back to attackers (different mechanism)

---

*Data extracted from decompiled IL2CPP constructor at 0x180467990 and C# interop wrapper*