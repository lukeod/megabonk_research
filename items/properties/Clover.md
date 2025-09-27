# Clover

## Overview
- **Item ID**: ItemClover (from C# class)
- **Constructor Address**: 0x180404E10
- **Category**: Utility/Luck
- **Rarity**: Common (based on simple mechanics)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| luckPerAmount | float | 0.075 | 7.5% luck per stack |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 30 | Luck | amount * 0.075 | Linear |

## Special Mechanics
The Clover is one of the simplest items in MegaBonk, providing a straightforward luck bonus without any complex mechanics, procs, or special behaviors.

- **Pure Stat Modifier**: Only modifies the Luck stat (EStat ID 30)
- **No Event Subscriptions**: Does not subscribe to any game events
- **No Procs**: No chance-based effects or triggers
- **No Special Updates**: No Tick() implementation beyond base class

## Formulas
```
Luck Bonus = amount * 0.075
```

Where:
- `amount` = number of Clover stacks/items
- Base luck per stack = 7.5%

## Implementation Details
- **Update Frequency**: Only updates when amount changes (OnInitOrAmountChanged)
- **Event Subscriptions**: None
- **Stack Behavior**: Linear scaling - each additional stack adds exactly 7.5% luck

## C# Pseudocode
```csharp
// Constructor logic
public ItemClover(ItemInventory itemInventoryRef) : base(itemInventoryRef)
{
    this.luckPerAmount = 0.075f;
}

// Stat application
protected override void OnInitOrAmountChanged()
{
    StatModifier luckMod = new StatModifier();
    luckMod.statType = EStat.Luck; // ID 30
    luckMod.modifierType = 2; // Additive
    luckMod.value = amount * luckPerAmount;
    SetStat(luckMod);
}
```

## Technical Notes
- **IL2CPP Implementation**: Uses native field access for `luckPerAmount` property
- **Memory Layout**: Single float field (0.075) stored in object
- **Performance**: Minimal overhead - only calculates stat on amount changes
- **Threading**: No threading concerns as it's purely stat-based

## Related Items
- **CreditCardGreen**: Also provides luck bonuses but contextual (per chest opened)
- **Key**: Uses luck for proc calculations but doesn't modify luck stat directly

## Luck System Context
In MegaBonk, the Luck stat (EStat 30) appears to influence:
- Chest contents and quality
- Proc chances for various items
- Drop rates from enemies
- Critical hit calculations for some items

The Clover provides the most straightforward way to increase luck, making it valuable for players seeking better RNG outcomes across all game systems.

---

**Data Sources:**
- Constructor: `extracted_constructors/items/Clover.c`
- C# Class: `decompiled/Assembly-CSharp/Assets.Scripts.Inventory__Items__Pickups.Items.ItemImplementations/ItemClover.cs`
- Item Reference: `megabonk_research/items.md`