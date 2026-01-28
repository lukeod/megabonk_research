# Dragonfire

## Overview
- **Item ID**: EItem.Dragonfire (ID: 22)
- **Constructor Address**: 0x180454F40
- **Category**: Elemental/Proc-based Damage Item
- **Rarity**: Unknown (determinable from game data)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| procChancePerAmount | float | 0.15 | 15% proc chance per stack (before hyperbolic scaling) |
| burnChancePerAmount | float | 0.15 | 15% burn chance per stack (before hyperbolic scaling) |
| damageSource | string | "Dragonfire" | Damage attribution string (EItem enum value 22) |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | - | - | No direct stat modifications |

## Special Mechanics

### Proc System
Dragonfire operates on a dual-proc system with hyperbolic scaling:

1. **Firefield Creation**: Creates firefields at enemy positions when main proc triggers
2. **Burn Application**: Applies burn debuff to enemies hit by fire-element damage

### Hyperbolic Scaling
Both proc chances use hyperbolic scaling to prevent reaching 100%:
- **Formula**: `chance = (amount × baseChance) / ((amount × baseChance) + 0.5)`
- **Scaling Factor**: 0.5 (prevents saturation)

### Fire Element Requirement
Burn application only triggers when:
- Damage container element == 3 (Fire element)
- Burn proc succeeds

### Firefield Mechanics
When firefield proc succeeds:
- Creates pooled GameObject with Firefield component
- Positions based on enemy center position with ground raycast
- Adds random offset within 0.5 unit radius
- Scales with player stats (EStat 9 and 10)

## Formulas

### Proc Chance Calculation
```
procChance = (amount × 0.15) / ((amount × 0.15) + 0.5)
burnChance = (amount × 0.15) / ((amount × 0.15) + 0.5)
```

### Firefield Scaling
```
radius = clamp(EStat9 × 6.0, 1.0, 12.0)
lifetime = clamp(EStat10 × 2.0, 1.0, 3.0)
```

### Burn Duration
```
burnDuration = 3.0 seconds (constant)
```

## Implementation Details
- **Update Frequency**: On hit (event-driven)
- **Event Subscriptions**: ProcOnHitEffects override
- **Stack Behavior**: Hyperbolic scaling prevents linear stacking
- **Element Check**: Only burns on fire damage (element ID 3)
- **Object Pooling**: Uses PoolManager for firefield GameObjects

## C# Pseudocode
```csharp
// Constructor logic
public ItemDragonfire(ItemInventory itemInventoryRef) : base(itemInventoryRef)
{
    procChancePerAmount = 0.15f;
    burnChancePerAmount = 0.15f;
    damageSource = EItem.Dragonfire.ToString(); // "Dragonfire"
}

// Amount change handler
protected override void OnInitOrAmountChanged()
{
    // Hyperbolic scaling with factor 0.5
    procChance = (amount * procChancePerAmount) / ((amount * procChancePerAmount) + 0.5f);
    burnChance = (amount * burnChancePerAmount) / ((amount * burnChancePerAmount) + 0.5f);
}

// Main proc logic
public override void ProcOnHitEffects(DamageContainer dc)
{
    // Burn application (fire damage only)
    if (dc.element == 3 && TryProc(dc.procCoefficient, burnChance))
    {
        dc.enemy.AddDebuff(DebuffType.Burn, dc, 3.0f, 1, false);
    }

    // Firefield creation
    if (TryProc(dc.procCoefficient, procChance))
    {
        CreateFirefield(dc.enemy);
    }
}

private void CreateFirefield(Enemy enemy)
{
    GameObject fireFieldObj = PoolManager.Instance.fireFieldPool.Get();

    // Position at enemy location with ground raycast
    Vector3 enemyPos = enemy.GetCenterPosition();
    Vector3 groundPos = RaycastUtility.RayToGround(enemyPos, 9999f);

    // Add random offset
    Vector3 randomOffset = Random.insideUnitSphere.XZVector() * 0.5f;
    Vector3 finalPos = groundPos + randomOffset;

    fireFieldObj.transform.position = finalPos;

    // Scale with player stats
    float radius = Mathf.Clamp(PlayerStats.GetStat(EStat.AreaMultiplier) * 6f, 1f, 12f);
    float lifetime = Mathf.Clamp(PlayerStats.GetStat(EStat.Unknown10) * 2f, 1f, 3f);

    // Configure firefield
    Firefield firefield = fireFieldObj.GetComponent<Firefield>();
    firefield.Set(finalPos, enemy.GetFeetPosition(), radius, lifetime,
                 dc.damage, 0, damageSource);
}
```

## Technical Notes

### Memory Management
- Uses object pooling for firefields to prevent garbage collection spikes
- Properly handles IL2CPP interop for field access
- Damage source string is managed through garbage collection barriers

### Performance Considerations
- Hyperbolic scaling prevents excessive proc rates that could cause performance issues
- Ground raycast limited to reasonable distance (9999 units)
- Clamped radius and lifetime values prevent extreme scaling

### Element System Integration
- Integrates with game's element system (fire = 3)
- Burn debuff uses standard debuff system (type 4)
- Damage attribution through damageSource field

## Related Items
- **IceCrystal**: Similar proc-based elemental item with freeze
- **LightningOrb**: Another elemental proc item with stun
- **MoldyCheese**: Poison-based proc item with similar mechanics
- **BloodyCleaver**: Bleed-based proc item with lifesteal trigger

## Synergies
- **High proc coefficient weapons**: Amplifies both burn and firefield procs
- **Fire damage items**: Required for burn application
- **Area multiplier items**: Increases firefield radius
- **EStat 10 items**: Increases firefield lifetime

---

*Data extracted from IL2CPP decompilation at constructor address 0x180454F40*
*Last updated: Analysis of MegaBonk decompiled source*