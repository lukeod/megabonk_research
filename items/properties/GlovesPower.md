# GlovesPower

## Overview
- **Item ID**: EItem.GlovesPower
- **Constructor Address**: 0x18040FD30
- **Category**: Special Attack/Area Effect
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| knockbackForce | float | 9999.0 | Extreme knockback strength |
| procChancePerAmount | float | 0.08 | 8% proc chance per stack |
| procChance | float | Calculated | Hyperbolic scaling result |
| baseDamageMultiplier | float | 1.0 | Base damage scaling |
| radiusPerAmount | float | 5.0 | Radius increase per stack |
| radius | float | Calculated | Dynamic based on amount |
| cooldown | float | Calculated | Dynamic cooldown |
| readyAtTime | float | Dynamic | Next proc availability |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | No direct stat modifications | N/A | N/A |

## Special Mechanics

### Proc System
- **Trigger**: On hit effects (ProcOnHitEffects)
- **Chance**: Uses hyperbolic scaling: `procChance = (amount * 0.08) / ((amount * 0.08) + 0.5)`
- **Cooldown**: Dynamic cooldown prevents rapid procs

### Knockback Effect
- **Force**: 9999.0 (extremely high knockback)
- **Area**: Affects all enemies within calculated radius
- **Damage**: Uses GetDamage() method for area damage calculation

### Area of Effect
- **Base Radius**: 8.0 units
- **Final Radius**: `(amount * 5.0) + 10.0`
- **Target Selection**: Gets all enemies within radius using WeaponUtility

### Visual Effects
- **Effect**: Instantiates and plays EffectPlayer on proc
- **Scaling**: Effect scales with calculated radius
- **Position**: Positioned at player location

## Formulas

### Cooldown Calculation
```
cooldown = clamp(3.2 - (amount * 0.2), 0.2, 2.0)
```
- Decreases from 3.2 seconds at 1 stack
- Minimum: 0.2 seconds
- Maximum: 2.0 seconds

### Radius Calculation
```
radius = (amount * radiusPerAmount) + 10.0
radius = (amount * 5.0) + 10.0
```

### Proc Chance (Hyperbolic Scaling)
```
baseChance = amount * procChancePerAmount
procChance = baseChance / (baseChance + 0.5)
```

## Implementation Details
- **Update Frequency**: OnInitOrAmountChanged recalculates values when stack changes
- **Event Subscriptions**: Responds to hit events via ProcOnHitEffects
- **Stack Behavior**: Linear scaling for most properties, hyperbolic for proc chance
- **Effect Management**: Reuses DamageContainer for efficiency
- **Target Detection**: Uses WeaponUtility.GetEnemiesInRadius for area targeting

## C# Pseudocode
```csharp
// Simplified constructor logic
public ItemGlovesPower(ItemInventory inventory) : base(inventory)
{
    knockbackForce = 9999.0f;
    procChancePerAmount = 0.08f;
    baseDamageMultiplier = 1.0f;
    radiusPerAmount = 5.0f;
    radius = 8.0f;
    cooldown = 1.0f;

    // Initialize reusable damage container
    reuseDc = new DamageContainer(0.0f, damageSource, false);
}

// Update values when amount changes
protected override void OnInitOrAmountChanged()
{
    // Dynamic cooldown calculation
    float newCooldown = 3.2f - (amount * 0.2f);
    cooldown = Mathf.Clamp(newCooldown, 0.2f, 2.0f);

    // Dynamic radius calculation
    radius = (amount * radiusPerAmount) + 10.0f;

    // Hyperbolic proc chance scaling
    float baseChance = amount * procChancePerAmount;
    procChance = baseChance / (baseChance + 0.5f);
}

// Proc effect implementation
public override void ProcOnHitEffects(DamageContainer dc)
{
    if (Time.time < readyAtTime) return;
    if (!TryProc(dc.procCoefficient, procChance)) return;

    // Set next ready time
    readyAtTime = Time.time + cooldown;

    // Get all enemies in radius and apply knockback damage
    var enemies = WeaponUtility.GetEnemiesInRadius(dc.enemy.position, radius);
    foreach (var enemy in enemies)
    {
        var damageContainer = GetDamageContainer(reuseDc, GetDamage(), 0.0f, damageSource, direction, enemy);
        damageContainer.knockbackForce = knockbackForce; // 9999.0
        enemy.DamageFromPlayerOther(damageContainer);
    }

    // Play visual effect
    PlayEffectAtPlayerPosition();
}
```

## Technical Notes
- **Performance**: Reuses DamageContainer to minimize garbage collection
- **Knockback Scaling**: Extreme knockback value (9999.0) provides massive displacement
- **Hyperbolic Scaling**: Prevents proc chance from reaching 100%, maintaining balance
- **Effect Pooling**: Visual effects are instantiated once and reused
- **Cooldown Management**: Prevents rapid successive procs while allowing reasonable frequency

## Related Items
- **Other Gloves Series**: GlovesBlood, GlovesCursed, GlovesLightning, GlovesPoison
- **Knockback Items**: Bonker (area damage with knockback)
- **Area Effect Items**: ElectricPlug, ToxicBarrel, SpicyMeatball

---

**Data Sources**:
- megabonk_research/items.md (lines 570-586)
- extracted_constructors/items/GlovesPower.c
- decompiled/Assembly-CSharp/Assets.Scripts.Inventory__Items__Pickups.Items.ItemImplementations/ItemGlovesPower.cs
- decompiled/Assembly-CSharp/Assets.Scripts.Inventory__Items__Pickups.Items/ItemBase.cs