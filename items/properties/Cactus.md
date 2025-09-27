# Cactus

## Overview
- **Item ID**: EItem.Cactus
- **Constructor Address**: 0x180404130
- **Category**: Defensive/Retaliation
- **Rarity**: Common

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| damagePerAmount | float | 5.0 | Damage per stack |
| numProjectilesPerAmount | int | 2 | Projectiles per stack |
| damage | float | Calculated | amount * damagePerAmount |
| numProjectiles | int | Calculated | amount * numProjectilesPerAmount + 1 |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| No direct stat modifiers | - | - | - |

## Special Mechanics
- **Counter-Attack System**: Triggers when player takes damage
- **Projectile Generation**: Creates directional projectiles around the player
- **Thorns Damage Addition**: Adds player's thorns stat (EStat 3) to base damage
- **Multi-Target**: Hits all enemies within 15.0 unit range using sphere cast
- **Particle Effects**: Spawns visual effects with configurable projectile count

### Trigger Conditions
- Activates on `PlayerHealth.A_TakeDamage` event
- Does not distinguish between shield damage and health damage
- No cooldown or proc chance - triggers on every damage instance

### Targeting System
- Uses sphere cast with 0.5 radius and 15.0 range
- Casts rays in circular pattern around player
- Number of rays equals clamped `numProjectiles` value
- Ray directions calculated using quaternion rotation around player's up vector

## Formulas

### Damage Calculation
```
finalDamage = (amount * 5.0) + PlayerStats.GetStat(EStat.Thorns)
```

### Projectile Count
```
numProjectiles = (amount * 2) + 1
// Clamped between 2 and 50 for casting
// Clamped between 3 and 40 for particle effects
```

### Ray Direction Calculation
```
angleStep = 360.0 / clampedProjectiles
for i in 0 to clampedProjectiles:
    angle = i * angleStep
    direction = Quaternion.AngleAxis(angle, playerUpVector) * playerForwardVector
```

## Implementation Details
- **Update Frequency**: Event-driven (no polling)
- **Event Subscriptions**: PlayerHealth.A_TakeDamage
- **Stack Behavior**: Linear scaling for both damage and projectile count
- **Performance**: Uses object pooling for particle effects
- **Range Check**: 15.0 unit sphere cast from player position

### Damage Application
- Uses `WeaponUtility.GetDamageContainer` for consistent damage processing
- Applies damage via `Enemy.DamageFromPlayerOther`
- Includes knockback factor of 0.5
- Triggers `EffectManager.EnemyHitEffect` for visual feedback

### Particle System Integration
- Retrieves particle system from pooled GameObject
- Configures emission burst count based on projectile count
- Positions at player location with player rotation
- Particle count clamped to prevent performance issues

## C# Pseudocode
```csharp
// Constructor logic
public ItemCactus(ItemInventory itemInventoryRef) : base(itemInventoryRef)
{
    damagePerAmount = 5.0f;
    numProjectilesPerAmount = 2;
}

// Amount change handler
protected override void OnInitOrAmountChanged()
{
    numProjectiles = amount * numProjectilesPerAmount + 1;
    damage = amount * damagePerAmount;
}

// Main trigger logic
private void OnTakeDamage(PlayerHealth ph, DamageContainer dc, bool isShieldDamage)
{
    float finalDamage = damage + PlayerStats.GetStat(EStat.Thorns);
    int clampedProjectiles = Mathf.Clamp(numProjectiles, 2, 50);

    // Spawn particle effect
    GameObject particleObj = PoolManager.Instance.GetCactusParticle();
    ConfigureParticleSystem(particleObj, clampedProjectiles);

    // Generate ray directions
    List<Vector3> rayDirections = new List<Vector3>();
    Vector3 playerForward = player.transform.forward;
    Vector3 playerUp = player.transform.up;

    for (int i = 0; i < clampedProjectiles; i++)
    {
        float angle = i * (360.0f / clampedProjectiles);
        Quaternion rotation = Quaternion.AngleAxis(angle, playerUp);
        Vector3 direction = rotation * playerForward;
        rayDirections.Add(direction);
    }

    // Cast rays and damage enemies
    foreach (Vector3 direction in rayDirections)
    {
        Ray ray = new Ray(player.transform.position, direction);
        RaycastHit[] hits = Physics.SphereCastAll(ray, 0.5f, 15.0f, enemyLayerMask);

        foreach (RaycastHit hit in hits)
        {
            if (EnemyManager.Instance.GetEnemy(hit.collider, out Enemy enemy))
            {
                DamageContainer damageContainer = WeaponUtility.GetDamageContainer(
                    finalDamage, 0.5f, damageSource, direction, enemy);
                enemy.DamageFromPlayerOther(damageContainer);
                EffectManager.Instance.EnemyHitEffect(
                    enemy.GetCenterPosition(), Vector3.zero, 1, damageSource);
            }
        }
    }
}
```

## Technical Notes
- **Memory Management**: Uses IL2CPP interop with proper garbage collection barriers
- **Performance Optimization**: Clamps projectile counts to prevent excessive raycasting
- **Thread Safety**: All operations execute on main thread via Unity event system
- **Error Handling**: Includes null checks for all Unity object references

### Event Lifecycle
1. **Init()**: Subscribes to PlayerHealth.A_TakeDamage event
2. **OnTakeDamage()**: Executes counter-attack logic
3. **Cleanup()**: Unsubscribes from PlayerHealth.A_TakeDamage event

### Debugging Considerations
- OnTakeDamage method address: 0x1804035E0
- Static damage source string stored in class static fields
- Particle effects use pooled GameObjects for performance

## Related Items
- **QuinsMask**: Similar thorns-based retaliation mechanics
- **SpikyShield**: Armor-based retaliation system
- **Mirror**: Damage reflection mechanics
- **ElectricPlug**: Chain lightning on damage taken

---

**Data Sources:**
- `megabonk_research/items.md` - Basic property overview
- `extracted_constructors/items/Cactus.c` - Constructor and initialization logic
- `decompiled/Assembly-CSharp/.../ItemCactus.cs` - Class structure and field definitions
- IDA Pro decompilation of OnTakeDamage (0x1804035E0) - Core functionality implementation