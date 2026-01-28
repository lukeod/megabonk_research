# WizardsHat

## Overview
- **Item ID**: EItem.WizardsHat
- **Constructor Address**: 0x1804629A0
- **Category**: Tome Enhancement
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| tomeLevelsPerAmountMax | int | 10 | Maximum tome levels granted per stack |
| tomeLevelsPerAmountMin | int | 2 | Minimum tome levels granted per stack (floor) |
| startFalloffAtAmount | int | 1 | Stack count where falloff begins |
| tomeLevels | int | Dynamic | Accumulated tome levels (calculated) |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | N/A | N/A | No direct stat modifications |

## Special Mechanics
- **Tome Level Enhancement**: Grants additional tome levels which enhance spell/tome abilities
- **Diminishing Returns**: After the first stack, each additional stack grants fewer tome levels
- **Minimum Floor**: Even with heavy stacking, each new stack grants at least 2 tome levels
- **Cumulative Stacking**: Tome levels accumulate across stacks (adds to existing tomeLevels)

## Formulas

### Tome Levels Per Stack Calculation
```
if (amount - startFalloffAtAmount <= 0):
    // First stack (or before falloff threshold)
    levelsToAdd = tomeLevelsPerAmountMax  // 10 levels
else:
    // Additional stacks with falloff
    diminishedValue = tomeLevelsPerAmountMax - (amount - startFalloffAtAmount)
    levelsToAdd = max(diminishedValue, tomeLevelsPerAmountMin)  // min 2 levels

tomeLevels = tomeLevels + levelsToAdd
```

### Simplified Formula
```
Stack 1: +10 tome levels
Stack 2: +max(10 - 1, 2) = +9 tome levels
Stack 3: +max(10 - 2, 2) = +8 tome levels
Stack 4: +max(10 - 3, 2) = +7 tome levels
...
Stack 9+: +2 tome levels (minimum floor reached)
```

## Implementation Details
- **Update Frequency**: On init and when amount changes
- **Event Subscriptions**: OnInitOrAmountChanged hook
- **Stack Behavior**: Diminishing returns with a minimum floor
- **Cumulative**: Adds to existing tomeLevels rather than recalculating from scratch

## C# Pseudocode
```csharp
// Constructor logic
public ItemWizardsHat(ItemInventory itemInventoryRef) {
    this.tomeLevelsPerAmountMax = 10;
    this.tomeLevelsPerAmountMin = 2;
    this.startFalloffAtAmount = 1;
    base(itemInventoryRef);
}

// OnInitOrAmountChanged
protected override void OnInitOrAmountChanged() {
    int currentTomeLevels = this.tomeLevels;
    int maxLevels = this.tomeLevelsPerAmountMax;

    if (this.amount - this.startFalloffAtAmount <= 0) {
        // First stack or before falloff - grant max levels
        this.tomeLevels = currentTomeLevels + maxLevels;
    } else {
        // After falloff threshold - diminishing returns
        int minLevels = this.tomeLevelsPerAmountMin;
        int diminished = maxLevels - (this.amount - this.startFalloffAtAmount);

        // Clamp to minimum
        int levelsToAdd = (diminished >= minLevels) ? diminished : minLevels;
        this.tomeLevels = currentTomeLevels + levelsToAdd;
    }
}
```

## Technical Notes
- **Falloff Mechanic**: The diminishing returns start after `startFalloffAtAmount` (1) stacks
- **Minimum Guarantee**: Each stack always grants at least `tomeLevelsPerAmountMin` (2) levels
- **Cumulative Design**: The code adds to existing tomeLevels rather than recalculating total
- **Integer Math**: All calculations use integer arithmetic

## Scaling Analysis
| Stacks | Levels This Stack | Total Tome Levels |
|--------|-------------------|-------------------|
| 1 | +10 | 10 |
| 2 | +9 | 19 |
| 3 | +8 | 27 |
| 4 | +7 | 34 |
| 5 | +6 | 40 |
| 6 | +5 | 45 |
| 7 | +4 | 49 |
| 8 | +3 | 52 |
| 9 | +2 | 54 |
| 10 | +2 | 56 |
| 15 | +2 | 66 |
| 20 | +2 | 76 |

## Related Items
- **Tome-related items**: Items that interact with or benefit from tome levels
- **Other diminishing return items**: Items using similar falloff mechanics

---

*Data extracted from decompiled IL2CPP constructor at address 0x1804629A0*
