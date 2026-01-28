# TacticalGlasses

## Overview
- **Item ID**: EItem.TacticalGlasses
- **Constructor Address**: 0x180467BE0
- **Category**: Damage/Power
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| damagePerAmount | float | 0.2 | 20% damage bonus per stack |
| additiveDamage | float | Dynamic | Calculated in PreAttack |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | None | N/A | Direct damage modification |

## Special Mechanics
- **High HP Target Bonus**: Provides damage bonus only against enemies with >90% health
- **Conditional Damage**: Only applies bonus when enemy HP ratio > 0.9 (90%)
- **PreAttack Hook**: Uses PreAttack method to conditionally modify damage
- **Additive Damage**: Adds bonus damage directly to attack modifier rather than as a stat

## Formulas
- **Damage Bonus**: `amount * 0.2` (20% per stack)
- **HP Threshold**: Enemy HP must be > 90% for bonus to apply
- **Total Bonus**: `damagePerAmount * amount` applied as additive damage

## Implementation Details
- **Update Frequency**: Per attack (PreAttack method)
- **Event Subscriptions**: None (direct method override)
- **Stack Behavior**: Linear scaling - each stack adds 20% damage
- **Damage Application**: Uses `StatComponents.AddAdditive()` for damage modification

## C# Pseudocode
```csharp
// Constructor
public ItemTacticalGlasses(ItemInventory itemInventoryRef)
{
    this.damagePerAmount = 0.2f;
    base(itemInventoryRef);
}

// PreAttack method
public override void PreAttack(DamageContainer dc, StatComponents itemAttackModifier)
{
    if (dc?.enemy != null)
    {
        float enemyHpRatio = dc.enemy.GetHpRatio();

        // Only apply bonus if enemy has >90% health
        if (enemyHpRatio >= 0.9f)
        {
            if (itemAttackModifier != null)
            {
                // Apply additive damage bonus
                itemAttackModifier.AddAdditive(this.additiveDamage);
            }
        }
    }
}
```

## Technical Notes
- Unlike most items, TacticalGlasses doesn't use the standard stat modifier system
- The damage bonus is applied conditionally through the PreAttack method
- The item specifically targets high-health enemies, making it effective against fresh spawns and bosses
- The 90% threshold means even minor damage will disable the bonus
- Uses additive damage calculation rather than multiplicative

## Related Items
- **BrassKnuckles**: Also provides conditional damage based on proximity
- **Scarf**: Provides conditional damage based on grounded state
- **GamerGoggles**: Provides damage bonus at low health (opposite condition)

---

**Data Sources**:
- megabonk_research/items.md (line 1038-1049)
- extracted_constructors/items/TacticalGlasses.c
- decompiled/Assembly-CSharp/.../ItemTacticalGlasses.cs