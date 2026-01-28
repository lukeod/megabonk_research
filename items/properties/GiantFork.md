# GiantFork

## Overview
- **Item ID**: ItemGiantFork (Assets.Scripts.Inventory__Items__Pickups.Items.ItemImplementations.ItemGiantFork)
- **Constructor Address**: 0x180458280
- **Category**: Damage/Critical Strike Enhancement
- **Rarity**: Standard Item

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| critChancePerAmount | float | 0.15 | 15% critical chance per stack |
| megaCritChancePerAmount | float | 0.14 | 14% mega critical chance per stack |
| megaCritDamageMultiplier | float | 4.0 | Base mega critical damage multiplier |
| extraDamageMultiplierPerAmount | float | 0.15 | Extra damage multiplier per additional stack |
| finalMegacritMultiplier | float | 4.0 | Initial value, recalculated on amount change |
| megaCritChance | float | Calculated | amount * megaCritChancePerAmount |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 18 | Critical Chance | amount * 0.15 | Linear per stack |

## Special Mechanics

### Mega Critical System
The GiantFork implements a unique "mega critical" system that builds upon the standard critical strike mechanics:

1. **Standard Critical Enhancement**: Each stack provides 15% critical strike chance through the standard EStat system
2. **Mega Critical Proc**: When a standard critical hit occurs, there's an additional chance to trigger a "mega critical"
3. **Mega Critical Chance**: Calculated as `amount * 0.14` (14% per stack)
4. **Mega Critical Damage**: Base multiplier of 4.0x, plus 0.15x for each additional stack beyond the first

### Trigger Conditions
- Mega critical can only trigger on attacks that are already critical hits
- Uses the `TryProc` system with proc coefficient scaling
- Sets damage effect type to 2 (likely a special visual/audio effect)

## Formulas

### Critical Chance Calculation
```
standardCritChance = amount * 0.15
```

### Mega Critical Chance Calculation
```
megaCritChance = amount * 0.14
```

### Final Mega Crit Multiplier Calculation
```
if (amount <= 1) {
    finalMegacritMultiplier = megaCritDamageMultiplier  // 4.0
} else {
    finalMegacritMultiplier = megaCritDamageMultiplier + (amount - 1) * extraDamageMultiplierPerAmount
    // = 4.0 + (amount - 1) * 0.15
}
```

### Mega Critical Damage Application
```
if (isCritical && TryProc(megaCritChance, procCoefficient)) {
    damageEffect = 2  // Special effect type
    itemAttackModifier.AddMultiplier(finalMegacritMultiplier)
}
```

## Implementation Details
- **Update Frequency**: On initialization and amount changes only
- **Event Subscriptions**:
  - `OnInitOrAmountChanged()` - Updates stat modifiers and mega crit chance
  - `PreAttack()` - Processes mega critical chance before damage calculation
- **Stack Behavior**: Linear scaling for both standard and mega critical chances

## C# Pseudocode
```csharp
// Constructor logic
public ItemGiantFork(ItemInventory itemInventoryRef) : base(itemInventoryRef)
{
    critChancePerAmount = 0.15f;
    megaCritChancePerAmount = 0.14f;
    megaCritDamageMultiplier = 4.0f;
    extraDamageMultiplierPerAmount = 0.15f;
    finalMegacritMultiplier = 4.0f;
}

// Initialization and stack update logic
protected override void OnInitOrAmountChanged()
{
    // Calculate current mega crit chance
    megaCritChance = amount * megaCritChancePerAmount;

    // Set standard critical chance stat modifier
    var critModifier = new StatModifier()
    {
        type = 2, // Additive
        statType = EStat.CriticalChance, // 18
        value = amount * critChancePerAmount
    };
    SetStat(critModifier);

    // Calculate final mega crit multiplier
    finalMegacritMultiplier = megaCritDamageMultiplier;
    if (amount > 1)
    {
        finalMegacritMultiplier = megaCritDamageMultiplier + (amount - 1) * extraDamageMultiplierPerAmount;
    }
}

// Pre-attack processing
public override void PreAttack(DamageContainer dc, StatComponents itemAttackModifier)
{
    // Only process if the attack is already a critical hit and has enemy target
    if (dc.crit && dc.enemy != null && TryProc(megaCritChance, dc.procCoefficient))
    {
        dc.damageEffect = 2; // Special mega crit effect
        itemAttackModifier.AddMultiplier(finalMegacritMultiplier);
    }
}
```

## Technical Notes
- The mega critical system is separate from the standard critical system, allowing for multiplicative scaling
- Proc coefficient scaling ensures the mega critical chance is balanced with attack speed and multi-hit abilities
- The damage effect type 2 likely triggers special visual/audio feedback for mega crits
- Both critical systems stack additively with additional stacks of the item

## Related Items
- **DemonBlade**: Also provides critical chance (1% flat) with healing on crit
- **GrandmasSecretTonic**: Provides critical chance (2% flat) with area damage on crit
- Items that enhance critical damage or proc coefficient would synergize well

---

*Data extracted from decompiled IL2CPP constructors and C# class definitions*