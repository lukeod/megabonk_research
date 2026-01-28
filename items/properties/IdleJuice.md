# IdleJuice

## Overview
- **Item ID**: EItem.IdleJuice (from decompiled C#)
- **Constructor Address**: 0x18045E690
- **Category**: Utility/Damage Enhancement
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| damagePerAmount | float | 1.0 | Base damage per stack |
| damagePerSecond | float | 0.039999999 (~0.04) | Damage accumulation rate per second |
| setupTime | float | 0.60000002 (~0.6) | Time to establish campfire when stationary |
| distThreshold | float | 1.75 | Maximum distance from campfire position |
| updateDamageInterval | float | 1.0 | Update frequency for damage accumulation |
| maxDamage | float | Dynamic | Maximum accumulated damage (calculated) |
| currentDamage | float | Dynamic | Current accumulated damage |
| campfirePos | Vector3 | Dynamic | Position where campfire is established |
| startCampfireTime | float | Dynamic | Time when campfire setup begins |
| isCampActive | bool | Dynamic | Whether campfire system is active |
| nextUpdateDamageTime | float | Dynamic | Next time to update damage accumulation |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 12 | Power/Damage | currentDamage | Dynamic accumulation |

## Special Mechanics

### Campfire System
The IdleJuice item implements a campfire-based idle damage accumulation system similar to the Campfire item:

1. **Position Tracking**: Continuously monitors player position and establishes a "campfire" position
2. **Distance Check**: Player must stay within 1.75 units of the campfire position
3. **Setup Time**: Requires 0.6 seconds of stationary positioning to activate
4. **Damage Accumulation**: While active and under max damage, accumulates 0.04 damage per second
5. **Maximum Cap**: Damage accumulation is capped at `maxDamage` (calculated value)

### State Management
- **Inactive State**: When player moves too far from campfire position, system deactivates and resets damage to 0
- **Setup State**: When player stays in position for setup time, campfire activates
- **Active State**: Damage accumulates every 1 second until max is reached

## Formulas

### Distance Calculation
```
distance = sqrt((campfirePos.x - playerPos.x)² + (campfirePos.y - playerPos.y)² + (campfirePos.z - playerPos.z)²)
```

### Damage Accumulation
```
if (isCampActive && currentDamage < maxDamage && time > nextUpdateDamageTime):
    currentDamage = min(currentDamage + damagePerSecond, maxDamage)
    nextUpdateDamageTime = time + updateDamageInterval
```

### Maximum Damage
```
maxDamage = damagePerAmount * amount
// Where amount is the stack count
```

## Implementation Details
- **Update Frequency**: Tick() method called every frame for real-time position monitoring
- **Event Subscriptions**: None (self-contained system)
- **Stack Behavior**: Linear scaling - each stack adds 1.0 to maximum damage potential
- **Performance**: Uses 1-second intervals for damage updates to optimize performance

## C# Pseudocode
```csharp
// Simplified constructor logic
public ItemIdleJuice(ItemInventory itemInventoryRef)
{
    damagePerAmount = 1.0f;
    damagePerSecond = 0.04f;
    setupTime = 0.6f;
    distThreshold = 1.75f;
    updateDamageInterval = 1.0f;

    base(itemInventoryRef);
}

// Simplified tick logic
public override void Tick()
{
    Vector3 playerPos = MyPlayer.Instance.transform.position;
    float distanceFromCamp = Vector3.Distance(playerPos, campfirePos);

    if (distanceFromCamp > distThreshold) {
        // Player moved away - reset system
        campfirePos = playerPos;
        startCampfireTime = MyTime.time + setupTime;

        if (isCampActive) {
            isCampActive = false;
            SetStat(new StatModifier { stat = 12, operationType = 2, value = 0 });
            currentDamage = 0;
        }
    } else if (!isCampActive && MyTime.time > startCampfireTime) {
        // Setup time completed - activate campfire
        isCampActive = true;
        currentDamage = 0;
        campfirePos = playerPos;
        startCampfireTime = MyTime.time + setupTime;
    }

    // Accumulate damage if active
    if (isCampActive && currentDamage < maxDamage && MyTime.time > nextUpdateDamageTime) {
        currentDamage = Math.Min(currentDamage + damagePerSecond, maxDamage);
        nextUpdateDamageTime = MyTime.time + updateDamageInterval;
        SetStat(new StatModifier { stat = 12, operationType = 2, value = currentDamage });
    }
}
```

## Technical Notes
- Uses Unity's Vector3 for 3D position tracking
- Implements precise distance calculations using sqrt for accuracy
- Time-based system relies on MyTime.time for consistency
- StatModifier uses operationType = 2 (additive) for damage bonuses
- Damage accumulation is clamped to prevent exceeding maximum
- System completely resets when player moves, ensuring no partial accumulation retention

## Related Items
- **Campfire**: Shares similar campfire positioning system but provides health regeneration instead of damage
- **BeefyRing**: Another item with dynamic stat updates based on current state (HP-based damage)
- **GamerGoggles**: Similar real-time stat calculation based on player state

---

*Data sources: megabonk_research/items.md, extracted_constructors/items/IdleJuice.c, decompiled C# ItemIdleJuice.cs*