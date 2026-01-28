# GlovesPoison

## Overview
- **Item ID**: EItem.GlovesPoison
- **Constructor Address**: 0x18045A4C0
- **Category**: Elemental/Status (Area of Effect Poison)
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| cooldown | float | 8.5 | Seconds between activations |
| baseDamageMultiplier | float | 1.5 | Base damage scaling factor |
| baseRadius | float | 15.0 | Area of effect radius |
| poisonStacksPerAmount | int | 10 | Poison stacks applied per item stack |
| readyAtTime | float | dynamic | Time when next activation is ready |
| reuseDc | DamageContainer | dynamic | Reusable damage container object |
| fx | EffectPlayer | dynamic | Visual effect player component |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | N/A | N/A | No direct stat modifications |

## Special Mechanics
- **Poison Area Attack**: On hit proc, damages all enemies within `baseRadius` (15.0 units) of the hit target
- **Poison Debuff**: Applies poison debuff (ID 1) for 5.0 seconds to all affected enemies
- **Poison Stacking**: Applies `poisonStacksPerAmount` (10) poison stacks per activation (does not scale with item amount)
- **Cooldown System**: Has an 8.5 second cooldown after activation, tracked via `readyAtTime`
- **Visual Effects**: Spawns poison effect at target location when activated

## Formulas
- **Damage Calculation**: `GetDamage() * baseDamageMultiplier` (1.5x multiplier)
- **Area Radius**: Fixed at 15.0 units (no scaling)
- **Poison Stacks**: `poisonStacksPerAmount` (fixed at 10 stacks per activation)
- **Poison Duration**: Fixed at 5.0 seconds
- **Cooldown Reset**: `MyTime.time + cooldown` (current time + 8.5 seconds)

## Implementation Details
- **Update Frequency**: Checked on each hit via `ProcOnHitEffects`
- **Event Subscriptions**: Responds to on-hit events from player attacks
- **Stack Behavior**: Poison stacks applied are fixed at 10 per activation (does not scale with item amount)
- **Damage Source**: Uses static `damageSource` field for damage attribution
- **Effect Management**: Instantiates and manages poison visual effects via EffectManager

## C# Pseudocode
```csharp
// Simplified constructor logic
public ItemGlovesPoison(ItemInventory itemInventoryRef)
{
    this.cooldown = 8.5f;
    this.baseDamageMultiplier = 1.5f;
    this.baseRadius = 15.0f;
    this.poisonStacksPerAmount = 10;

    // Create reusable damage container
    this.reuseDc = new DamageContainer(0.0f, damageSource, null);

    base(itemInventoryRef);
}

// Simplified proc logic
public override void ProcOnHitEffects(DamageContainer dc)
{
    if (MyTime.time >= readyAtTime && dc?.enemy != null)
    {
        Vector3 enemyPos = dc.enemy.transform.position;
        Collider[] enemies = WeaponUtility.GetEnemiesInRadius(enemyPos, baseRadius);

        readyAtTime = MyTime.time + 0.02f; // Brief delay during processing

        foreach (var collider in enemies)
        {
            if (EnemyManager.Instance.GetEnemy(collider, out Enemy enemy))
            {
                // Calculate and apply damage
                float damage = GetDamage() * baseDamageMultiplier;
                var damageContainer = WeaponUtility.GetDamageContainer(
                    reuseDc, damage, 0.0f, damageSource, Vector3.zero, enemy);

                enemy.DamageFromPlayerOther(damageContainer);

                // Apply poison debuff
                enemy.AddDebuff(
                    1,                              // Poison debuff type
                    damageContainer,
                    5.0f,                           // Duration
                    poisonStacksPerAmount,          // Fixed at 10 stacks
                    null);
            }
        }

        // Set full cooldown and play effects
        readyAtTime = MyTime.time + cooldown;
        PlayPoisonEffect(enemyPos);
    }
}
```

## Technical Notes
- **Effect Instantiation**: Creates poison effect GameObject on first use via EffectManager
- **Object Reuse**: Efficiently reuses DamageContainer object to minimize allocations
- **Position Tracking**: Effect is positioned at the original target enemy's location
- **Bounds Checking**: Uses array bounds checking for enemy iteration safety
- **Memory Management**: Uses IL2CPP garbage collection barriers for object references

## Related Items
- **GlovesBlood**: Similar area attack with healing and bleeding (9.0s cooldown)
- **GlovesLightning**: Area lightning attack with stun (10.0s cooldown, 8.0 radius)
- **GlovesCursed**: Area curse attack with difficulty scaling (varies by proc chance)
- **ToxicBarrel**: Poison area effect on player damage (0.25s cooldown, smaller radius)
- **MoldyCheese**: Single-target poison application on hit

---

*Data extracted from decompiled IL2CPP constructor at 0x18045A4C0 and C# interface analysis*