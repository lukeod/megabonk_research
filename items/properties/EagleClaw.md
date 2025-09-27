# EagleClaw

## Overview
- **Item ID**: EItem.EagleClaw
- **Constructor Address**: 0x180408BC0
- **Category**: Damage/Control - Airborne Specialist
- **Rarity**: Unknown (not available in constructor data)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| procChancePerAmount | float | 0.08 | 8% base proc chance per stack |
| damageAddition | float | 0.66 | Base damage addition (calculated) |
| damageAdditionPerAmount | float | 0.66 | 66% damage per stack |
| knockupForce | float | 3.5 | Base knockup force |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | None | N/A | No direct stat modifications |

## Special Mechanics

### Proc System
- **Proc Chance Formula**: `(amount * 0.08) / ((amount * 0.08) + 0.5)`
- Uses hyperbolic scaling to prevent 100% proc rates
- Maximum theoretical proc chance approaches 100% but never reaches it

### Damage Scaling
- **Damage Bonus**: Only applies when enemy is airborne (not grounded)
- **Damage Formula**: `amount * 0.66` additive damage
- Checked in `PreAttack` method before damage calculation

### Knockup Mechanics
- **Knockup Force**: Base 3.5, scaled by player's knockback stat (EStat 24)
- **Final Knockup**: `knockupForce * PlayerStats.GetStat(24)`
- **Knockback**: Also applies horizontal knockback in damage direction
- Only triggers when enemy is grounded and proc succeeds

## Formulas

### Proc Chance Calculation
```
procChance = (stacks * 0.08) / ((stacks * 0.08) + 0.5)
```

### Damage Addition
```
damageBonus = stacks * 0.66  // Only when enemy is airborne
```

### Knockup Force
```
finalKnockup = 3.5 * PlayerStats.GetStat(24)  // EStat 24 = Knockback multiplier
```

## Implementation Details
- **Update Frequency**: Only recalculates on stack amount changes
- **Event Subscriptions**: Hooks into attack pipeline via `PreAttack` and `ProcOnHitEffects`
- **Stack Behavior**: Linear scaling for both proc chance (with hyperbolic cap) and damage

## C# Pseudocode
```csharp
// Simplified constructor logic
public ItemEagleClaw(ItemInventory itemInventoryRef)
{
    this.procChancePerAmount = 0.08f;
    this.damageAddition = 0.66f;
    this.damageAdditionPerAmount = 0.66f;
    this.knockupForce = 3.5f;
}

protected override void OnInitOrAmountChanged()
{
    float amount = (float)this.amount;
    // Hyperbolic scaling for proc chance
    this.procChance = (amount * procChancePerAmount) / ((amount * procChancePerAmount) + 0.5f);
    this.damageAddition = amount * damageAdditionPerAmount;
}

public override void PreAttack(DamageContainer dc, StatComponents itemAttackModifier)
{
    if (dc?.enemy?.enemyMovement != null)
    {
        // Only add damage if enemy is airborne (not grounded)
        if (!dc.enemy.enemyMovement.grounded)
        {
            itemAttackModifier?.AddAdditive(damageAddition);
        }
    }
}

public override void ProcOnHitEffects(DamageContainer dc)
{
    if (dc?.enemy?.enemyMovement != null && dc.enemy.enemyMovement.grounded)
    {
        if (ItemUtility.TryProc(dc.procCoefficient, procChance))
        {
            float knockbackStat = PlayerStats.GetStat(24); // Knockback multiplier
            float finalKnockup = knockupForce * knockbackStat;

            // Apply knockup
            dc.enemy.enemyMovement.KnockUp(finalKnockup);

            // Apply horizontal knockback in damage direction
            dc.enemy.enemyMovement.Knockback(dc.direction, finalKnockup);
        }
    }
}
```

## Technical Notes
- **Grounded Check**: Item specifically checks if enemy is grounded before applying knockup
- **Airborne Check**: Damage bonus only applies to airborne enemies, creating a synergistic loop
- **Proc Coefficient**: Respects attack's proc coefficient for balanced multi-hit scenarios
- **Knockback Stat**: Uses EStat 24 (knockback multiplier) to scale both knockup and knockback forces

## Related Items
- **Bonker**: Similar area knockback mechanics
- **GlovesPower**: Also uses massive knockback forces
- **SpicyMeatball**: Chain reaction mechanics that could benefit from airborne enemies
- **TacticalGlasses**: Damage bonus items with conditional triggers

---

*Generated from decompiled IL2CPP constructor at 0x180408BC0 and C# interface analysis*