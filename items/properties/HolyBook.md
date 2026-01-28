# HolyBook

## Overview
- **Item ID**: 54 (EItem.HolyBook)
- **Constructor Address**: 0x18045D5A0
- **Category**: Health/Healing
- **Rarity**: Standard Item

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| maxHpPerAmount | float | 100.0 | Max HP per stack |
| hpRegenPerAmount | float | 50.0 | HP regeneration per stack |
| overhealPerAmount | float | 0.25 | Overheal bonus per stack |
| radiusPerAmount | float | 1.0 | Radius scaling per stack |
| cooldown | float | 1.5 | Cooldown between activations (seconds) |
| radius | float | Dynamic | Base radius: `amount * radiusPerAmount + 5.0` |
| healsThisTick | float | Variable | Tracks healing to be applied |
| nextDamageTime | float | Dynamic | Next activation time |
| damageSource | string | "HolyBook" | Source identifier for damage |
| dc | DamageContainer | Object | Damage container with 0.5 base damage |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 0 | Max Health | amount * 100.0 | Linear per stack |
| 1 | HP Regeneration | amount * 50.0 | Linear per stack |
| 47 | Overheal | amount * 0.25 | Linear per stack |

## Special Mechanics

### Healing System
- Subscribes to global healing events via `PlayerHealth.A_Heal`
- Tracks healing received in `healsThisTick` variable
- When player receives healing, accumulates the amount for later processing

### Area Damage Activation
- **Trigger Condition**: When `healsThisTick > 0` and cooldown has elapsed
- **Activation Frequency**: Every 1.5 seconds (cooldown)
- **Effect Radius**: `EStat(9) * (amount * 1.0 + 5.0)` - scales with Area multiplier stat
- **Damage Calculation**: `(playerMaxHP * 0.5) * healsThisTick`
- **Target Selection**: All enemies within radius

### Damage Processing
1. Calculates effective radius using EStat 9 (Area multiplier)
2. Finds all enemies within range using `WeaponUtility.GetEnemiesInRadius`
3. For each enemy:
   - Creates damage container with calculated damage
   - Applies damage via `Enemy.DamageFromPlayerOther`
   - Spawns visual effect at enemy position
4. Resets `healsThisTick` to 0 after processing

## Formulas

### Effective Radius
```
effectiveRadius = EStat(9) * (amount * 1.0 + 5.0)
```

### Damage per Enemy
```
damage = (playerMaxHP * 0.5) * healsThisTick
```

### Base Radius (without EStat scaling)
```
baseRadius = amount * 1.0 + 5.0
```

## Implementation Details
- **Update Frequency**: Continuous (Tick method called every frame)
- **Event Subscriptions**:
  - `PlayerHealth.A_Heal` (subscribed in Init, unsubscribed in Cleanup)
- **Stack Behavior**: All values scale linearly with stack count
- **Damage Type**: Player-sourced damage with "HolyBook" identifier

## C# Pseudocode
```csharp
// Simplified constructor logic
public ItemHolyBook(ItemInventory itemInventoryRef) : base(itemInventoryRef)
{
    maxHpPerAmount = 100.0f;
    hpRegenPerAmount = 50.0f;
    overhealPerAmount = 0.25f;
    radiusPerAmount = 1.0f;
    cooldown = 1.5f;
    damageSource = "HolyBook";
    dc = new DamageContainer(0.5f, damageSource);
}

// Stat application
protected override void OnInitOrAmountChanged()
{
    SetStat(new StatModifier(EStat.MaxHealth, amount * maxHpPerAmount));
    SetStat(new StatModifier(EStat.HPRegen, amount * hpRegenPerAmount));
    SetStat(new StatModifier(EStat.Overheal, amount * overhealPerAmount));
    radius = amount * radiusPerAmount + 5.0f;
}

// Event handling
public override void Init()
{
    PlayerHealth.A_Heal += OnHeal;
}

private void OnHeal(PlayerHealth ph, float hpHealed, bool isShield)
{
    healsThisTick += hpHealed;
}

// Main activation logic
public override void Tick()
{
    if (healsThisTick > 0 && MyTime.time >= nextDamageTime)
    {
        nextDamageTime = MyTime.time + cooldown;

        float effectiveRadius = PlayerStats.GetStat(EStat.AreaMultiplier) * radius;
        float damage = (MyPlayer.Instance.maxHP * 0.5f) * healsThisTick;

        healsThisTick = 0;

        var enemies = WeaponUtility.GetEnemiesInRadius(playerPosition, effectiveRadius);
        foreach (var enemy in enemies)
        {
            enemy.DamageFromPlayerOther(GetDamageContainer(damage));
            EffectManager.Instance.EnemyHitEffect(enemy.position, direction, damageSource);
        }

        A_OnUse?.Invoke(healsThisTick);
    }
}
```

## Technical Notes

### Performance Considerations
- Uses object pooling for damage containers
- Efficient radius-based enemy finding
- Cooldown prevents excessive activation

### Event System Integration
- Properly subscribes/unsubscribes from global heal events
- Uses Unity's delegate system for loose coupling
- Cleanup method ensures no memory leaks

### Scaling Mechanics
- All numerical values scale linearly with stack count
- Radius benefits from EStat 9 (Area multiplier) stat
- Damage scales with both healing received and player max HP

## Related Items
- **Medkit**: Pure HP regeneration item
- **Chonkplate**: Combines overheal with lifesteal
- **Beacon**: Area-based healing item
- **LeechingCrystal**: High HP with negative regen trade-off

---

*Data extracted from decompiled IL2CPP constructors and C# source analysis*