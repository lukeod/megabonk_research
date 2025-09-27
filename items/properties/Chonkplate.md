# Chonkplate

## Overview
- **Item ID**: EItem.Chonkplate
- **Constructor Address**: 0x180404BB0
- **Category**: Defensive/Health Enhancement
- **Rarity**: Standard Item

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| overhealPerAmount | float | 0.75 | Overheal bonus per stack |
| lifestealPerAmount | float | 0.2 | Lifesteal bonus per stack |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 47 | Overheal | amount * 0.75 | Linear scaling |
| 17 | Lifesteal | amount * 0.2 | Linear scaling |

## Special Mechanics
Chonkplate is a straightforward defensive item that provides passive stat bonuses without any proc-based or event-driven mechanics. It combines two key survivability stats:

1. **Overheal**: Allows the player to heal beyond maximum HP, creating a temporary buffer
2. **Lifesteal**: Restores health based on damage dealt to enemies

The item has no special conditions, cooldowns, or reactive behaviors - it simply provides flat stat increases that scale linearly with stack count.

## Formulas
- **Overheal Bonus**: `stacks × 0.75`
- **Lifesteal Bonus**: `stacks × 0.2` (20% per stack)

## Implementation Details
- **Update Frequency**: Static (no continuous updates)
- **Event Subscriptions**: None
- **Stack Behavior**: Linear scaling for both stats

## C# Pseudocode
```csharp
public class ItemChonkplate : ItemBase
{
    public float overhealPerAmount = 0.75f;
    public float lifestealPerAmount = 0.2f;

    protected override void OnInitOrAmountChanged()
    {
        // Apply overheal stat modifier
        SetStat(new StatModifier
        {
            statType = EStat.Overheal,      // ID 47
            modifierType = StatModifierType.Additive,
            value = amount * overhealPerAmount
        });

        // Apply lifesteal stat modifier
        SetStat(new StatModifier
        {
            statType = EStat.Lifesteal,     // ID 17
            modifierType = StatModifierType.Additive,
            value = amount * lifestealPerAmount
        });
    }
}
```

## Technical Notes
- Simple implementation with only constructor and stat initialization
- No override methods for Tick(), PreAttack(), or ProcOnHitEffects()
- Uses standard StatModifier system for both stats
- No special memory allocation or performance considerations
- Both properties are applied with StatModifierType.Additive (ID 2)

## Related Items
**Similar Defensive Items**:
- **HolyBook**: Also provides overheal (0.25 per stack) plus max HP and HP regen
- **Gasmask**: Provides conditional overheal based on poisoned enemies
- **SpikyShield**: Defensive item focusing on armor and retaliation

**Lifesteal Synergies**:
- **DemonBlade**: Provides crit chance and heal on crit
- **BloodyCleaver**: Proc-based bleeding effect triggered by lifesteal
- **GlovesBlood**: Explosive healing effects

**Stack Scaling Comparison**:
- Chonkplate: 75% overheal + 20% lifesteal per stack
- HolyBook: 25% overheal + 100 max HP + 50 HP regen per stack
- Most defensive items scale linearly without diminishing returns

---

*Data extracted from IL2CPP constructor analysis and decompiled C# code*