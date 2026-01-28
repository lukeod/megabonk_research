# Snek

## Overview
- **Item ID**: EItem.Snek (83)
- **Constructor Address**: 0x180465430
- **Category**: Poison/Debuff-based Damage
- **Rarity**: Unknown
- **Base Class**: ItemBase (inherit standard item behavior)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| poisonChancePerAmount | float | 4.0 | 400% poison chance per stack (guarantees stacks) |
| poisonBurstChancePerAmount | float | 0.05 | 5% burst chance per stack |
| damageRatio | float | 0.4 | Base 40% damage multiplier (modified on init) |
| damageRatioPerAmount | float | 0.4 | 40% damage increase per stack |
| damageSource | string | "Snek" | Damage source identifier (from EItem.Snek = 83) |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | No direct stat modifications | - | - |

## Special Mechanics

### Poison Application System
- Applies poison debuff (EDebuff = 1) on hit
- Poison chance is calculated as guaranteed stacks + fractional chance
- Multiple poison stacks can be applied per hit

### Poison Burst System
- Has a chance to trigger a "burst" effect that deals instant damage
- Burst chance scales with stacks: `amount * 0.05`
- On burst, queues enemy for explosion processing in Tick()

### Tick Processing
- Iterates through `queuedExplosionEnemies` HashSet each tick
- For enemies with active poison debuff:
  - Calculates total poison stacks (including pending debuffs)
  - Computes poison damage using `DebuffPoison.GetPoisonDamagePerTick(stacks)`
  - Removes poison debuff from enemy
  - Deals burst damage with damageEffect = 3

### Visual Effect Spawning
- On poison burst, spawns a visual effect from object pool
- Effect positioned at closest point on enemy bounds to player
- Position offset calculated using `Vector3.up * 3.0` plus normalized direction

## Formulas

### Poison Stack Calculation
```
guaranteedStacks = floor(poisonChanceTotal / poisonChancePerAmount)
// Actually uses: guaranteedStacks = floor(amount * 4.0)
fractionalChance = poisonChanceTotal - guaranteedStacks
if (TryProc(procCoefficient, fractionalChance))
    stacks++
```

### On Init/Amount Changed
```
poisonChanceTotal = amount * poisonChancePerAmount  // amount * 4.0
poisonBurstChanceTotal = amount * poisonBurstChancePerAmount  // amount * 0.05
damageRatio = (amount * damageRatioPerAmount) + 0.5  // (amount * 0.4) + 0.5
```

### Burst Damage Calculation
```
burstDamage = originalDamage * damageRatio
damageRatio = (amount * 0.4) + 0.5
```

### Poison Burst Tick Damage
```
tickDamage = DebuffPoison.GetPoisonDamagePerTick(totalStacks) * remainingDuration
```

## Implementation Details
- **Update Frequency**: Tick method processes queued explosion enemies
- **Event Subscriptions**: ProcOnHitEffects (triggered on successful hits)
- **Stack Behavior**: Linear scaling for all properties
- **Debuff Type**: EDebuff = 1 (Poison)
- **Debuff Duration**: 5.0 seconds per application
- **Damage Effect**: 3 (used for burst damage)
- **Performance**: Uses HashSet for queued enemies, reuses DamageContainer

