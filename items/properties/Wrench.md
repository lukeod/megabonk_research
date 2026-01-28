# Wrench

## Overview
- **Item ID**: EItem.Wrench
- **Constructor Address**: 0x180468FE0
- **Category**: Utility/Equipment Enhancement
- **Rarity**: Common (based on position in item list)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| chargeSpeedIncrease | float | 0.04 (4%) | Per stack charge speed bonus |
| chargeRewardIncrease | float | 0.075 (7.5%) | Per stack charge reward bonus |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | N/A | N/A | No direct stat modifications |

## Special Mechanics
The Wrench affects the game's equipment charging system through two key mechanics:

### Charge Speed Enhancement
- **Base Value**: 4% per stack
- **Effect**: Reduces the time required to charge equipment at Charge Shrines
- **Calculation**: Applied as a multiplier to charging speed
- **Method**: `GetChargeSpeedIncrease()` returns the cumulative bonus

### Charge Reward Enhancement
- **Base Value**: 7.5% per stack
- **Effect**: Increases the rewards received from completing Charge Shrine interactions
- **Calculation**: Applied as a multiplier to reward values
- **Method**: `GetChargeRewardMultiplier()` returns the cumulative bonus

## Formulas
```
Final Charge Speed = Base Speed * (1 + (amount * 0.04))
Final Charge Reward = Base Reward * (1 + (amount * 0.075))
```

Where `amount` is the number of Wrench stacks.

## Implementation Details
- **Update Frequency**: Passive effect, no periodic updates required
- **Event Subscriptions**: None (passive utility item)
- **Stack Behavior**: Linear scaling - each stack adds the same flat percentage bonus

## C# Pseudocode
```csharp
// Simplified constructor logic
public class ItemWrench : ItemBase
{
    public float chargeSpeedIncrease = 0.04f;    // 4% per stack
    public float chargeRewardIncrease = 0.075f;  // 7.5% per stack

    public float GetChargeSpeedIncrease()
    {
        return amount * chargeSpeedIncrease; // Linear scaling with stack count
    }

    public float GetChargeRewardMultiplier()
    {
        return amount * chargeRewardIncrease; // Linear scaling with stack count
    }
}
```

## Technical Notes
- Simple utility item with no complex proc mechanics or cooldowns
- Affects external game systems (Charge Shrines) rather than combat stats
- Values are stored as floating-point percentages (0.04 = 4%)
- No maximum stack limitations or diminishing returns
- Performance-friendly implementation with minimal overhead

## Related Items
- **Beacon**: Also affects shrine mechanics (increases shrine spawn rate and healing)
- **Key**: Utility item affecting chest/door mechanics
- **Campfire**: Equipment-related item with shrine-like activation mechanics

## Charge Shrine Integration
The Wrench specifically enhances interactions with Charge Shrines, which are special structures in the game that:
- Require the player to stand within range for a duration (`chargeTime`)
- Provide rewards upon completion
- Can be golden variants with enhanced rewards
- Track completion state and timing

The Wrench's bonuses apply when:
1. **Charge Speed**: Reduces the `chargeTime` required to complete shrine charging
2. **Charge Rewards**: Multiplies the final reward value dispensed by the shrine

---
*Data extracted from decompiled IL2CPP constructor at address 0x180468FE0*