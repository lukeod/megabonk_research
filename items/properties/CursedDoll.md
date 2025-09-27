# CursedDoll

## Overview
- **Item ID**: EItem.CursedDoll (43)
- **Constructor Address**: 0x180406CA0
- **Category**: Utility/Special
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| maxNumCursedEnemies | int | 1 | Initial maximum cursed enemies |
| damageMaxHpPercentage | float | 0.3 | 30% of enemy max HP |
| amountPerDoll | int | 2 | Cursed enemies per stack |
| attackCooldown | float | 1.0 | Seconds between attacks |
| nextAttackTime | float | 0.0 | Time tracking for next attack |
| reuseDc | DamageContainer | Created | Reusable damage container |
| damageSource | string | "CursedDoll" | Damage attribution |
| cursedEnemies | HashSet<Enemy> | New | Set of currently cursed enemies |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | None | N/A | No direct stat modifications |

## Special Mechanics

### Curse System
- **Maximum Cursed Enemies**: amount * 2 (starts at 2 with 1 stack)
- **Target Selection**: Randomly selects alive enemies from EnemyManager
- **Exclusion Logic**: Only targets enemies not already cursed and not dead
- **Persistence**: Cursed enemies remain in set until they die or are cleaned up

### Damage Calculation
- **Regular Enemies**: 30% of enemy's maximum HP
- **Boss Enemies**: 70% of player's base damage (30% reduction from normal)
- **Damage Type**: 7 (Special damage effect)
- **Source Attribution**: Tagged as "CursedDoll" damage

### Attack Pattern
- **Update Frequency**: Every game tick (Tick() method)
- **Cooldown**: 1.0 second between attack cycles
- **Target Processing**: Iterates through all cursed enemies each attack cycle
- **Visual Effects**: Spawns cursed hit pool effects at enemy head position

## Formulas

### Maximum Cursed Enemies
```
maxNumCursedEnemies = amount * amountPerDoll
// With amountPerDoll = 2:
maxNumCursedEnemies = amount * 2
```

### Damage Calculation
```csharp
// For regular enemies:
damage = enemy.maxHp * damageMaxHpPercentage
damage = enemy.maxHp * 0.3

// For boss enemies:
damage = player.baseDamage * 0.7
```

### Attack Timing
```csharp
nextAttackTime = currentTime + attackCooldown
// attackCooldown = 1.0 seconds
```

## Implementation Details
- **Update Frequency**: Every frame via Tick() method
- **Event Subscriptions**: Enemy.A_EnemyReleasedFromPool (for cleanup)
- **Stack Behavior**: Linear scaling - each stack adds 2 more possible cursed enemies
- **Memory Management**: Uses HashSet for O(1) lookup and automatic deduplication

## C# Pseudocode
```csharp
// Simplified constructor logic
public ItemCursedDoll(ItemInventory itemInventoryRef) : base(itemInventoryRef)
{
    maxNumCursedEnemies = 1;
    damageMaxHpPercentage = 0.3f;
    amountPerDoll = 2;
    attackCooldown = 1.0f;

    // Create reusable damage container
    reuseDc = new DamageContainer(1.0f, "CursedDoll");
    damageSource = "CursedDoll";
    cursedEnemies = new HashSet<Enemy>();
}

// Amount change handler
protected override void OnInitOrAmountChanged()
{
    maxNumCursedEnemies = amount * amountPerDoll;
}

// Main update logic
public override void Tick()
{
    if (Time.time < nextAttackTime) return;

    nextAttackTime = Time.time + attackCooldown;

    // Add new cursed enemies if under limit
    if (cursedEnemies.Count < maxNumCursedEnemies)
    {
        foreach (var enemy in EnemyManager.Instance.enemies.Values)
        {
            if (enemy != null && !enemy.IsDead && !cursedEnemies.Contains(enemy))
            {
                cursedEnemies.Add(enemy);
                break; // Add one per cycle
            }
        }
    }

    // Damage all cursed enemies
    foreach (var cursedEnemy in cursedEnemies)
    {
        if (cursedEnemy != null && !cursedEnemy.IsDead)
        {
            // Spawn visual effect
            var hitEffect = PoolManager.Instance.cursedHitPool.Get();
            if (hitEffect != null)
            {
                hitEffect.transform.position = cursedEnemy.GetHeadPosition();
            }

            // Calculate damage
            float damage;
            if (cursedEnemy.IsBoss)
            {
                damage = MyPlayer.Instance.baseDamage * 0.7f;
            }
            else
            {
                damage = cursedEnemy.maxHp * damageMaxHpPercentage;
            }

            // Apply damage
            reuseDc.damage = damage;
            reuseDc.enemy = cursedEnemy;
            reuseDc.damageEffect = 7;
            cursedEnemy.DamageFromPlayerOther(reuseDc);
        }
    }

    // Clean up dead enemies
    cursedEnemies.RemoveWhere(enemy => enemy == null || enemy.IsDead);
}
```

## Technical Notes
- **Performance**: Uses HashSet for efficient enemy tracking and deduplication
- **Visual Effects**: Integrates with Unity's object pooling system for hit effects
- **Boss Handling**: Special damage calculation for boss enemies to prevent trivial kills
- **Event Cleanup**: Properly subscribes/unsubscribes from enemy death events
- **Thread Safety**: All operations are single-threaded within Unity's main thread

## Related Items
- **GlovesBlood**: Similar HP-percentage based damage mechanics
- **DemonicBlood/DemonicSoul**: Other items that track enemy interactions
- **Ghost**: Another item that spawns effects on enemies

---
*Generated from IL2CPP decompiled constructor at 0x180406CA0 and C# interop definitions*