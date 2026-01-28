# GymSauce

## Overview
- **Item ID**: EItem.GymSauce
- **Constructor Address**: 0x18043E590
- **Category**: Damage/Power
- **Rarity**: Common (inferred from simple stat scaling mechanics)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| damagePerAmount | float | 0.1 | 10% damage increase per stack |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 12 | Power/Damage | amount * 0.1 | Linear scaling |

## Special Mechanics
GymSauce is a straightforward damage amplifier with no special behaviors, procs, or conditions. It provides a pure damage increase with no trade-offs, making it one of the simplest and most reliable damage items in the game.

## Formulas
- **Damage Bonus**: `amount × 0.1` (10% per stack)
- **Total Damage Multiplier**: `1.0 + (stacks × 0.1)`

## Implementation Details
- **Update Frequency**: Only updates when item amount changes (OnInitOrAmountChanged)
- **Event Subscriptions**: None - passive stat modifier only
- **Stack Behavior**: Linear scaling with no diminishing returns or caps

## C# Pseudocode
```csharp
public class ItemGymSauce : ItemBase
{
    private float damagePerAmount = 0.1f;

    protected override void OnInitOrAmountChanged()
    {
        // Set damage stat modifier (EStat.Power = 12, ModifierType.ADDITIVE = 2)
        SetStat(new StatModifier
        {
            modifierType = 2,           // ADDITIVE
            statType = 12,              // EStat.Power
            value = amount * damagePerAmount
        });
    }

    // All other methods use base implementation
    public override void Init() { base.Init(); }
    public override void Cleanup() { base.Cleanup(); }
    public override void Tick() { base.Tick(); }
    public override void PreAttack(DamageContainer dc, StatComponents itemAttackModifier) { base.PreAttack(dc, itemAttackModifier); }
    public override void ProcOnHitEffects(DamageContainer dc) { base.ProcOnHitEffects(dc); }
}
```

## Technical Notes
- GymSauce is one of the simplest items in the game, implementing only the essential OnInitOrAmountChanged method
- Uses additive stat modification (type 2) rather than multiplicative, meaning it adds to the base damage rather than multiplying it
- No memory allocations or complex state management required
- Constructor sets damagePerAmount to 0.1 (10%) as a constant value
- The item provides consistent, predictable damage scaling that stacks well with other items

## Related Items
- **Beer**: Higher damage bonus (20% per stack) but with health penalty trade-off
- **TimeBracelet**: Same damage per stack (8% vs 10%) but applies to time-based damage (EStat 32)
- **TacticalGlasses**: Conditional 20% damage bonus against high-health enemies
- **GamerGoggles**: Conditional damage scaling based on player's health percentage
- **Scarf**: Similar linear damage scaling (33% per stack) but conditional on grounded state

---

*Data extracted from IL2CPP constructor at 0x18043E590 and decompiled C# sources*