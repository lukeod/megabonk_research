# TurboSocks

## Overview
- **Item ID**: EItem.TurboSocks
- **Constructor Address**: 0x18043C780
- **Category**: Movement/Speed Enhancement
- **Rarity**: Unknown (not determinable from available code)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| moveSpeedPerAmount | float | 0.15 | Movement speed bonus per stack |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 25 | Movement Speed | amount * 0.15 | Linear |

## Special Mechanics
TurboSocks is a straightforward movement speed enhancement item with no special proc effects, conditions, or complex behaviors. It provides a simple linear scaling movement speed bonus.

## Formulas
- **Movement Speed Bonus**: `amount * moveSpeedPerAmount`
- **Where**:
  - `amount` = number of TurboSocks stacks
  - `moveSpeedPerAmount` = 0.15 (15% per stack)

## Implementation Details
- **Update Frequency**: On initialization and amount change only
- **Event Subscriptions**: None
- **Stack Behavior**: Additive - each stack provides additional 15% movement speed

## C# Pseudocode
```csharp
public class ItemTurboSocks : ItemBase
{
    private float moveSpeedPerAmount = 0.15f;

    protected override void OnInitOrAmountChanged()
    {
        // Create stat modifier for movement speed (EStat 25)
        StatModifier speedModifier = new StatModifier();
        speedModifier.statType = 25; // Movement Speed
        speedModifier.value = amount * moveSpeedPerAmount;

        // Apply the stat modifier
        SetStat(speedModifier);
    }
}
```

## Technical Notes
- **Performance**: Very lightweight item with no runtime overhead beyond stat calculation
- **Implementation**: Uses the standard ItemBase stat modifier system
- **Precision**: Constructor sets moveSpeedPerAmount to 0.15000001 (float precision artifact)
- **No Dependencies**: Does not interact with other game systems beyond basic stat modification

## Related Items
- **FlappyFeathers**: Provides movement speed boost on jump (speedBoostPerAmount: 1.8)
- **GoldenSneakers**: Generates gold based on distance traveled
- **Rollerblades**: Converts movement speed to attack speed
- **PhantomShroud**: Provides evasion and conditional speed boost
- **CowardsCloak**: Provides speed boost when taking damage

---

**Data Sources:**
- megabonk_research/items.md (line 1083-1096)
- extracted_constructors/items/TurboSocks.c (constructor and OnInitOrAmountChanged implementation)
- decompiled/Assembly-CSharp/Assets.Scripts.Inventory__Items__Pickups.Items.ItemImplementations/ItemTurboSocks.cs (class structure)