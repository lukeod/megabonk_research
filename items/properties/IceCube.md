# IceCube

## Overview
- **Item ID**: EItem.IceCube (ID: 18)
- **Constructor Address**: 0x18045DE40
- **Category**: Elemental/Status
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| procChancePerAmount | float | 0.2 | 20% proc chance per stack |
| freezeChancePerAmount | float | 0.4 | 40% freeze chance per stack |
| damageRatio | float | 0.8 | 80% base damage multiplier |
| damageRatioPerAmount | float | 0.4 | 40% damage per stack |
| damageSource | string | "IceCube" | Source identifier for damage tracking |
| reuseDc | DamageContainer | - | Reusable damage container for projectiles |

## Stat Modifiers
No direct EStat modifications - this item works through proc effects and damage containers.

## Special Mechanics
- **Dual Proc System**: Has two separate proc chances for different effects
- **Ice Projectile Creation**: Spawns ice projectiles on proc that deal damage and freeze
- **Freeze on Hit**: Initial hit can apply freeze debuff if element is ice (element ID 2)
- **Projectile Freeze**: Spawned projectiles can also apply freeze debuff

## Formulas

### Proc Chance Calculation (Hyperbolic Scaling)
```
procChance = (amount * procChancePerAmount) / ((amount * procChancePerAmount) + 0.5)
```
Where:
- `amount` = stack count
- `procChancePerAmount` = 0.2 (20% per stack)

### Freeze Chance Calculation (Hyperbolic Scaling)
```
freezeChance = (amount * freezeChancePerAmount) / ((amount * freezeChancePerAmount) + 0.6)
```
Where:
- `amount` = stack count
- `freezeChancePerAmount` = 0.4 (40% per stack)

### Damage Calculation
```
finalDamage = originalDamage * (amount * damageRatioPerAmount)
```
Where:
- `originalDamage` = incoming damage from DamageContainer
- `amount` = stack count
- `damageRatioPerAmount` = 0.4 (40% per stack)

Note: The base `damageRatio` of 0.8 appears to be overwritten during `OnInitOrAmountChanged()`

## Implementation Details
- **Update Frequency**: On amount change via `OnInitOrAmountChanged()`
- **Event Subscriptions**: Responds to `ProcOnHitEffects` events
- **Stack Behavior**: Both chances use hyperbolic scaling to prevent 100% rates
- **Freeze Duration**: 3.0 seconds (hardcoded)
- **Debuff Type**: Freeze (ID: 2)

## C# Pseudocode
```csharp
// Constructor logic
public ItemIceCube(ItemInventory itemInventoryRef)
{
    procChancePerAmount = 0.2f;
    freezeChancePerAmount = 0.4f;
    damageRatio = 0.8f;
    damageRatioPerAmount = 0.4f;
    damageSource = "IceCube";

    // Create reusable damage container
    reuseDc = new DamageContainer(0.0f, damageSource);
}

// Update calculations when amount changes
protected override void OnInitOrAmountChanged()
{
    // Hyperbolic scaling for proc chance
    procChance = (amount * procChancePerAmount) /
                 ((amount * procChancePerAmount) + 0.5f);

    // Hyperbolic scaling for freeze chance
    freezeChance = (amount * freezeChancePerAmount) /
                   ((amount * freezeChancePerAmount) + 0.6f);

    // Linear scaling for damage
    damageRatio = amount * damageRatioPerAmount;
}

// Main proc effect
public override void ProcOnHitEffects(DamageContainer dc)
{
    // If hit with ice damage, try to freeze immediately
    if (dc.element == 2) // Ice element
    {
        if (ItemUtility.TryProc(dc.procCoefficient, freezeChance))
        {
            dc.enemy.AddDebuff(2, dc, 3.0f, 1); // Freeze for 3 seconds
            A_FreezeEnemy?.Invoke(); // Trigger freeze action
        }
    }

    // Try to proc ice projectile
    if (ItemUtility.TryProc(dc.procCoefficient, procChance))
    {
        // Spawn ice projectile from pool
        GameObject projectile = PoolManager.Instance.GetProjectile();

        // Position near enemy with random offset
        Vector3 enemyPos = dc.enemy.GetCenterPosition();
        Vector3 randomOffset = Random.insideUnitSphere.XZVector() * 0.5f;
        projectile.transform.position = enemyPos + randomOffset;

        // Configure damage container
        reuseDc.damage = damageRatio * dc.damage;
        reuseDc.enemy = dc.enemy;
        reuseDc.element = 2; // Ice element

        // Apply projectile damage
        dc.enemy.DamageFromPlayerOther(reuseDc);

        // Try to freeze with projectile
        if (ItemUtility.TryProc(1.0f, freezeChance))
        {
            reuseDc.enemy.AddDebuff(2, reuseDc, 3.0f, 1);
            A_FreezeEnemy?.Invoke();
        }
    }
}
```

## Technical Notes
- Uses Unity's object pooling system for projectile management
- Implements hyperbolic scaling on both proc and freeze chances to prevent 100% rates
- The 0.5 and 0.6 constants in hyperbolic formulas are balance factors
- Projectiles spawn with slight random positioning around the target
- Two separate freeze checks: one for ice-element hits, one for spawned projectiles
- Static action `A_FreezeEnemy` can be subscribed to for additional freeze effects

## Related Items
- **IceCrystal**: Similar freeze mechanics but different proc system
- **LightningOrb**: Similar projectile spawning with status effects
- **Dragonfire**: Comparable elemental proc system with burn instead of freeze

---

*Data extracted from constructor at 0x18045DE40 and decompiled C# implementation*