# QuinsMask

## Overview
- **Item ID**: EItem.QuinsMask (11)
- **Constructor Address**: 0x180422B10
- **Category**: Defensive/Retaliation
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| thornsPerAmount | float | 20.0 | Thorns damage per stack |
| baseRadius | float | 5.0 | Base spread radius |
| radiusPerAmount | float | 1.0 | Radius increase per stack |
| maxRadius | float | 10.0 | Maximum spread radius |
| damageSpreadMultiplier | float | 0.5 | 50% damage spread to nearby enemies |
| procChance | float | 0.5 | 50% chance to proc spread effect |
| maxProcsPerTick | int | 100 | Maximum procs per game tick |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 3 | Thorns/Retaliation | amount * 20.0 | Linear |

## Special Mechanics

### Damage Sources Filtering
The mask only responds to specific damage sources:
- Player thorns damage (from PlayerHealth.thornsDamageSource)
- Cactus item damage (from ItemCactus.damageSource)
- Aegis weapon damage (weapon ID 7 from DataManager)

### Area Damage Spreading
When triggered:
1. **Proc Check**: 50% chance per hit from valid damage sources
2. **Target Selection**: Finds all enemies within calculated radius around the original target
3. **Damage Application**: Deals 50% of original damage to each enemy in range
4. **Visual Effect**: Spawns particle effects at random positions around the target

### Radius Calculation
The effective radius is calculated as:
```
effectiveRadius = EStat9 * ((amount * radiusPerAmount) + baseRadius)
effectiveRadius = clamp(effectiveRadius, 1.0, maxRadius)
```
- EStat 9 is the Area/Radius Multiplier stat
- Base formula: `(stacks * 1.0) + 5.0`
- Clamped between 1.0 and 10.0 units
- Further modified by EStat 9 multiplier

## Formulas

### Thorns Damage
```
thornsDamage = amount * 20.0
```

### Spread Damage
```
spreadDamage = originalDamage * 0.5
```

### Effective Radius
```
baseRadiusCalculation = (amount * 1.0) + 5.0
effectiveRadius = clamp(EStat9 * baseRadiusCalculation, 1.0, 10.0)
```

### Proc Rate
```
procRate = 0.5 (fixed 50% chance)
```

## Implementation Details
- **Update Frequency**: On damage events (event-driven)
- **Event Subscriptions**: Responds to player damage events with specific damage sources
- **Stack Behavior**: Each stack increases thorns stat by 20.0 and radius by 1.0
- **Proc Limiting**: Maximum 100 procs per tick to prevent performance issues

## C# Pseudocode
```csharp
// Simplified constructor logic
public ItemQuinsMask(ItemInventory itemInventoryRef) : base(itemInventoryRef)
{
    thornsPerAmount = 20.0f;
    baseRadius = 5.0f;
    radiusPerAmount = 1.0f;
    maxRadius = 10.0f;
    damageSpreadMultiplier = 0.5f;
    procChance = 0.5f;
    maxProcsPerTick = 100;

    // Setup damage container for spread damage
    procDc = new DamageContainer(0.0f, "QuinsMask", null);

    // Initialize damage source filtering
    damageSources = new HashSet<string>();
    damageSources.Add(PlayerHealth.thornsDamageSource);
    damageSources.Add(ItemCactus.damageSource);
    damageSources.Add(DataManager.GetWeapon(7).damageSourceName); // Aegis
}

protected override void OnInitOrAmountChanged()
{
    // Calculate effective radius
    float calculatedRadius = EStat9 * ((amount * radiusPerAmount) + baseRadius);
    radius = Mathf.Clamp(calculatedRadius, 1.0f, maxRadius);

    // Set thorns stat modifier
    SetStat(new StatModifier(EStat.Thorns, amount * thornsPerAmount));
}

public override void ProcOnHitEffects(DamageContainer dc)
{
    if (numProcsThisTick >= maxProcsPerTick) return;
    if (!damageSources.Contains(dc.damageSource)) return;
    if (!ItemUtility.TryProc(procChance, dc.procCoefficient)) return;

    numProcsThisTick++;

    // Find enemies in radius
    Vector3 targetPos = dc.enemy.GetCenterPosition();
    var enemiesInRadius = EnemyTargeting.GetEnemiesInRadius(targetPos, radius);

    // Damage each enemy
    foreach (var enemy in enemiesInRadius)
    {
        if (enemy.IsDead()) continue;

        procDc.Reuse(0.0f, damageSource);
        procDc.damage = damageSpreadMultiplier * dc.damage;
        procDc.enemy = enemy;

        enemy.DamageFromPlayerOther(procDc);
    }

    // Spawn visual effect
    SpawnParticleEffect(targetPos);
}

public override void Tick()
{
    numProcsThisTick = 0; // Reset proc counter each tick
}
```

## Technical Notes
- Uses hyperbolic area scaling with EStat 9 (Area/Radius Multiplier)
- Particle effects are spawned from object pool for performance
- Damage source filtering prevents infinite loops with other thorns effects
- Proc limiting (100/tick) prevents performance degradation in high-density scenarios
- The aegis damage source is dynamically retrieved from weapon data (ID 7)

## Related Items
- **Cactus**: Also triggers on player damage, creates synergy
- **SpikyShield**: Another thorns-based item, stacks multiplicatively
- **Mirror**: Similar defensive retaliation concept
- **Aegis Weapon**: One of the trigger sources for the spread effect

---

*Data extracted from IL2CPP constructor at 0x180422B10 via IDA Pro MCP analysis*