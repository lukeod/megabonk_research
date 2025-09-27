# Kevin

## Overview
- **Item ID**: ItemKevin
- **Constructor Address**: 0x180414E20
- **Category**: Self-Damage / Risk-Reward
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| damageChancePerAmount | float | 0.25 | 25% chance per stack |
| damageChance | float | Dynamic | Calculated from amount |
| numHits | int | Dynamic | Tracks accumulated hits |
| damageSource | string | Static | Damage source identifier |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | N/A | N/A | N/A |

Kevin does not directly modify any player stats through the EStat system.

## Special Mechanics

Kevin implements a unique self-damage system that works as follows:

1. **Enemy Damage Tracking**:
   - Subscribes to the global `Enemy.A_Damage` event during `Init()`
   - Each time any enemy takes damage, `OnEnemyDamaged()` is called
   - If player health > 1, increments `numHits` counter

2. **Self-Damage Resolution**:
   - Called every frame via `Tick()` → `CheckSelfDamage()`
   - Only triggers if player health > 1.0
   - For each accumulated hit in `numHits`:
     - Calculates damage based on `damageChance` (25% per stack)
     - Uses floor + fractional chance system for damage calculation
     - Applies accumulated damage to player if health would remain > 1
   - Resets `numHits` to 0 after processing

3. **Damage Calculation**:
   - Each hit: `floor(damageChance) + (random() < fractional_part ? 1 : 0)`
   - Total damage = sum of all hits processed
   - Minimum player health after damage = 1 (cannot kill player)

## Formulas

### Damage Chance Per Hit
```
damageChance = amount * 0.25
```

### Damage Per Hit Calculation
```
baseDamage = floor(damageChance)
extraDamage = (random(0,1) < (damageChance - baseDamage)) ? 1 : 0
finalDamage = baseDamage + extraDamage
```

### Safety Check
```
if (playerHealth - totalDamage <= 1) {
    totalDamage = playerHealth - 1
}
```

## Implementation Details
- **Update Frequency**: Every frame via `Tick()`
- **Event Subscriptions**: `Enemy.A_Damage` (global enemy damage event)
- **Stack Behavior**: Linear scaling of damage chance per stack
- **Safety Mechanism**: Cannot reduce player health below 1 HP

## C# Pseudocode
```csharp
// Constructor
public ItemKevin(ItemInventory itemInventoryRef) {
    damageChancePerAmount = 0.25f;
    base(itemInventoryRef);
}

// Event subscription
public override void Init() {
    Enemy.A_Damage += OnEnemyDamaged;
}

// Track enemy damage events
private void OnEnemyDamaged(Enemy enemy, DamageContainer dc) {
    if (MyPlayer.Instance.inventory.playerHealth.currentHP > 1.0f) {
        numHits++;
    }
}

// Process self-damage each frame
public override void Tick() {
    CheckSelfDamage();
}

private void CheckSelfDamage() {
    var playerHealth = MyPlayer.Instance.inventory.playerHealth;

    if (playerHealth.currentHP > 1.0f && numHits > 0) {
        damageChance = amount * damageChancePerAmount; // amount * 0.25

        int totalDamage = 0;
        for (int i = 0; i < numHits; i++) {
            int baseDamage = (int)Math.Floor(damageChance);
            float fractionalChance = damageChance - baseDamage;
            int extraDamage = (MyRandom.random.NextDouble() < fractionalChance) ? 1 : 0;
            totalDamage += baseDamage + extraDamage;
        }

        numHits = 0;

        // Ensure player doesn't die
        if (playerHealth.currentHP - totalDamage <= 1) {
            totalDamage = (int)playerHealth.currentHP - 1;
        }

        if (totalDamage >= 1) {
            playerHealth.DamagePlayerExternal(
                totalDamage, 0f, Vector3.zero,
                true, damageSource, 5, false, false, false
            );
        }
    }
}
```

## Technical Notes

1. **Event Management**: Kevin properly subscribes/unsubscribes from the global damage event in `Init()`/`Cleanup()`
2. **Performance**: Processes damage every frame, but only when `numHits > 0` and player health > 1
3. **Safety First**: Multiple safeguards prevent Kevin from killing the player
4. **Fractional Damage**: Uses probabilistic rounding for non-integer damage values
5. **Global Scope**: Triggers on ANY enemy taking damage, not just player-caused damage

## Related Items
- **Risk-Reward Items**: Beer (health reduction for damage), GlovesCursed (health reduction with difficulty increase)
- **Self-Damage Mechanics**: No direct equivalents, Kevin is unique in its self-damage approach

---

*Data extracted from decompiled IL2CPP constructors and native code analysis via IDA Pro*