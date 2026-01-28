# BobLantern

## Overview
- **Item ID**: EItem.BobLantern
- **Constructor Address**: 0x18043B5B0
- **Category**: Area of Effect / Timed Explosion
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| cooldownMin | float | 5.0 | Minimum cooldown between explosions |
| cooldownMax | float | 45.0 | Maximum/starting cooldown |
| cooldownReductionPerAmount | float | 3.0 | Cooldown reduction per stack |
| radiusMin | float | 50.0 | Minimum explosion radius |
| radiusMax | float | 250.0 | Maximum explosion radius |
| radiusPerAmount | float | 10.0 | Radius increase per stack |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | No direct stat modifiers | N/A | N/A |

## Special Mechanics
- **Timed Explosions**: Automatically explodes on a timer (not proc-based)
- **Initial Delay**: 2.0 seconds after init before first explosion
- **Cooldown Scaling**: Cooldown decreases with more stacks (clamped to cooldownMin)
- **Radius Scaling**: Explosion radius increases with more stacks (clamped to radiusMax)
- **Tick-Based**: Checks time every game tick to determine when to explode

## Formulas

### Cooldown Calculation
```
rawCooldown = cooldownMax - (amount * cooldownReductionPerAmount)
rawCooldown = 45.0 - (amount * 3.0)
cooldown = clamp(rawCooldown, cooldownMin, cooldownMax)
cooldown = clamp(rawCooldown, 5.0, 45.0)
```

### Radius Calculation
```
rawRadius = radiusMin + (amount * radiusPerAmount)
rawRadius = 50.0 + (amount * 10.0)
radius = clamp(rawRadius, radiusMin, radiusMax)
radius = clamp(rawRadius, 50.0, 250.0)
```

### Next Explosion Time
```
nextExplodeTime = currentTime + 2.0  // On init/amount change
```

## Implementation Details
- **Update Frequency**: Tick-based (checked every frame)
- **Event Subscriptions**: None (uses Tick method)
- **Stack Behavior**:
  - Cooldown decreases linearly (faster explosions)
  - Radius increases linearly (larger explosions)
  - Both are clamped to min/max values
- **Explosion Trigger**: When `currentTime >= nextExplodeTime`, calls Explode method

## C# Pseudocode
```csharp
// Constructor logic
public ItemBobLantern(ItemInventory itemInventoryRef) : base(itemInventoryRef, 0)
{
    this.cooldownMin = 5.0f;
    this.cooldownMax = 45.0f;
    this.cooldownReductionPerAmount = 3.0f;
    this.radiusMin = 50.0f;
    this.radiusMax = 250.0f;
    this.radiusPerAmount = 10.0f;
}

// Amount change recalculation
protected override void OnInitOrAmountChanged()
{
    // Calculate cooldown (decreases with stacks)
    float rawCooldown = cooldownMax - (amount * cooldownReductionPerAmount);
    cooldown = Mathf.Clamp(rawCooldown, cooldownMin, cooldownMax);

    // Set next explosion time (2 second delay)
    nextExplodeTime = MyTime.time + 2.0f;

    // Calculate radius (increases with stacks)
    float rawRadius = radiusMin + (amount * radiusPerAmount);
    radius = Mathf.Clamp(rawRadius, radiusMin, radiusMax);

    // Update visual size
    RefreshExplosionSize();
}

// Tick method - called every frame
public override void Tick()
{
    if (MyTime.time >= nextExplodeTime)
    {
        Explode();
    }
}
```

## Technical Notes
- Uses `MyTime.time` for game time (handles pause states)
- Explosion size is visually updated via `RefreshExplosionSize()` method
- Cooldown and radius are recalculated whenever stack count changes
- The 2.0 second initial delay prevents immediate explosion on pickup
- Min/max clamping prevents extreme values at high or low stack counts

## Scaling Analysis

### Single Stack (Amount = 1)
- Cooldown: 42.0 seconds (45.0 - 1 * 3.0)
- Radius: 60.0 (50.0 + 1 * 10.0)

### Five Stacks (Amount = 5)
- Cooldown: 30.0 seconds (45.0 - 5 * 3.0)
- Radius: 100.0 (50.0 + 5 * 10.0)

### Ten Stacks (Amount = 10)
- Cooldown: 15.0 seconds (45.0 - 10 * 3.0)
- Radius: 150.0 (50.0 + 10 * 10.0)

### Maximum Efficiency
- Cooldown reaches minimum (5.0s) at: 14+ stacks (45.0 - 14 * 3.0 = 3.0, clamped to 5.0)
- Radius reaches maximum (250.0) at: 20 stacks (50.0 + 20 * 10.0 = 250.0)

## Related Items
- **Campfire**: Similar area-based passive effect
- **ToxicBarrel**: Area damage on timer
- **LightningOrb**: Periodic area damage mechanic
- **BobDead**: Related "Bob" item family

---

*Data extracted from decompiled IL2CPP constructors via IDA Pro MCP and C# decompilation*
