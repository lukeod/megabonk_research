# GlovesLightning

## Overview
- **Item ID**: ItemGlovesLightning (from C# class name)
- **Constructor Address**: 0x180459DA0
- **Category**: Elemental/Status (Active Lightning Attack)
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| cooldown | float | 10.0 | Cooldown between activations in seconds |
| baseDamageMultiplier | float | 3.0 | Base damage scaling factor |
| baseRadius | float | 8.0 | Area of effect radius |
| readyAtTime | float | Dynamic | Tracks when item can proc again |
| reuseDc | DamageContainer | Object | Reusable damage container for efficiency |
| fx | EffectPlayer | Object | Visual effect player component |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | - | - | No direct stat modifications |

## Special Mechanics
- **Lightning Area Attack**: On successful hit (ProcOnHitEffects), deals lightning damage to all enemies within radius
- **Lightning Debuff**: Applies lightning/stun debuff (ID 8) for 3.0 seconds to all affected enemies
- **Cooldown System**: Uses 10-second cooldown with 0.02-second minimum trigger interval
- **Visual Effects**: Instantiates and positions lightning effect at enemy location
- **Single Target Trigger**: Requires hitting an enemy to activate area effect

## Formulas
- **Damage Calculation**: `GetDamage() * baseDamageMultiplier (3.0)`
- **Effect Radius**: Fixed `baseRadius (8.0)` units
- **Cooldown Formula**: Fixed `10.0` seconds
- **Debuff Duration**: Fixed `3.0` seconds for lightning debuff

## Implementation Details
- **Update Frequency**: No periodic updates (event-driven only)
- **Event Subscriptions**: Responds to ProcOnHitEffects events
- **Stack Behavior**: No explicit stacking behavior mentioned in code
- **Damage Source**: Uses static damageSource field from class
- **Effect Management**: Lazy-loads visual effect on first use, reuses for subsequent activations

## C# Pseudocode
```csharp
// Simplified constructor logic
public ItemGlovesLightning(ItemInventory itemInventoryRef) : base(itemInventoryRef)
{
    cooldown = 10.0f;
    baseDamageMultiplier = 3.0f;
    baseRadius = 8.0f;
    reuseDc = new DamageContainer(0.0f, damageSource);
}

// Main proc logic
public override void ProcOnHitEffects(DamageContainer dc)
{
    if (readyAtTime <= MyTime.time && dc?.enemy != null && !dc.enemy.IsDeadOrDyingNextFrame())
    {
        // Set minimum trigger interval
        readyAtTime = MyTime.time + 0.02f;

        Vector3 enemyPosition = dc.enemy.transform.position;
        var enemiesInRadius = WeaponUtility.GetEnemiesInRadius(enemyPosition, baseRadius);

        foreach (var collider in enemiesInRadius)
        {
            if (EnemyManager.Instance.GetEnemy(collider, out Enemy enemy))
            {
                // Create damage container and apply damage
                float damage = GetDamage();
                reuseDc = WeaponUtility.GetDamageContainer(reuseDc, damage, 0.0f, damageSource, Vector3.zero, enemy);
                enemy.DamageFromPlayerOther(reuseDc);

                // Apply lightning debuff
                enemy.AddDebuff(8, reuseDc, 3.0f, 1);
            }
        }

        // Set actual cooldown and play effect
        readyAtTime = MyTime.time + cooldown;
        PlayLightningEffect(dc.enemy.transform.position);
    }
}
```

## Technical Notes
- **Performance Optimization**: Reuses DamageContainer object to avoid garbage collection
- **Effect Instantiation**: Visual effect is instantiated from EffectManager on first use and reused
- **Enemy Validation**: Checks for dead/dying enemies before applying effects
- **Collision Detection**: Uses WeaponUtility.GetEnemiesInRadius for area detection
- **Static Damage Source**: Uses class-level static damageSource field

## Related Items
- **GlovesBlood**: Similar area-effect gloves with healing component
- **GlovesPoison**: Area poison variant with larger radius (15.0 vs 8.0)
- **GlovesPower**: Knockback-focused variant with dynamic cooldown
- **ElectricPlug**: Another lightning-based item with chain mechanics
- **LightningOrb**: Alternative lightning item with random targeting

---

*Data extracted from:*
- *megabonk_research/items.md (line 537-551)*
- *decompiled/Assembly-CSharp/Assets.Scripts.Inventory__Items__Pickups.Items.ItemImplementations/ItemGlovesLightning.cs*
- *extracted_constructors/items/GlovesLightning.c (constructor at 0x180459DA0, ProcOnHitEffects at 0x1804597A0)*