## C# Pseudocode
```csharp
public class ItemSnek : ItemBase
{
    public float poisonChancePerAmount = 4.0f;
    public float poisonBurstChancePerAmount = 0.05f;
    public float damageRatio = 0.4f;
    public float damageRatioPerAmount = 0.4f;
    public string damageSource = "Snek";

    public float poisonChanceTotal;
    public float poisonBurstChanceTotal;

    private HashSet<Enemy> queuedExplosionEnemies;
    private DamageContainer reuseDc;

    public ItemSnek(ItemInventory itemInventoryRef)
    {
        queuedExplosionEnemies = new HashSet<Enemy>();
        damageSource = EItem.Snek.ToString();  // ID 83
        reuseDc = new DamageContainer(0f, damageSource);
        base(itemInventoryRef);
    }

    protected override void OnInitOrAmountChanged()
    {
        poisonChanceTotal = amount * poisonChancePerAmount;
        poisonBurstChanceTotal = amount * poisonBurstChancePerAmount;
        damageRatio = (amount * damageRatioPerAmount) + 0.5f;
    }

    public override void Tick()
    {
        if (queuedExplosionEnemies.Count <= 0) return;

        foreach (var enemy in queuedExplosionEnemies)
        {
            if (enemy == null || enemy.IsDeadOrDyingNextFrame()) continue;
            if (!enemy.HasDebuff(EDebuff.Poison)) continue;

            int stacks = enemy.debuffs[EDebuff.Poison].GetStacks();

            // Include pending debuff stacks
            if (enemy.debuffsToAdd.ContainsKey(EDebuff.Poison))
            {
                stacks += enemy.debuffsToAdd[EDebuff.Poison].stacks;
                enemy.debuffsToAdd.Remove(EDebuff.Poison);
            }

            float poisonDamagePerTick = DebuffPoison.GetPoisonDamagePerTick(stacks);
            float remainingDuration = enemy.debuffs[EDebuff.Poison].remainingDuration;

            enemy.RemoveDebuff(EDebuff.Poison);

            reuseDc.Reuse(0f, damageSource);
            reuseDc.damage = remainingDuration * poisonDamagePerTick;
            reuseDc.damageEffect = 3;

            enemy.DamageFromPlayerOther(reuseDc);
        }

        queuedExplosionEnemies.Clear();
    }

    public override void ProcOnHitEffects(DamageContainer dc)
    {
        if (poisonChanceTotal <= 0f) return;

        int stacks = 0;
        if (poisonChancePerAmount > 0f)
            stacks = (int)Math.Floor(poisonChanceTotal);

        float fractionalChance = poisonChanceTotal - stacks;
        if (fractionalChance > 0f && ItemUtility.TryProc(dc.procCoefficient, fractionalChance))
            stacks++;

        if (stacks <= 0) return;

        // Apply poison debuff
        dc.enemy.AddDebuff(EDebuff.Poison, dc, 5.0f, stacks);

        // Check for poison burst
        if (ItemUtility.TryProc(dc.procCoefficient, poisonBurstChanceTotal))
        {
            reuseDc.Reuse(0f, damageSource);
            reuseDc.damage = damageRatio * dc.damage;
            reuseDc.enemy = dc.enemy;

            dc.enemy.DamageFromPlayerOther(reuseDc);
            queuedExplosionEnemies.Add(dc.enemy);

            // Spawn visual effect from pool
            var effectObj = PoolManager.Instance.effectPool.Get();
            if (effectObj != null)
            {
                // Position at enemy with offset toward player
                var playerPos = MyPlayer.Instance.transform.position;
                var closestPoint = dc.enemy.collider.ClosestPointOnBounds(playerPos);
                effectObj.transform.position = closestPoint;

                // Apply additional offset (Vector3.up * 3 + direction * 0.75)
                var currentPos = effectObj.transform.position;
                var direction = (playerPos - dc.enemy.transform.position).XZNormalized();
                effectObj.transform.position = currentPos + Vector3.up * 3f + direction * 0.75f;
            }
        }
    }
}
```

## Technical Notes
- **Object Pooling**: Uses PoolManager for visual effect instances
- **Reusable DamageContainer**: Creates a reusable DC in constructor for performance
- **HashSet for Queuing**: Uses HashSet<Enemy> to prevent duplicate explosion processing
- **Proc Coefficient**: Respects `DamageContainer.procCoefficient` for proc scaling
- **Guaranteed Stacks**: With 4.0 poison chance per amount, each stack guarantees 4 poison stacks
- **Two-Phase Damage**: Initial burst damage on proc + tick damage from poison explosion

## Related Items
- **Other Poison Items**: Items that apply or interact with poison debuff
- **Proc-based Items**: Items using similar proc chance mechanics
- **Debuff Items**: Items that apply status effects to enemies

## Scaling Analysis
| Stacks | Poison Stacks/Hit | Burst Chance | Damage Multiplier | Notes |
|--------|-------------------|--------------|-------------------|-------|
| 1 | 4 | 5% | 90% | Base effectiveness |
| 2 | 8 | 10% | 130% | Double poison stacks |
| 3 | 12 | 15% | 170% | High poison damage |
| 5 | 20 | 25% | 250% | Very strong poison |
| 10 | 40 | 50% | 450% | Massive poison stacking |

---

*Data extracted from decompiled IL2CPP constructor at 0x180465430 and related methods*
