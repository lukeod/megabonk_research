# Bonker

## Overview
- **Item ID**: EItem.Bonker (ID: 3)
- **Constructor Address**: 0x18043BD60
- **Category**: Area of Effect / Proc-based Damage
- **Rarity**: Unknown (determinable from game data)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| baseChance | float | 0.02 | 2% base proc chance |
| baseDamageMultiplier | float | 20.0 | Base damage multiplier |
| chancePerStack | float | 0.015 | 1.5% additional chance per stack |
| damageMultiplierPerStack | float | 10.0 | Additional damage per stack |
| radiusPerStack | float | 1.0 | Radius increase per stack |
| radius | float | 3.5 | Base radius (initial value) |
| maxRadius | float | 10.0 | Maximum radius cap |
| maxProcsPerTick | int | 5 | Maximum procs per tick/frame |
| damageSource | string | "Bonker" | Damage source identifier |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | No direct stat modifiers | N/A | N/A |

## Special Mechanics
- **Area Damage**: Creates explosive area damage on hit proc
- **Knockback Multiplier**: Applies 1.25x knockback to area damage
- **Proc Limitation**: Maximum 5 procs per game tick to prevent performance issues
- **Dynamic Radius**: Final radius = min((amount * radiusPerStack) + 6.0, maxRadius)
- **Chain Reactions**: Can damage multiple enemies in radius after initial hit

## Formulas

### Proc Chance Calculation
```
if (amount <= 0):
    chance = baseChance (0.02)
else:
    chance = baseChance + (amount - 1) * chancePerStack
    chance = 0.02 + (amount - 1) * 0.015
```

### Damage Calculation
```
if (amount <= 0):
    damageMultiplier = baseDamageMultiplier (20.0)
else:
    damageMultiplier = baseDamageMultiplier + (amount - 1) * damageMultiplierPerStack
    damageMultiplier = 20.0 + (amount - 1) * 10.0

finalDamage = originalDamage * damageMultiplier
```

### Radius Calculation
```
finalRadius = (amount * radiusPerStack) + 6.0
finalRadius = (amount * 1.0) + 6.0
if (finalRadius > maxRadius):
    finalRadius = maxRadius  // maxRadius = 10.0
```

### Knockback Calculation
```
areaKnockback = originalKnockback * 1.25
```

## Implementation Details
- **Update Frequency**: Triggered on hit events (ProcOnHitEffects)
- **Event Subscriptions**: Responds to player attack hit events
- **Stack Behavior**: Linear scaling for all properties (chance, damage, radius)
- **Proc System**: Uses ItemUtility.TryProc with damage container's proc coefficient
- **Visual Effects**: Spawns explosion effect from object pool at enemy position

## C# Pseudocode
```csharp
// Simplified constructor logic
public ItemBonker(ItemInventory itemInventoryRef) : base(itemInventoryRef)
{
    baseChance = 0.02f;
    baseDamageMultiplier = 20.0f;
    chancePerStack = 0.015f;
    damageMultiplierPerStack = 10.0f;
    radiusPerStack = 1.0f;
    radius = 3.5f;
    maxRadius = 10.0f;
    maxProcsPerTick = 5;
    damageSource = "Bonker";

    // Create reusable damage container
    reuseDc = new DamageContainer(0.0f, damageSource);
}

// On amount change
protected override void OnInitOrAmountChanged()
{
    if (amount <= 0)
    {
        chance = baseChance;
        damageMultiplier = baseDamageMultiplier;
    }
    else
    {
        chance = baseChance + (amount - 1) * chancePerStack;
        damageMultiplier = baseDamageMultiplier + (amount - 1) * damageMultiplierPerStack;
    }

    radius = (amount * radiusPerStack) + 6.0f;
    if (radius > maxRadius)
        radius = maxRadius;
}

// Proc on hit
public override void ProcOnHitEffects(DamageContainer dc)
{
    if (dc == null) return;

    if (!ItemUtility.TryProc(dc.procCoefficient, chance)) return;

    numProcsThisTick++;

    // Spawn explosion effect at enemy position
    var explosion = PoolManager.Instance.explosionPool.Get();
    if (explosion != null)
    {
        var enemyPos = dc.enemy.GetCenterPosition();
        enemyPos.y += dc.enemy.GetMinSpacing() * 0.5f;
        explosion.transform.position = enemyPos;
    }

    // Damage initial target
    reuseDc.damage = damageMultiplier * dc.damage;
    reuseDc.enemy = dc.enemy;
    reuseDc.damageEffect = 1; // Explosion effect
    reuseDc.direction = dc.direction;
    dc.enemy.DamageFromPlayerOther(reuseDc);

    if (numProcsThisTick >= maxProcsPerTick) return;

    // Find and damage nearby enemies
    var nearbyEnemies = ItemUtility.GetNearbyEnemies(
        dc.enemy.GetCenterPosition(),
        radius,
        dc.enemy
    );

    foreach (var enemy in nearbyEnemies)
    {
        reuseDc.Reuse(0.0f, damageSource);
        reuseDc.damage = dc.damage;
        reuseDc.enemy = enemy;
        reuseDc.knockback = dc.knockback * 1.25f; // Increased knockback
        reuseDc.direction = (enemy.GetCenterPosition() - MyPlayer.Instance.transform.position).normalized;

        enemy.DamageFromPlayerOther(reuseDc);
    }
}
```

## Technical Notes
- **Performance Optimization**: Uses object pooling for explosion effects
- **Proc Limiting**: `maxProcsPerTick` prevents cascading explosions from causing frame drops
- **Memory Management**: Reuses DamageContainer instance to reduce garbage collection
- **Positioning**: Explosions spawn slightly above enemy center for visual clarity
- **Direction Calculation**: Area damage uses player-to-enemy direction for knockback

## Related Items
- **SpicyMeatball**: Similar chain explosion mechanics with different proc conditions
- **GlovesBlood**: Area damage on hit with additional healing component
- **QuinsMask**: Thorns-based area damage that spreads damage on taking hits
- **ToxicBarrel**: Area effect triggered by player damage instead of dealing damage

---

*Data extracted from decompiled IL2CPP constructors via IDA Pro MCP and C# decompilation*