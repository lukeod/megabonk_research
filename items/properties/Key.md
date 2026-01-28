# Key

## Overview
- **Item ID**: EItem.Key
- **Constructor Address**: 0x18045FA00
- **Category**: Utility / Chance-Based
- **Rarity**: Unknown (not determinable from constructor)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| chancePerStack | float | 0.1 | 10% chance per stack |
| currentChance | float | 0.1 | Calculated current chance (initial value) |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| N/A | No direct stat modifications | N/A | Item uses chance-based mechanics |

## Special Mechanics
- **Chance Stacking**: Uses hyperbolic scaling to prevent chance from exceeding 100%
- **Stack Behavior**: Each stack contributes 10% to the hyperbolic formula
- **Scaling Cap**: Hyperbolic curve with max of 1.0 (100%)

## Formulas

### Current Chance (Hyperbolic Scaling)
```
rawChance = amount * chancePerStack
currentChance = HyperbolicScaling(rawChance, 1.0, 1.0)
```
- **Input**: `amount * 0.1` (10% per stack)
- **Max Factor**: 1.0 (100% theoretical max)
- **Base Factor**: 1.0

### Hyperbolic Formula
```
currentChance = rawChance / (rawChance + 1.0) * 1.0
```

### Example Stack Values
| Stacks | Raw Chance | Current Chance |
|--------|------------|----------------|
| 1 | 0.1 | ~9.1% |
| 2 | 0.2 | ~16.7% |
| 3 | 0.3 | ~23.1% |
| 5 | 0.5 | ~33.3% |
| 10 | 1.0 | ~50.0% |
| 20 | 2.0 | ~66.7% |

## Implementation Details
- **Update Frequency**: On initialization and amount changes only
- **Event Subscriptions**: None visible in constructor
- **Stack Behavior**: Hyperbolic scaling for diminishing returns

## C# Pseudocode
```csharp
public class ItemKey : ItemBase
{
    private float chancePerStack = 0.1f;
    private float currentChance = 0.1f;

    protected override void OnInitOrAmountChanged()
    {
        float rawChance = amount * chancePerStack;
        currentChance = rawChance;
        currentChance = StatScaling.HyperbolicScaling(rawChance, 1.0f, 1.0f);
    }
}
```

## Technical Notes
- Uses standard `StatScaling.HyperbolicScaling()` method for chance calculation
- Constructor sets both `chancePerStack` and `currentChance` to 0.1
- The `currentChance` field is recalculated on every stack change
- Hyperbolic scaling parameters (1.0, 1.0) allow approaching but never reaching 100%
- Simple implementation focused purely on chance calculation

## Related Items
- Items using hyperbolic scaling for proc chances
- Other utility items with chance-based effects

---

*Generated from IL2CPP constructor analysis at address 0x18045FA00*
