# Skuleg

## Overview
- **Item ID**: EItem.Skuleg
- **Constructor Address**: 0x180464160
- **Category**: Difficulty Modifier / Meta Progression
- **Rarity**: Unknown
- **Base Class**: ItemBase (inherit standard item behavior)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| difficultyPerAmount | float | 0.07 | 7% difficulty increase per stack |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 38 | Difficulty | amount * 0.07 | Linear per stack |

## Special Mechanics
- **Pure Difficulty Scaling**: Skuleg is a simple difficulty modifier item with no complex mechanics
- **No Event Subscriptions**: Does not respond to any game events (damage, death, etc.)
- **No Proc Effects**: No chance-based or triggered abilities
- **No Update Loop**: Does not use Tick() method for continuous effects
- **Additive Stacking**: Multiple Skuleg items stack additively

## Formulas
- **Difficulty Increase**: `amount × 0.07`
- **Total Difficulty**: Base Game Difficulty + (Skuleg Stacks × 0.07)

## Implementation Details
- **Update Frequency**: Only on initialization and amount changes
- **Event Subscriptions**: None
- **Stack Behavior**: Linear additive stacking
- **Stat Modifier Type**: StatModifierType.Additive (ID: 2)
- **EStat Target**: EStat 38 (Difficulty)

## C# Pseudocode
```csharp
public class ItemSkuleg : ItemBase
{
    public float difficultyPerAmount = 0.07f;

    protected override void OnInitOrAmountChanged()
    {
        // Create difficulty stat modifier
        StatModifier difficultyMod = new StatModifier
        {
            statType = 38,           // Difficulty EStat
            modifierType = 2,        // Additive
            value = amount * difficultyPerAmount  // 7% per stack
        };

        SetStat(difficultyMod);
    }

    // All other methods (Tick, PreAttack, ProcOnHitEffects) are empty
}
```

## Technical Notes
- **Simplest Item Implementation**: Skuleg is one of the most straightforward items in the game
- **One-Line Logic**: The entire functionality is a single stat modification
- **No Performance Impact**: No continuous calculations or event handling
- **Immediate Effect**: Difficulty change applies instantly when item is acquired
- **No Side Effects**: Only affects difficulty stat, no other game mechanics

## Related Items
- **Beer**: Another stat modifier item (damage increase with health penalty)
- **GlovesCursed**: More complex difficulty modifier with additional effects
- **Other Meta Items**: Items that affect game progression rather than direct combat

## Game Impact
- **Increases Enemy Scaling**: Higher difficulty typically means stronger enemies
- **Risk vs Reward**: Players might take Skuleg for increased challenge and potentially better rewards
- **Speedrun Considerations**: May be avoided in speedruns due to increased difficulty
- **Build Synergy**: Could synergize with items that benefit from difficult encounters

---

*Data extracted from IL2CPP decompiled constructor at address 0x180464160 and C# decompiled source files*