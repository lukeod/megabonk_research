# Pumpkin

## Overview
- **Item ID**: EItem.Pumpkin
- **Constructor Address**: 0x180462BC0
- **Category**: Resource/Economy
- **Rarity**: Unknown (not determinable from decompiled code)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| extraPotsPerAmount | int | 8 | Extra pots spawned per stack |
| rewardMultiplierPerAmount | float | 0.18 | Reward multiplier per stack (18%) |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| N/A | N/A | N/A | N/A |

## Special Mechanics

### Extra Pots
- Spawns additional pots per item stack
- Each stack adds 8 extra pots

### Reward Multiplier
- Increases rewards from pots/pickups
- Each stack adds 18% (0.18) to reward multiplier
- Linear scaling with stack count

## Formulas

### Extra Pots
```
extraPots = amount * 8
```
- 1 stack: 8 extra pots
- 2 stacks: 16 extra pots
- 3 stacks: 24 extra pots

### Reward Multiplier
```
rewardMultiplier = amount * 0.18
```
- 1 stack: +18% rewards
- 2 stacks: +36% rewards
- 3 stacks: +54% rewards

## Implementation Details
- **Update Frequency**: Only on amount change (passive item)
- **Event Subscriptions**: None visible in constructor
- **Stack Behavior**: Linear additive scaling

## C# Pseudocode
```csharp
public class ItemPumpkin : ItemBase
{
    private int extraPotsPerAmount = 8;
    private float rewardMultiplierPerAmount = 0.18f;

    public ItemPumpkin(ItemInventory itemInventoryRef) : base(itemInventoryRef)
    {
        extraPotsPerAmount = 8;
        rewardMultiplierPerAmount = 0.18f;
    }

    // Likely used by pot spawning system
    public int GetExtraPots()
    {
        return amount * extraPotsPerAmount;
    }

    // Likely used by reward calculation system
    public float GetRewardMultiplier()
    {
        return amount * rewardMultiplierPerAmount;
    }
}
```

## Technical Notes
- Simple passive item with no active mechanics
- Constructor only sets field values, no event subscriptions
- Likely integrated with pot spawning and reward systems elsewhere
- No OnInitOrAmountChanged method visible in extracted code
- Properties are read by external systems (pot spawners, reward calculators)

## Related Items
- **Coin**: Another economy-related item affecting gold/rewards
- **Magnet**: Affects pickup collection range
- **Clover**: Affects luck/drop rates

---

*Data extracted from decompiled IL2CPP constructor analysis*
