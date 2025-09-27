# GlovesCursed

## Overview
- **Item ID**: ItemGlovesCursed
- **Constructor Address**: 0x18040E3F0
- **Category**: Cursed Weapon/Area Effect
- **Rarity**: Unknown (likely rare due to cursed nature)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| procChancePerAmount | float | 0.05 | 5% proc chance per stack |
| difficultyPerAmount | float | 0.1 | 10% difficulty increase per stack |
| maxHpMultiplierPerAmount | float | 0.8 | 80% HP multiplier per stack (reduces HP) |
| baseDamageMultiplier | float | 0.85 | 85% base damage scaling |
| baseRadius | float | 4.0 | Area of effect radius |
| procChance | float | Calculated | Final calculated proc chance |
| reuseDc | DamageContainer | Object | Reusable damage container for efficiency |
| fx | EffectPlayer | Object | Visual effect player |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 0 | Max Health | pow(0.8, amount) | Exponential (multiplicative) |
| 38 | Difficulty | amount * 0.1 | Linear additive |

## Special Mechanics
- **Risk/Reward Design**: Reduces player HP exponentially while increasing difficulty and providing powerful area damage
- **Proc-Based Activation**: Triggers on hit with chance based on proc coefficient
- **Area Damage**: Damages all enemies within baseRadius (4.0 units) of the target
- **Hyperbolic Scaling**: Proc chance uses formula: `(baseChance / (baseChance + 0.8)) * 0.5`
- **Visual Effects**: Instantiates and plays effect at target location on proc

## Formulas

### Proc Chance Calculation
```
rawChance = amount * procChancePerAmount  // amount * 0.05
finalChance = (rawChance / (rawChance + 0.8)) * 0.5
```

### Health Reduction
```
healthMultiplier = pow(0.8, amount)
// 1 stack: 80% HP, 2 stacks: 64% HP, 3 stacks: 51.2% HP, etc.
```

### Difficulty Increase
```
difficultyIncrease = amount * 0.1
```

### Damage Calculation
```
damage = GetDamage() * baseDamageMultiplier  // * 0.85
```

## Implementation Details
- **Update Frequency**: Event-driven (on hit)
- **Event Subscriptions**: ProcOnHitEffects
- **Stack Behavior**:
  - Proc chance scales hyperbolically (diminishing returns)
  - Health penalty scales exponentially (increasingly severe)
  - Difficulty scales linearly
- **Performance Optimization**: Uses reusable DamageContainer to avoid allocation overhead

## C# Pseudocode
```csharp
// Constructor logic
public ItemGlovesCursed(ItemInventory itemInventory) {
    procChancePerAmount = 0.05f;
    difficultyPerAmount = 0.1f;
    maxHpMultiplierPerAmount = 0.8f;
    baseDamageMultiplier = 0.85f;
    baseRadius = 4.0f;

    // Create reusable damage container
    reuseDc = new DamageContainer(0.0f, damageSource);
}

// OnInitOrAmountChanged logic
protected override void OnInitOrAmountChanged() {
    // Calculate hyperbolic proc chance
    float rawChance = amount * procChancePerAmount;
    procChance = (rawChance / (rawChance + 0.8f)) * 0.5f;

    // Set exponential health reduction
    SetStat(EStat.MaxHealth, pow(maxHpMultiplierPerAmount, amount), StatModifierType.Multiplicative);

    // Set linear difficulty increase
    SetStat(EStat.Difficulty, amount * difficultyPerAmount, StatModifierType.Additive);
}

// ProcOnHitEffects logic
public override void ProcOnHitEffects(DamageContainer dc) {
    if (!ItemUtility.TryProc(dc.procCoefficient, procChance)) return;

    Enemy targetEnemy = dc.enemy;
    Vector3 targetPos = targetEnemy.transform.position;

    // Get all enemies in radius
    Collider[] enemies = WeaponUtility.GetEnemiesInRadius(targetPos, baseRadius);

    foreach (Collider enemy in enemies) {
        if (EnemyManager.GetEnemy(enemy, out Enemy enemyComponent)) {
            float damage = GetDamage();

            // Create damage container for this enemy
            DamageContainer damageContainer = WeaponUtility.GetDamageContainer(
                reuseDc, damage, 0.0f, damageSource, Vector3.zero, enemyComponent);

            // Deal damage
            enemyComponent.DamageFromPlayerOther(damageContainer);
        }
    }

    // Play visual effect at target location
    if (fx == null) {
        GameObject effectPrefab = EffectManager.Instance.cursedGlovesEffect;
        GameObject effectInstance = Object.Instantiate(effectPrefab);
        fx = effectInstance.GetComponent<EffectPlayer>();
    }

    fx.gameObject.SetActive(true);
    fx.transform.position = targetPos;
    fx.Play();
}
```

## Technical Notes
- **Memory Management**: Uses object pooling pattern with reusable DamageContainer
- **Effect Management**: Lazy instantiation of visual effects to reduce overhead
- **Damage Source**: Uses static damage source identifier for consistent attribution
- **Null Safety**: Extensive null checking throughout implementation to prevent crashes
- **Performance**: Efficient area damage implementation using Unity's physics queries

## Related Items
- **Other Cursed Items**: Likely synergizes with other items that increase difficulty for rewards
- **Area Damage Items**: Similar to Bonker, SpicyMeatball in area effect mechanics
- **Risk/Reward Items**: Beer (damage vs HP), CursedDoll (enemy management)
- **Proc-Based Items**: Dragonfire, IceCrystal, LightningOrb use similar proc mechanics

## Balancing Considerations
- **High Risk**: Exponential HP reduction makes stacking dangerous
- **High Reward**: Area damage with reasonable proc chance
- **Difficulty Scaling**: Increases game difficulty, affecting all enemies
- **Diminishing Returns**: Hyperbolic proc chance prevents 100% proc rate
- **Stack Limit Considerations**: No hard limit, but exponential HP loss naturally limits viable stacks

---

**Data Sources:**
- megabonk_research/items.md (lines 517-535)
- extracted_constructors/items/GlovesCursed.c
- decompiled/Assembly-CSharp/.../ItemGlovesCursed.cs
- decompiled/Assembly-CSharp/.../ItemBase.cs