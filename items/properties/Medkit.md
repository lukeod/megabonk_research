# Medkit

## Overview
- **Item ID**: EItem.Medkit
- **Constructor Address**: 0x180415C40
- **Category**: Health/Healing
- **Rarity**: N/A (determined from game data)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| hpRegenPerAmount | float | 45.0 | HP regeneration per stack |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 1 | HP Regeneration | amount * 45.0 | Linear |

## Special Mechanics
The Medkit is a straightforward healing item with no special behaviors or procs. It simply provides a flat HP regeneration bonus that scales linearly with the number of stacks.

## Formulas
- **HP Regeneration**: `amount * hpRegenPerAmount` where `hpRegenPerAmount = 45.0`
- **Final HP Regen**: `stack_count * 45.0`

## Implementation Details
- **Update Frequency**: Passive (no tick-based updates)
- **Event Subscriptions**: None
- **Stack Behavior**: Linear stacking - each additional stack adds exactly 45.0 HP regeneration

## C# Pseudocode
```csharp
public class ItemMedkit : ItemBase
{
    public float hpRegenPerAmount = 45.0f;

    protected override void OnInitOrAmountChanged()
    {
        // Create stat modifier for HP regeneration
        StatModifier regenModifier = new StatModifier();
        regenModifier.statType = EStat.HPRegeneration; // ID: 1
        regenModifier.modifierType = StatModifierType.Additive; // ID: 2
        regenModifier.value = amount * hpRegenPerAmount;

        // Apply the stat modifier
        SetStat(regenModifier);
    }
}
```

## Technical Notes
- This is one of the simplest items in the game with minimal implementation complexity
- No special mechanics, event handling, or complex calculations
- The item uses the standard ItemBase framework for stat modification
- HP regeneration is applied as an additive modifier (type 2) to EStat 1
- No performance considerations as it's a passive stat boost

## Related Items
Items with similar mechanics or synergies:
- **HolyBook**: Also provides HP regeneration (50.0 per stack) plus max HP and overheal
- **LeechingCrystal**: Provides max HP but reduces HP regeneration (-0.5 per stack)
- **Campfire**: Provides time-based healing when stationary (1100.0 per minute per stack)
- **Oats**: Provides max HP but no regeneration

---

*Data extracted from:*
- *Constructor: D:\dev\megabonk\extracted_constructors\items\Medkit.c*
- *Decompiled C#: D:\dev\megabonk\decompiled\Assembly-CSharp\Assets.Scripts.Inventory__Items__Pickups.Items.ItemImplementations\ItemMedkit.cs*
- *Items overview: D:\dev\megabonk\megabonk_research\items.md*