# GrandmasSecretTonic

## Overview
- **Item ID**: EItem.GrandmasSecretTonic (ID: 11)
- **Constructor Address**: 0x18045C6A0
- **Category**: Area Damage/Critical Strike Enhancement
- **Rarity**: Not determinable from code

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| critChanceTotal | float | 0.02 | 2% critical chance |
| baseRadius | float | 3.0 | Base area radius |
| radiusPerAmount | float | 1.0 | Radius increase per stack |
| maxRadius | float | 8.0 | Maximum radius cap |
| damageSpreadMultiplier | float | 0.5 | 50% damage for spread effect |
| procChance | float | 0.5 | 50% chance to trigger on crit |
| maxProcsPerTick | int | 100 | Maximum procs per game tick |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 18 | Critical Chance | 0.02 | Static |

## Special Mechanics
- **Critical Spread Damage**: On critical hit, 50% chance to deal damage to nearby enemies
- **Area Calculation**: Radius = min(maxRadius, max(1.0, (baseRadius + amount * radiusPerAmount) * EStat9))
- **Damage Distribution**: Spread damage = original damage * damageSpreadMultiplier (50%)
- **Visual Effect**: Spawns particle effect at hit location with random offset
- **Proc Limiting**: Limited to 100 procs per tick to prevent performance issues

## Formulas
- **Final Radius**: `min(8.0, max(1.0, (3.0 + amount * 1.0) * Area_Multiplier_Stat))`
- **Spread Damage**: `original_damage * 0.5`
- **Proc Chance**: `50% * proc_coefficient` (from damage container)

## Implementation Details
- **Update Frequency**: Radius recalculated on OnInitOrAmountChanged
- **Event Subscriptions**: ProcOnHitEffects for critical hit detection
- **Stack Behavior**: Linear radius scaling, static crit chance
- **Performance**: Proc counter reset every Tick() call

## C# Pseudocode
```csharp
// Constructor logic
public ItemGrandmasSecretTonic(ItemInventory inventory) {
    critChanceTotal = 0.02f;
    baseRadius = 3.0f;
    radiusPerAmount = 1.0f;
    maxRadius = 8.0f;
    damageSpreadMultiplier = 0.5f;
    procChance = 0.5f;
    maxProcsPerTick = 100;

    // Create damage container for proc effects
    procDc = new DamageContainer(0.0f, "GrandmasSecretTonic");

    // Set critical chance stat
    SetStat(new StatModifier(EStat.CriticalChance, ModifierType.Additive, 0.02f));
}

// On critical hit proc
public override void ProcOnHitEffects(DamageContainer dc) {
    if (numProcsThisTick >= maxProcsPerTick) return;

    if (dc.crit && TryProc(procChance, dc.procCoefficient)) {
        numProcsThisTick++;
        Vector3 hitPosition = dc.enemy.GetCenterPosition();

        // Get nearby enemies in radius
        var nearbyEnemies = GetNearbyEnemies(hitPosition, radius, dc.enemy);

        foreach (var enemy in nearbyEnemies) {
            if (!enemy.IsDead()) {
                // Deal spread damage
                procDc.Reuse(dc.damage * damageSpreadMultiplier, damageSource);
                procDc.enemy = enemy;
                enemy.DamageFromPlayerOther(procDc);
            }
        }

        // Spawn visual effect at random offset from hit position
        SpawnParticleEffect(hitPosition + RandomOffset());
    }
}
```

## Technical Notes
- Uses Unity's object pooling system for particle effects
- Damage spread uses separate DamageContainer to prevent infinite recursion
- EStat 9 (Area/Radius Multiplier) affects final radius calculation
- Proc coefficient scaling allows for balanced interaction with other items
- Random particle position uses `XZVector(Random.insideUnitSphere * 0.5)` for offset

## Related Items
- **QuinsMask**: Similar area damage spread mechanics with thorns
- **SpicyMeatball**: Chain explosion area damage system
- **GiantFork**: Critical chance enhancement item
- **Bonker**: Area damage with radius scaling

---

*Data extracted from decompiled IL2CPP constructor at 0x18045C6A0 and C# implementation*