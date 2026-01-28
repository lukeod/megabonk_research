# SluttyCannon

## Overview
- **Item ID**: EItem.SluttyCannon (29)
- **Constructor Address**: 0x180464800
- **Category**: Projectile/Proc-based Damage
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| procChancePerAmount | float | 0.2 | 20% base proc chance per stack |
| damageRatio | float | 1.0 | Base damage multiplier |
| damageRatioPerAmount | float | 0.4 | 40% damage increase per stack |
| damageSource | string | "SluttyCannon" | Damage source identifier |
| maxProcsPerTick | int | 4 | Maximum rocket spawns per tick |
| numProcsThisTick | int | 0 | Current tick proc counter (reset each tick) |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | No direct stat modifications | - | - |

## Special Mechanics

### Rocket Spawning System
- Spawns rocket projectiles on hit when proc chance succeeds
- Uses hyperbolic scaling to prevent 100% proc rates
- Rockets are spawned from object pool for performance
- Maximum of 4 rockets per game tick to prevent lag

### Proc Chance Calculation
```
procChance = HyperbolicScaling(amount * 0.2, 1.0, 0.5)
if (procChance > 0.6) procChance = 0.6  // Hard cap at 60%
```

### Damage Calculation
```
rocketDamage = originalDamage * damageRatio
damageRatio = (amount * 0.4) + 1.0
```

## Formulas

### Hyperbolic Scaling Formula
The proc chance uses hyperbolic scaling with parameters:
- Base value: `amount * 0.2`
- Scale factor: `1.0`
- Diminishing factor: `0.5`
- Hard cap: `60%` (0.6)

### Damage Scaling
- **Base damage**: 100% of original attack damage
- **Per stack**: +40% damage multiplier
- **Final formula**: `damage * (1.0 + amount * 0.4)`

### Rocket Position Calculation
Rockets spawn at player position plus a small random offset using `UnityEngine.Vector3.up`

## Implementation Details
- **Update Frequency**: Every game tick (Tick method resets proc counter)
- **Event Subscriptions**: ProcOnHitEffects (triggered on successful hits)
- **Stack Behavior**: Linear damage scaling, hyperbolic proc chance scaling
- **Performance**: Uses object pooling for rocket instances
- **Proc Limit**: 4 rockets maximum per tick to prevent performance issues

## C# Pseudocode
```csharp
// Simplified constructor logic
public ItemSluttyCannon(ItemInventory itemInventoryRef) {
    procChancePerAmount = 0.2f;
    damageRatio = 1.0f;
    damageRatioPerAmount = 0.4f;
    damageSource = "SluttyCannon";
    maxProcsPerTick = 4;

    base(itemInventoryRef);
}

// On amount changed or initialization
protected override void OnInitOrAmountChanged() {
    // Calculate proc chance with hyperbolic scaling
    procChance = StatScaling.HyperbolicScaling(
        amount * procChancePerAmount, 1.0f, 0.5f);

    // Hard cap at 60%
    if (procChance > 0.6f)
        procChance = 0.6f;

    // Linear damage scaling
    damageRatio = (amount * damageRatioPerAmount) + 1.0f;
}

// Reset proc counter each tick
public override void Tick() {
    numProcsThisTick = 0;
}

// Main proc logic
public override void ProcOnHitEffects(DamageContainer dc) {
    if (numProcsThisTick >= maxProcsPerTick) return;
    if (dc == null) return;

    if (ItemUtility.TryProc(dc.procCoefficient, procChance)) {
        numProcsThisTick++;

        // Get rocket from object pool
        var rocketObj = PoolManager.Instance.rocketPool.Get();
        if (rocketObj != null) {
            var rocket = rocketObj.GetComponent<Rocket>();
            if (rocket != null) {
                var playerPos = MyPlayer.Instance.transform.position;
                var spawnPos = playerPos + Vector3.up; // Small offset
                var rocketDamage = dc.damage * damageRatio;

                rocket.Set(spawnPos, rocketDamage, 0.0f,
                          false, true, damageSource);
            }
        }
    }
}
```

## Technical Notes
- **Object Pooling**: Uses `PoolManager.Instance.rocketPool` for performance
- **Null Safety**: Extensive null checking in ProcOnHitEffects
- **Proc Coefficient**: Respects `DamageContainer.procCoefficient` for proc scaling
- **Memory Management**: Uses IL2CPP garbage collection with `GC_end_stubborn_change`
- **Thread Safety**: Tick-based reset ensures consistent behavior across frames

## Related Items
- **Other Rocket Items**: Items that spawn similar projectile types
- **Proc-based Items**: Items using similar hyperbolic scaling (Dragonfire, EagleClaw, IceCrystal, etc.)
- **Rate-Limited Items**: Items with maxProcsPerTick mechanics (Bonker, WeebHeadset)

## Scaling Analysis
| Stacks | Proc Chance | Damage Multiplier | Notes |
|--------|-------------|-------------------|-------|
| 1 | ~16.7% | 140% | Base effectiveness |
| 2 | ~26.7% | 180% | Good scaling |
| 3 | ~33.3% | 220% | Diminishing proc returns |
| 5 | ~41.7% | 300% | Strong damage, moderate proc |
| 10 | ~52.6% | 500% | Near cap proc chance |
| 15+ | 60% (capped) | 700%+ | Damage continues scaling |

---

*Data extracted from decompiled IL2CPP constructor at 0x180464800 and C# interop layer*