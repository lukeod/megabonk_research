# GoldenSneakers

## Overview
- **Item ID**: EItem.GoldenSneakers
- **Constructor Address**: 0x180410890
- **Category**: Economy/Movement
- **Rarity**: Common

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| goldPerMeterBase | float | 0.05 | Base gold generation per meter traveled |
| checkInterval | float | 0.5 | How often position is checked (seconds) |
| goldPerMeter | float | Dynamic | Calculated gold per meter (runtime) |
| nextCheckTime | float | Dynamic | Next scheduled position check |
| lastPos | Vector3 | Dynamic | Previous player position |
| accumulatedGold | float | Dynamic | Gold accumulator before spawning |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | N/A | N/A | No direct stat modifications |

## Special Mechanics
- **Gold Generation**: Generates gold based on distance traveled by the player
- **Distance Tracking**: Monitors player movement every 0.5 seconds
- **Gold Accumulation**: Accumulates fractional gold until reaching 1.0, then spawns actual gold pickup
- **Scaling Formula**: goldPerMeter = (amount-1) * 0.025 + 0.05

## Formulas
- **Gold per Meter**: `goldPerMeter = (amount - 1) * 0.025 + goldPerMeterBase`
- **Base Gold Rate**: `0.05 gold per meter`
- **Stack Scaling**: `+0.025 gold per meter per additional stack`
- **Distance Calculation**: Uses Vector3 magnitude calculation between current and last position
- **Gold Spawning**: Spawns `floor(accumulatedGold)` gold when accumulator >= 1.0

## Implementation Details
- **Update Frequency**: Every 0.5 seconds (checkInterval)
- **Event Subscriptions**: None (purely time-based)
- **Stack Behavior**: Linear scaling - each additional stack adds 0.025 gold per meter
- **Gold Spawning**: Uses `MoneyUtility.SpawnMoney()` at player position
- **Position Tracking**: Stores last known player position as Vector3

## C# Pseudocode
```csharp
// Constructor logic
public ItemGoldenSneakers(ItemInventory itemInventoryRef) {
    this.goldPerMeterBase = 0.05f;
    this.checkInterval = 0.5f;
    base(itemInventoryRef);
}

// OnInitOrAmountChanged logic
protected override void OnInitOrAmountChanged() {
    // Calculate scaling gold per meter
    goldPerMeter = (amount - 1) * (goldPerMeterBase * 0.5f) + goldPerMeterBase;
    // Equivalent to: goldPerMeter = (amount - 1) * 0.025f + 0.05f

    // Store initial player position
    lastPos = MyPlayer.Instance.transform.position;
}

// Tick logic
public override void Tick() {
    if (MyTime.time >= nextCheckTime) {
        nextCheckTime = MyTime.time + checkInterval;

        Vector3 currentPos = MyPlayer.Instance.transform.position;
        Vector3 deltaPos = currentPos - lastPos;
        float distance = deltaPos.magnitude;

        // Accumulate gold based on distance traveled
        accumulatedGold += distance * goldPerMeter;

        // Spawn gold when we have at least 1 unit
        if (accumulatedGold >= 1.0f) {
            int goldToSpawn = (int)Math.Floor(accumulatedGold);
            accumulatedGold -= goldToSpawn;

            MoneyUtility.SpawnMoney(goldToSpawn, currentPos);
        }

        // Update position for next check
        lastPos = currentPos;
    }
}
```

## Technical Notes
- **Performance**: Position checks limited to 2Hz to reduce CPU overhead
- **Precision**: Uses float accumulation to handle fractional gold generation
- **Movement Detection**: Only generates gold when player actually moves between checks
- **Gold Spawning**: Spawns physical gold pickups at player location rather than directly adding to inventory
- **Stack Efficiency**: Each additional stack provides 50% increase in base generation rate

## Related Items
- **CreditCardGreen**: Another economy item that scales with chest openings
- **CreditCardRed**: Damage-focused economy item
- **GoldenShield**: Defensive economy item that generates gold on taking damage

## Example Scaling
| Stacks | Gold per Meter | Gold per 100m | Notes |
|--------|----------------|---------------|-------|
| 1 | 0.050 | 5.0 | Base rate |
| 2 | 0.075 | 7.5 | +50% increase |
| 3 | 0.100 | 10.0 | +100% increase |
| 4 | 0.125 | 12.5 | +150% increase |
| 5 | 0.150 | 15.0 | +200% increase |

---

*Data extracted from IL2CPP constructor at 0x180410890 and decompiled C# sources*