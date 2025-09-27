# SoulHarvester

## Overview
- **Item ID**: 47 (EItem.SoulHarvester)
- **Constructor Address**: 0x180424800
- **Category**: Death-Triggered Projectile Item
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| damageSource | string | "SoulHarvester" | Damage source identifier (from enum toString) |
| numProjectiles | int | 2 | Base number of projectiles spawned |
| damageMultiplier | float | amount | Set to current stack amount |
| maxProjectiles | static int | Unknown | Static field, value not visible in constructor |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | N/A | N/A | No direct stat modifications |

## Special Mechanics
- **Event-Driven**: Subscribes to the global `Enemy.A_EnemyReleasedFromPool` event during `Init()`
- **Death Response**: Triggers `OnEnemyDied()` method when any enemy dies
- **Projectile Spawning**: Spawns projectiles at enemy death location via `SpawnProjectile(Vector3 pos)`
- **Dynamic Damage**: Damage multiplier equals the current stack amount (linear scaling)

## Formulas
- **Damage Multiplier**: `damageMultiplier = amount` (where amount is stack count)
- **Projectiles per Activation**: Fixed at 2 base projectiles (may be modified by other mechanics)

## Implementation Details
- **Update Frequency**: Event-driven (no periodic updates)
- **Event Subscriptions**:
  - Subscribes to `Enemy.A_EnemyReleasedFromPool` on initialization
  - Unsubscribes during cleanup to prevent memory leaks
- **Stack Behavior**: Each stack increases damage multiplier linearly

## C# Pseudocode
```csharp
// Constructor logic
public ItemSoulHarvester(ItemInventory itemInventoryRef) {
    damageSource = "SoulHarvester";  // EItem enum value 47 converted to string
    numProjectiles = 2;
    // Base constructor call
}

// On initialization or amount change
protected override void OnInitOrAmountChanged() {
    damageMultiplier = (float)amount;  // Linear scaling with stack count
}

// Initialization - subscribe to enemy death events
public override void Init() {
    Enemy.A_EnemyReleasedFromPool += OnEnemyDied;
}

// Cleanup - unsubscribe from events
public override void Cleanup() {
    Enemy.A_EnemyReleasedFromPool -= OnEnemyDied;
}

// Triggered when any enemy dies
private void OnEnemyDied(Enemy enemy) {
    Vector3 enemyPosition = enemy.transform.position;
    SpawnProjectile(enemyPosition);
}

// Spawn projectiles at specified location
private void SpawnProjectile(Vector3 pos) {
    // Implementation details not visible in decompiled code
    // Likely spawns numProjectiles (2) at the given position
    // Damage scaled by damageMultiplier (= amount)
}
```

## Technical Notes
- Uses Unity's delegate system for efficient event handling
- Properly manages event subscription/unsubscription to prevent memory leaks
- IL2CPP decompiled code shows the item responds to the "EnemyReleasedFromPool" event, which appears to be triggered on enemy death
- The actual projectile spawning and damage calculation logic is implemented in native code and not visible in the C# decompilation
- Static `maxProjectiles` field suggests there may be a cap on projectile spawning, but the value is not set in the constructor

## Related Items
- **DemonicBlood**: Also gains stacks/power on enemy death
- **DemonicSoul**: Similar death-triggered mechanics
- **BobDead**: Another projectile-spawning item (though movement-based rather than death-based)

---

*Data extracted from decompiled IL2CPP constructor at address 0x180424800 and C# class definition*