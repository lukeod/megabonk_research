# BobDead

## Overview
- **Item ID**: EItem.BobDead (46)
- **Constructor Address**: 0x18043AEC0
- **Category**: Movement-triggered projectile spawner
- **Rarity**: Unknown (requires additional data)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| damageSource | string | "BobDead" | Damage source identifier |
| unitsPerProjectile | float | 14.0 | Distance player must move to spawn a projectile |
| minSpawnTime | float | 0.05 | Minimum time between spawn checks (50ms) |
| maxGhosts | int | Static field | Maximum number of ghosts (value TBD) |
| nextCheckTime | float | Dynamic | Next time to check for spawning |
| accumulatedDistance | float | Dynamic | Distance accumulated since last spawn |
| lastPos | Vector3 | Dynamic | Last recorded player position |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 10 | DurationMultiplier | Used in damage calculation | Multiplier * 10.0 |

## Special Mechanics
- **Movement-based Spawning**: Spawns ghost projectiles when the player moves sufficient distance
- **Distance Tracking**: Continuously tracks player movement and accumulates distance
- **Spawn Rate Limiting**: Uses `minSpawnTime` to prevent excessive spawning
- **Multi-projectile**: Spawns `amount` number of projectiles per trigger

## Formulas
- **Ghost Damage**: `baseDamage * 1.5`
- **EStat 10 Multiplier**: `GetStat(10) * 10.0` (DurationMultiplier * 10)
- **Distance Threshold**: Player must move ≥ `unitsPerProjectile` units (14.0)
- **Spawn Count**: `amount` projectiles per distance threshold reached

## Implementation Details
- **Update Frequency**: Checked every `minSpawnTime` seconds (0.05s = 20Hz)
- **Event Subscriptions**: None (runs on Tick)
- **Stack Behavior**: Each stack adds one additional ghost projectile per spawn

## C# Pseudocode
```csharp
// Constructor logic
public ItemBobDead(ItemInventory itemInventoryRef) {
    this.damageSource = "BobDead";
    this.unitsPerProjectile = 14.0f;
    this.minSpawnTime = 0.05f;
    // Initialize last position to current player position
    this.lastPos = MyPlayer.Instance.transform.position;
}

// Tick logic
public override void Tick() {
    if (MyTime.time <= nextCheckTime) return;

    nextCheckTime = MyTime.time + minSpawnTime;

    Vector3 currentPos = MyPlayer.Instance.transform.position;
    Vector3 movement = currentPos - lastPos;
    float distanceMoved = movement.magnitude;

    accumulatedDistance += distanceMoved;

    if (accumulatedDistance >= unitsPerProjectile) {
        accumulatedDistance = 0.0f;

        // Spawn ghosts for each stack
        for (int i = 0; i < amount; i++) {
            float damage = MyPlayer.Instance.baseDamage * 1.5f;
            float estat10Multiplier = PlayerStats.GetStat(10) * 10.0f;
            EffectManager.Instance.SpawnGhostProjectile(
                damage,
                estat10Multiplier,
                damageSource
            );
        }
    }

    lastPos = currentPos;
}
```

## Technical Notes
- **Performance**: Efficient distance-based spawning with rate limiting
- **Position Tracking**: Updates player position reference after each check
- **Ghost Projectiles**: Uses EffectManager to spawn ghost projectiles with enhanced damage
- **EStat Integration**: Leverages DurationMultiplier stat with 10x scaling factor
- **Stack Scaling**: Linear scaling - each stack adds one more ghost per spawn trigger

## Related Items
- **Ghost**: Similar ghost-based mechanics
- **EnergyCore**: Periodic projectile spawning pattern
- **SoulHarvester**: Event-triggered projectile spawning

---

**Data Sources:**
- `megabonk_research/items.md` - BobDead section
- `extracted_constructors/items/BobDead.c` - Decompiled constructor at 0x18043AEC0
- `decompiled/Assembly-CSharp/Assets.Scripts.Inventory__Items__Pickups.Items.ItemImplementations/ItemBobDead.cs` - C# interface
- `decompiled/Assembly-CSharp/Assets.Scripts.Menu.Shop/EStat.cs` - EStat enum definitions