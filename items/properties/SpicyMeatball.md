# SpicyMeatball

## Overview
- **Item ID**: EItem.SpicyMeatball (ID: 15)
- **Constructor Address**: 0x180425B80
- **Category**: Area Damage / Chain Explosion
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| baseRadius | float | 3.0 | Base explosion radius |
| radiusPerAmount | float | 1.0 | Radius increase per stack |
| maxRadius | float | 8.0 | Maximum explosion radius |
| damageSpreadMultiplier | float | 0.65 | Damage scaling for chain explosions |
| procChance | float | 0.25 | 25% chance to trigger on hit |
| maxProcsPerTick | int | 50 | Maximum procs per game tick |
| numProcsThisTick | int | 0 | Current tick proc counter (resets each tick) |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | - | - | No direct stat modifications |

## Special Mechanics
- **Chain Explosions**: On successful proc (25% chance), creates an area explosion that damages nearby enemies
- **Area Scaling**: Explosion radius scales with EStat 9 (Area/Radius Multiplier)
- **Proc Limiting**: Limited to 50 procs per game tick to prevent performance issues
- **Damage Source**: Uses item-specific damage source string for tracking
- **Reusable Damage Container**: Uses a pre-allocated damage container for performance

## Formulas
- **Final Radius**: `min(max(1.0, EStat9 * (baseRadius + (amount * radiusPerAmount))), maxRadius)`
- **Chain Damage**: `original_damage * damageSpreadMultiplier` (65% of original damage)
- **Proc Chance**: Fixed 25% chance per hit (modified by proc coefficient)

## Implementation Details
- **Update Frequency**: Tick() called every game tick to reset proc counter
- **Event Subscriptions**: Responds to ProcOnHitEffects events
- **Stack Behavior**: Radius increases linearly with stack count, damage multiplier remains constant
- **Performance Optimization**: Uses object pooling for explosion effects

## C# Pseudocode
```csharp
// Constructor initialization
public ItemSpicyMeatball(ItemInventory itemInventoryRef) : base(itemInventoryRef)
{
    baseRadius = 3.0f;
    radiusPerAmount = 1.0f;
    maxRadius = 8.0f;
    damageSpreadMultiplier = 0.65f;
    procChance = 0.25f;
    damageSource = EItem.SpicyMeatball.ToString();
    reuseDc = new DamageContainer(0.0f, damageSource);
    maxProcsPerTick = 50;
}

// Radius calculation on init/amount change
protected override void OnInitOrAmountChanged()
{
    float calculatedRadius = PlayerStats.GetStat(EStat.AreaMultiplier) *
                           (baseRadius + (amount * radiusPerAmount));
    radius = Mathf.Clamp(calculatedRadius, 1.0f, maxRadius);
}

// Proc logic on hit
public override void ProcOnHitEffects(DamageContainer dc)
{
    if (numProcsThisTick >= maxProcsPerTick) return;

    if (ItemUtility.TryProc(procChance, dc.procCoefficient))
    {
        numProcsThisTick++;

        // Reuse damage container for performance
        reuseDc.Reuse(0.0f, damageSource);
        reuseDc.damage = damageSpreadMultiplier * dc.damage;
        reuseDc.enemy = dc.enemy;

        // Get explosion center and find nearby enemies
        Vector3 center = dc.enemy.GetCenterPosition();
        var nearbyEnemies = ItemUtility.GetNearbyEnemies(center, radius, dc.enemy);

        // Damage all enemies in radius
        foreach (var enemy in nearbyEnemies)
        {
            reuseDc.enemy = enemy;
            enemy.DamageFromPlayerOther(reuseDc);
        }

        // Spawn visual explosion effect
        SpawnExplosionEffect(center);
    }
}

// Reset proc counter each tick
public override void Tick()
{
    numProcsThisTick = 0;
}
```

## Technical Notes
- **IL2CPP Optimization**: Uses native field offsets for high-performance property access
- **Memory Management**: Reuses DamageContainer instances to reduce garbage collection
- **Visual Effects**: Spawns pooled GameObject explosion effects with random positioning
- **Collision Detection**: Uses sphere-based proximity detection for chain damage
- **Thread Safety**: Proc counter ensures consistent behavior in multi-threaded scenarios

## Related Items
- **Bonker**: Similar area damage mechanics with radius scaling
- **Dragonfire**: Another proc-based area effect item
- **GrandmasSecretTonic**: Shares crit-based area damage spreading
- **QuinsMask**: Area damage spreading on thorns proc

---

*Generated from decompiled IL2CPP constructor at 0x180425B80 and C# interface analysis*