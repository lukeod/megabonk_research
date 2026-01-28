# CursedDoll

## Overview
- **Item ID**: EItem.CursedDoll (43)
- **Constructor Address**: 0x180440A10
- **Category**: Utility/Special
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| damageMaxHpPercentage | float | 0.3 | 30% of enemy max HP |
| enemiesCursedPerDoll | int | 2 | Cursed enemies per stack |
| maxNumCursesPerCheck | int | 5 | Max enemies to curse per tick |
| attackCooldown | float | 1.0 | Seconds between attacks |
| maxNumCursedEnemies | int | Runtime | Calculated: amount * enemiesCursedPerDoll |
| reuseDc | DamageContainer | Created | Reusable damage container (initial damage 1.0) |
| damageSource | string | "CursedDoll" | Damage attribution (EItem enum ID 43) |
| cursedEnemies | HashSet<Enemy> | New | Set of currently cursed enemies |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | None | N/A | No direct stat modifications |

## Special Mechanics

### Curse System
- **Maximum Cursed Enemies**: amount * enemiesCursedPerDoll (2 per stack)
- **Max Curses Per Check**: 5 enemies can be cursed per tick cycle
- **Target Selection**: Iterates through enemies from EnemyManager
- **Exclusion Logic**: Only targets enemies not already cursed, not dead, and not teleporting
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
maxNumCursedEnemies = amount * enemiesCursedPerDoll
// With enemiesCursedPerDoll = 2:
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
    damageMaxHpPercentage = 0.3f;
    enemiesCursedPerDoll = 2;
    maxNumCursesPerCheck = 5;
    attackCooldown = 1.0f;

    // Create reusable damage container with EItem enum string
    string itemName = EItem.CursedDoll.ToString(); // enum value 43
    reuseDc = new DamageContainer(1.0f, itemName);
    damageSource = itemName;
    cursedEnemies = new HashSet<Enemy>();
}

// Amount change handler
protected override void OnInitOrAmountChanged()
{
    maxNumCursedEnemies = amount * enemiesCursedPerDoll;
}

// Main update logic
public override void Tick()
{
    if (MyTime.time < nextAttackTime) return;

    nextAttackTime = MyTime.time + attackCooldown;

    // Add new cursed enemies if under limit
    if (cursedEnemies.Count < maxNumCursedEnemies)
    {
        int cursedThisTick = 0;
        foreach (var enemy in EnemyManager.Instance.enemies.Values)
        {
            if (enemy == null) continue;
            if (enemy.IsDead() || enemy.IsTeleporting()) continue;
            if (cursedEnemies.Contains(enemy)) continue;

            cursedEnemies.Add(enemy);
            cursedThisTick++;

            if (cursedThisTick >= maxNumCursesPerCheck) break;
            if (cursedEnemies.Count >= maxNumCursedEnemies) break;
        }
    }

    // Damage all cursed enemies
    foreach (var cursedEnemy in cursedEnemies)
    {
        if (cursedEnemy == null || cursedEnemy.IsDead()) continue;

        // Spawn visual effect
        var hitEffect = PoolManager.Instance.cursedHitPool.Get();
        if (hitEffect != null)
        {
            hitEffect.transform.position = cursedEnemy.GetHeadPosition();
        }

        // Calculate damage
        float damage;
        if (cursedEnemy.IsBoss())
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

    // Clean up dead enemies
    cursedEnemies.RemoveWhere(enemy => enemy == null || enemy.IsDead());
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
*Generated from IL2CPP decompiled constructor at 0x180440A10 and C# interop definitions*