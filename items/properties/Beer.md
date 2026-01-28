# Beer

## Overview
- **Item ID**: EItem.Beer
- **Constructor Address**: 0x180439EF0
- **Category**: Damage/Power with Health Trade-off
- **Rarity**: Common (inferred from simple stat trade-off mechanics)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| damagePerStack | float | 0.2 | 20% damage increase per stack |
| maxHealthPerStack | float | -0.05 | 5% max health reduction per stack |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 12 | Power/Damage | amount * 0.2 | Linear scaling |
| 0 | Max Health | amount * -0.05 | Linear reduction |

## Special Mechanics
Beer is a straightforward trade-off item with no special behaviors, procs, or conditions. It provides a simple risk/reward mechanic where players gain significant damage at the cost of reduced maximum health.

## Formulas
- **Damage Bonus**: `amount × 0.2` (20% per stack)
- **Health Penalty**: `amount × -0.05` (-5% per stack)
- **Net Effect**: Each beer stack gives +20% damage but reduces max HP by 5%

## Implementation Details
- **Update Frequency**: Only updates when item amount changes (OnInitOrAmountChanged)
- **Event Subscriptions**: None - passive stat modifier only
- **Stack Behavior**: Linear scaling with no diminishing returns or caps

## C# Pseudocode
```csharp
public class ItemBeer : ItemBase
{
    private float damagePerStack = 0.2f;
    private float maxHealthPerStack = -0.05f;

    protected override void OnInitOrAmountChanged()
    {
        // Set damage stat modifier
        SetStat(new StatModifier
        {
            statType = EStat.Power,
            value = amount * damagePerStack
        });

        // Set health reduction modifier
        SetStat(new StatModifier
        {
            statType = EStat.MaxHealth,
            value = amount * maxHealthPerStack
        });
    }
}
```

## Technical Notes
- Beer uses only the base ItemBase functionality with no custom overrides for Init, Cleanup, Tick, PreAttack, or ProcOnHitEffects
- The negative health modifier is applied as a percentage, making it scale with the player's base health
- Implementation is extremely simple - just two stat modifiers that update when stack count changes
- No memory allocations or complex state management required

## Related Items
- **GymSauce**: Similar damage-only boost (10% per stack) without health penalty
- **GlovesBlood**: Another item that trades health-related mechanics for damage
- **LeechingCrystal**: Inverse trade-off (+50 max HP, -0.5 HP regen per stack)
- **GamerGoggles**: Conditional damage boost based on low health

---

*Data sources: extracted_constructors/items/Beer.c, decompiled C# class ItemBeer.cs, megabonk_research/items.md*