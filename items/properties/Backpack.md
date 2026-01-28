# Backpack

## Overview
- **Item ID**: EItem.Backpack
- **Constructor Address**: 0x180438D70
- **Category**: Utility/Support
- **Rarity**: [Not determinable from constructor code]

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| projectilesPerAmount | int | 1 | Projectiles added per stack |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 16 | Projectiles | amount | Linear scaling |

## Special Mechanics
The Backpack item provides a straightforward projectile count increase. Each stack adds one additional projectile to the player's attacks.

## Formulas
- **Projectile Count**: base_projectiles + (amount * 1)
- **Stat Modifier Value**: (float)amount

## Implementation Details
- **Update Frequency**: Only when initialized or amount changes (OnInitOrAmountChanged)
- **Event Subscriptions**: None - passive stat modifier only
- **Stack Behavior**: Linear - each stack adds exactly 1 projectile

## C# Pseudocode
```csharp
// Simplified constructor logic
public class ItemBackpack : ItemBase
{
    public int projectilesPerAmount = 1;

    public ItemBackpack(ItemInventory itemInventoryRef) : base(itemInventoryRef)
    {
        this.projectilesPerAmount = 1;
    }

    protected override void OnInitOrAmountChanged()
    {
        // Create new stat modifier for projectiles (EStat 16)
        StatModifier statMod = new StatModifier();
        statMod.operationType = 2;  // Additive modifier
        statMod.stat = 16;          // EStat.Projectiles
        statMod.value = (float)this.amount;  // Linear scaling
        SetStat(statMod);
    }
}
```

## Technical Notes
- Uses **additive stat modification** (operationType = 2) rather than multiplicative
- The item is purely passive - no event handlers, tick updates, or proc effects
- Inherits standard ItemBase functionality for amount tracking and stat management
- Constructor sets projectilesPerAmount to 1, though this value is currently not used in calculations (the direct amount is used instead)

## Related Items
- **Items with similar scaling**: Many items use linear amount-based scaling
- **Other projectile modifiers**: Items that affect EStat 16 (Projectiles)
- **Utility category items**: Beacon, Campfire, Clover, Ghost, Key, Wrench

---

*Data extracted from IL2CPP constructor at 0x180438D70 and decompiled C# source*