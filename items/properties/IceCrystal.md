# IceCrystal

## Overview
- **Item ID**: EItem.IceCrystal
- **Constructor Address**: 0x18045D880
- **Category**: Elemental/Status Effect
- **Rarity**: Common

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| procChancePerAmount | float | 0.075 | 7.5% proc chance per stack |
| freezeTime | float | Calculated | Duration based on stack count |
| procChance | float | Calculated | Final proc chance with hyperbolic scaling |

## Stat Modifiers
None - IceCrystal does not modify any EStat values directly.

## Special Mechanics
- **Freeze on Hit**: Triggers freeze debuff (ID 2) when proc occurs
- **Hyperbolic Scaling**: Uses diminishing returns formula to prevent 100% proc rates
- **Stack-Based Duration**: Freeze duration increases with item count

## Formulas

### Proc Chance Calculation
```
finalProcChance = (amount * procChancePerAmount) / ((amount * procChancePerAmount) + 0.75)
```
Where:
- `amount` = number of stacks
- `procChancePerAmount` = 0.075 (7.5% per stack)
- Scaling factor = 0.75

### Freeze Duration Calculation
```
freezeTime = clamp(amount * 0.1 + 2.5, 0, 6)
```
Where:
- Base duration = 2.5 seconds
- Duration per stack = 0.1 seconds
- Maximum duration = 6.0 seconds
- Minimum duration = 0.0 seconds (though realistically starts at 2.6s with 1 stack)

## Implementation Details
- **Update Frequency**: Triggered on hit events via `ProcOnHitEffects`
- **Event Subscriptions**: Responds to damage dealing events
- **Stack Behavior**: Linear duration scaling, hyperbolic proc chance scaling

## C# Pseudocode
```csharp
// Constructor logic
public ItemIceCrystal(ItemInventory itemInventoryRef) : base(itemInventoryRef)
{
    procChancePerAmount = 0.075f;
}

// Called when amount changes
protected override void OnInitOrAmountChanged()
{
    // Calculate hyperbolic scaling for proc chance
    float baseChance = amount * procChancePerAmount;
    procChance = baseChance / (baseChance + 0.75f);

    // Calculate freeze duration with clamping
    float duration = amount * 0.1f + 2.5f;
    freezeTime = Mathf.Clamp(duration, 0f, 6f);
}

// Proc logic on hit
public override void ProcOnHitEffects(DamageContainer dc)
{
    if (dc?.enemy == null) return;

    // Check if proc triggers
    if (ItemUtility.TryProc(dc.procCoefficient, procChance))
    {
        // Apply freeze debuff (ID 2)
        dc.enemy.AddDebuff(2, dc, freezeTime, 1);
    }
}
```

## Technical Notes
- Uses `ItemUtility.TryProc()` for chance calculations, which factors in proc coefficient
- Debuff ID 2 corresponds to Freeze status effect
- The hyperbolic scaling factor 0.75 is consistent with other items using similar mechanics
- Duration scaling is linear but capped to prevent excessively long freezes

## Related Items
- **IceCube**: Similar freeze mechanics but with projectile spawning
- **LightningOrb**: Uses similar hyperbolic scaling for stun effects
- **Dragonfire**: Comparable proc-based debuff system with burn effects

---

**Data Sources**:
- Constructor: `extracted_constructors/items/IceCrystal.c`
- Decompiled C#: `decompiled/Assembly-CSharp/.../ItemIceCrystal.cs`
- Item Summary: `megabonk_research/items.md`