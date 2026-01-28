# LeechingCrystal

## Overview
- **Item ID**: EItem.LeechingCrystal
- **Constructor Address**: 0x18045FD50
- **Category**: Health Management / Tradeoff Item
- **Rarity**: Unknown (not determinable from constructor)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| regenAdditivePerAmount | float | -0.5 | HP regeneration reduction per stack |
| maxHpPerAmount | float | 50.0 | Max HP increase per stack |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 0 | Max Health | amount * 50.0 | Linear scaling |
| 1 | HP Regeneration | amount * -0.5 | Linear scaling (negative) |

## Special Mechanics
- **Tradeoff Design**: Increases maximum health while reducing health regeneration
- **Risk/Reward**: Large health pool with reduced natural healing
- **Stack Behavior**: Both effects scale linearly with stack count

## Formulas
- **Max HP Bonus**: `stack_count * 50.0`
- **HP Regen Penalty**: `stack_count * -0.5`
- **Net Effect**: For each stack, gain 50 max HP but lose 0.5 HP/sec regeneration

## Implementation Details
- **Update Frequency**: On initialization and amount changes only
- **Event Subscriptions**: None (passive stat modifier)
- **Stack Behavior**: Linear scaling for both positive and negative effects

## C# Pseudocode
```csharp
public class ItemLeechingCrystal : ItemBase
{
    private float regenAdditivePerAmount = -0.5f;
    private float maxHpPerAmount = 50.0f;

    protected override void OnInitOrAmountChanged()
    {
        // Set Max HP stat modifier
        SetStat(new StatModifier
        {
            stat = EStat.MaxHealth, // ID 0
            additive = amount * maxHpPerAmount // amount * 50.0
        });

        // Set HP Regeneration stat modifier
        SetStat(new StatModifier
        {
            stat = EStat.HPRegeneration, // ID 1
            additive = amount * regenAdditivePerAmount // amount * -0.5
        });
    }
}
```

## Technical Notes
- Uses standard `ItemBase.SetStat()` method for stat application
- No active behavior - purely passive stat modifications
- Constructor sets fixed values, no runtime calculations
- Simple implementation with direct linear scaling

## Strategic Analysis
- **Early Game**: High value for survivability at low regeneration cost
- **Late Game**: May become problematic if other healing sources aren't available
- **Synergy**: Works well with lifesteal, healing items, or HP-on-kill effects
- **Anti-synergy**: Conflicts with regeneration-focused builds

## Related Items
- **Oats**: Similar max HP bonus (25 per stack) without regeneration penalty
- **HolyBook**: Provides both max HP (100) and regeneration (50) per stack
- **Medkit**: Pure regeneration increase (45 per stack)
- **BeefyRing**: HP-based damage scaling that could synergize with high max HP

---

*Generated from IL2CPP constructor analysis at address 0x18045FD50*