# EnergyCore

## Overview
- **Item ID**: EItem.EnergyCore (Value: 44)
- **Constructor Address**: 0x180409D50
- **Category**: Active/Projectile
- **Rarity**: Common/Uncommon (determinable from game context)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| orbsPerAmount | int | 2 | Orbs added per stack |
| numOrbs | int | 4 | Base number of orbs |
| cooldown | float | 4.0 | Seconds between volleys |
| cooldownPerOrb | float | 0.2 | Base time between individual orbs |
| nextSpawnTime | float | Runtime | Next volley spawn time |
| orbsLeftToShoot | int | Runtime | Orbs remaining in current volley |
| nextOrbTime | float | Runtime | Next individual orb spawn time |
| damageSource | string | "EnergyCore" | Damage attribution string |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | N/A | N/A | No direct stat modifications |

## Special Mechanics
- **Periodic Orb Volleys**: Fires volleys of energy orbs at regular intervals
- **Dynamic Orb Count**: Total orbs scales with item stacks
- **Staggered Firing**: Individual orbs fire with delays between them
- **No Stat Dependencies**: Operates independently of player stats

## Formulas

### Orb Count Calculation
```
numOrbs = (amount * orbsPerAmount) + 5
numOrbs = (amount * 2) + 5
```

### Cooldown Per Orb Calculation
```
totalOrbTime = min(numOrbs * 0.3, 1.5)
cooldownPerOrb = totalOrbTime / numOrbs
```

### Timing Sequence
1. Wait for main cooldown (4.0 seconds)
2. Fire orbs with calculated intervals between each
3. Reset and repeat

## Implementation Details
- **Update Frequency**: Every frame via Tick() method
- **Event Subscriptions**: None (purely time-based)
- **Stack Behavior**: Linear scaling for orb count, diminishing returns on firing speed

## C# Pseudocode
```csharp
// Constructor logic
orbsPerAmount = 2;
numOrbs = 4;
cooldown = 4.0f;
cooldownPerOrb = 0.2f;
damageSource = "EnergyCore";

// OnInitOrAmountChanged logic
numOrbs = amount * orbsPerAmount + 5;
float totalOrbTime = Mathf.Min(numOrbs * 0.3f, 1.5f);
cooldownPerOrb = totalOrbTime / numOrbs;

// Tick logic
if (orbsLeftToShoot <= 0) {
    if (MyTime.time >= nextSpawnTime) {
        orbsLeftToShoot = numOrbs;
        nextSpawnTime = MyTime.time + cooldown;
        nextOrbTime = MyTime.time;
    }
}

if (orbsLeftToShoot > 0 && MyTime.time >= nextOrbTime) {
    int orbIndex = numOrbs - orbsLeftToShoot;
    FireOrb(orbIndex);
    orbsLeftToShoot--;
    nextOrbTime = MyTime.time + cooldownPerOrb;
}
```

## Technical Notes
- **Performance**: Efficient time-based implementation with minimal computation
- **Scalability**: Good performance even with high stack counts due to optimized timing system
- **Orb Firing**: Uses index-based firing for consistent orb patterns
- **Time Management**: Utilizes game's MyTime system for consistent timing across frame rates

## Stack Analysis
| Stacks | Total Orbs | Orb Interval | Volley Duration |
|--------|------------|--------------|-----------------|
| 1 | 7 | ~0.214s | ~1.5s |
| 2 | 9 | ~0.167s | ~1.5s |
| 3 | 11 | ~0.136s | ~1.5s |
| 5 | 15 | ~0.100s | ~1.5s |
| 10 | 25 | ~0.060s | ~1.5s |

## Related Items
- **SoulHarvester**: Similar projectile spawning mechanics
- **SluttyCannon**: Comparable proc-based projectile system
- **BobDead**: Movement-triggered projectile generation

## Damage Scaling
- Damage is handled by the FireOrb method (implementation in IL2CPP native code)
- Likely scales with player's base damage and power stats
- No built-in damage modifiers in the item itself

---
*Data extracted from IL2CPP decompiled constructor at 0x180409D50 and C# class definition*