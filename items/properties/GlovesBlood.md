# GlovesBlood

## Overview
- **Item ID**: EItem.GlovesBlood
- **Constructor Address**: 0x180458B10
- **Category**: Health/Healing & Elemental/Status
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| cooldown | float | 9.0 | Seconds between activations |
| baseDamageMultiplier | float | 3.15 | Damage scaling factor |
| baseRadius | float | 10.0 | Effect radius in units |
| healPercentage | float | 0.075 | 7.5% heal ratio |
| readyAtTime | float | Dynamic | Next activation time |
| reuseDc | DamageContainer | Dynamic | Reusable damage container |
| fx | EffectPlayer | Dynamic | Visual effect player |
| damageSource | string | Static | Damage source identifier |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | - | - | No direct stat modifications |

## Special Mechanics

### Activation Trigger
- **Event**: On hit effects (`ProcOnHitEffects`)
- **Cooldown**: 9.0 seconds (hardcoded)
- **Condition**: Must be off cooldown (`readyAtTime <= currentTime`)

### Area Explosion Effect
1. **Target Selection**: Gets all enemies within `baseRadius` (10.0 units) of the hit enemy
2. **Damage Calculation**: Uses `GetDamage()` method with `baseDamageMultiplier` (3.15)
3. **Damage Application**: Each enemy takes explosion damage
4. **Bleeding Debuff**: Applies bleeding (debuff ID 64) for 5.0 seconds to all affected enemies
5. **Player Healing**: Player heals for percentage of max HP based on `healPercentage` (7.5%)

### Visual Effects
- **Effect Management**: Creates and manages explosion visual effects
- **Positioning**: Effects are positioned at the hit enemy's location
- **Instantiation**: Effects are instantiated from EffectManager on first use

## Formulas

### Damage Calculation
```
explosionDamage = GetDamage() * baseDamageMultiplier
// Where GetDamage() returns base weapon damage modified by item amount
```

### Healing Amount
```
healAmount = playerMaxHealth * healPercentage * amount
// healPercentage = 0.075 (7.5% per stack)
```

### Cooldown Management
```
nextActivationTime = currentTime + cooldown
// cooldown = 9.0 seconds (constant)
```

### Area of Effect
```
effectRadius = baseRadius
// baseRadius = 10.0 units (constant, no scaling)
```

## Implementation Details
- **Update Frequency**: Event-driven (on hit)
- **Event Subscriptions**: ProcOnHitEffects from ItemBase
- **Stack Behavior**: Multiple stacks increase heal amount linearly

### Debuff Application
- **Type**: Bleeding (ID 64)
- **Duration**: 5.0 seconds
- **Stacks**: Based on item amount

### Performance Optimizations
- **Damage Container Reuse**: Uses `reuseDc` field to avoid allocations
- **Effect Pooling**: Visual effects are instantiated once and reused
- **Lazy Loading**: Effects only created when first needed

## C# Pseudocode
```csharp
// Simplified constructor logic
public ItemGlovesBlood(ItemInventory itemInventoryRef) : base(itemInventoryRef)
{
    cooldown = 9.0f;
    baseDamageMultiplier = 3.15f;
    baseRadius = 10.0f;
    healPercentage = 0.075f;
    reuseDc = new DamageContainer(0.0f, damageSource);
}

// Core activation logic
public override void ProcOnHitEffects(DamageContainer dc)
{
    if (readyAtTime <= MyTime.time && dc?.enemy != null)
    {
        var enemyPosition = dc.enemy.transform.position;
        var enemiesInRadius = WeaponUtility.GetEnemiesInRadius(enemyPosition, baseRadius);

        readyAtTime = MyTime.time + 0.02f; // Small delay for processing

        foreach (var enemyCollider in enemiesInRadius)
        {
            if (EnemyManager.Instance.GetEnemy(enemyCollider, out Enemy enemy))
            {
                // Calculate and apply explosion damage
                float damage = GetDamage();
                var damageContainer = WeaponUtility.GetDamageContainer(
                    reuseDc, damage, 0.0f, damageSource, Vector3.zero, enemy);

                enemy.DamageFromPlayerOther(damageContainer);
                enemy.AddDebuff(64, damageContainer, 5.0f, amount); // Bleeding
            }
        }

        // Heal player
        int maxHp = MyPlayer.Instance.inventory.playerHealth.GetMaxHealth();
        MyPlayer.Instance.inventory.playerHealth.Heal(maxHp, healPercentage);

        // Set actual cooldown and play effects
        readyAtTime = MyTime.time + cooldown;
        PlayExplosionEffect(enemyPosition);
    }
}

private float GetDamage()
{
    return baseDamage * baseDamageMultiplier;
}
```

## Technical Notes

### Memory Management
- Uses IL2CPP garbage collection barriers (`j_j_GC_end_stubborn_change`)
- Reuses DamageContainer instances to reduce allocations
- Effect objects are cached and reused

### Thread Safety
- All operations occur on main thread
- Uses Unity's component system for safe object access

### Error Handling
- Extensive null checks throughout the implementation
- Graceful failure when enemies or components are missing

## Related Items

### Similar Mechanics
- **BloodyCleaver**: Also applies bleeding debuff (64) for 5.0 seconds
- **UnstableTransfusion**: Another bleeding-based item with 5.0 second duration
- **Bonker**: Area damage with radius scaling
- **GlovesPoison**: Similar glove item with area poison effect
- **GlovesLightning**: Similar glove item with area lightning effect

### Synergies
- **Area Multiplier (EStat 9)**: Does not affect this item's radius (hardcoded at 10.0)
- **Healing items**: Stacks additively with other healing sources
- **Bleeding synergies**: Works well with other bleeding-applying items

---

**Data Sources:**
- D:\dev\megabonk\megabonk_research\items.md (lines 500-515)
- D:\dev\megabonk\decompiled\Assembly-CSharp\Assets.Scripts.Inventory__Items__Pickups.Items.ItemImplementations\ItemGlovesBlood.cs
- D:\dev\megabonk\extracted_constructors\items\GlovesBlood.c
- D:\dev\megabonk\decompiled\Assembly-CSharp\Assets.Scripts.Inventory__Items__Pickups.Items\ItemBase.cs