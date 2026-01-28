# CowardsCloak

## Overview
- **Item ID**: Unknown
- **Constructor Address**: 0x18043ED00
- **Category**: Movement/Speed - Defensive Mobility
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| speedPerAmount | float | 0.05 | Base movement speed per item stack |
| speedPerStack | float | 0.3 | Movement speed per temporary coward stack |
| maxStacks | int | 2 | Maximum temporary stacks allowed |
| stacksPerAmount | int | 2 | Temporary stacks gained per item stack |
| stacksResetAtTime | float | Dynamic | Timestamp when temporary stacks expire |
| stacks | int | Dynamic | Current temporary stacks |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 25 | Movement Speed | speedPerAmount * amount + (speedPerStack * stacks) | Base + Temporary |

## Special Mechanics
- **Damage Trigger**: Subscribes to PlayerHealth.A_TakeDamage event
- **Temporary Stacks**: Gains stacks when taking damage, providing burst movement speed
- **Stack Decay**: Temporary stacks automatically decay over time
- **Dynamic Refresh**: Stats are refreshed when stacks are gained or lost

## Formulas
- **Base Speed Bonus**: `amount * 0.05` (5% per item stack)
- **Temporary Speed Bonus**: `stacks * 0.3` (30% per temporary stack)
- **Total Speed Bonus**: `(amount * 0.05) + (stacks * 0.3)`
- **Max Temporary Stacks**: `amount * 2` (2 stacks per item)
- **Stack Duration**: Based on `stacksResetAtTime` timer value

## Implementation Details
- **Update Frequency**: Per-frame via Tick() method
- **Event Subscriptions**: PlayerHealth.A_TakeDamage (OnDamage method)
- **Stack Behavior**: Temporary stacks reset when timer expires

## C# Pseudocode
```csharp
// Constructor logic
speedPerAmount = 0.05f;
speedPerStack = 0.3f;
maxStacks = 2;
stacksPerAmount = 2;

// On damage taken
void OnDamage(PlayerHealth ph, DamageContainer dc, bool shieldDamage) {
    AddTemporaryStack();
}

// Add temporary stack
void AddTemporaryStack() {
    if (stacks < maxStacks * amount) {
        stacks = Math.Min(stacks + stacksPerAmount, maxStacks * amount);
        stacksResetAtTime = MyTime.time + baseDuration;
        RefreshStats();
    }
}

// Tick behavior
void Tick() {
    if (stacks > 0 && MyTime.time > stacksResetAtTime) {
        stacks = 0;
        RefreshStats();
    }
}

// Stats calculation
void RefreshStats() {
    float baseSpeed = amount * speedPerAmount;        // 5% per item
    float tempSpeed = stacks * speedPerStack;         // 30% per stack
    SetStat(EStat.MovementSpeed, baseSpeed + tempSpeed);
}
```

## Technical Notes
- Uses Unity's event system for damage detection
- Implements both persistent and temporary stat bonuses
- Stack timing managed via global time system
- Stats automatically refresh on stack changes
- Memory-efficient event subscription/unsubscription in Init/Cleanup

## Related Items
- **TurboSocks**: Provides consistent movement speed (+15% per stack)
- **PhantomShroud**: Evasion-based speed boost on successful dodge
- **FlappyFeathers**: Jump-triggered speed boost
- **Rollerblades**: Attack speed scaling with movement speed

## Synergies
- **Defensive Items**: Triggers more frequently with items that increase survival
- **Health Items**: Higher health pool = more damage events = more speed triggers
- **Evasion Items**: Complements dodge-based mobility strategies
- **Combat Items**: Speed boost helps with hit-and-run tactics

---

*Data extracted from Assembly-CSharp.dll decompilation and IL2CPP constructor analysis*
*Constructor address: 0x18043ED00*
*Analysis based on MegaBonk reverse engineering project*