# Mirror

## Overview
- **Item ID**: 48 (Mirror)
- **Constructor Address**: 0x180416A80
- **Category**: Defensive - Damage Reflection
- **Rarity**: Unknown (determinable from game data)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| cooldown | float | 8.0 | Base cooldown between reflections |
| minCooldown | float | 4.0 | Minimum cooldown (hard limit) |
| damageMultiplier | float | 1.0 | Base damage multiplier (dynamic) |
| damagePerAmount | float | 0.25 | Damage multiplier increase per stack |
| canReflect | bool | true | Whether reflection is currently available |
| lastReflectedTime | float | 0.0 | Timestamp of last reflection |
| damageSource | string | "Mirror" | Damage source identifier |
| reuseDc | DamageContainer | Created | Reusable damage container for reflections |

## Stat Modifiers
This item does not directly modify any EStat values. All effects are implemented through special mechanics.

## Special Mechanics
- **Damage Reflection**: Reflects incoming damage back to attackers when available
- **Cooldown-based**: Cannot reflect damage while on cooldown
- **Scaling Damage**: Reflected damage increases with stack count
- **Event-driven**: Subscribes to PlayerHealth damage events (A_CheckStopDamage)
- **Ready State Notification**: Fires A_MirrorReady event when reflection becomes available

## Formulas
| Formula | Description |
|---------|-------------|
| `cooldown = max(8.0 - amount, 4.0)` | Cooldown reduction with minimum cap |
| `damageMultiplier = (amount * 0.25) + 1.0` | Reflected damage scaling |
| `canReflect = (currentTime > lastReflectedTime + cooldown)` | Availability check |

## Implementation Details
- **Update Frequency**: Checked every game tick via Tick() method
- **Event Subscriptions**:
  - `PlayerHealth.A_CheckStopDamage` (subscribed in Init, unsubscribed in Cleanup)
  - `ItemMirror.A_MirrorReady` (static event for UI/feedback)
- **Stack Behavior**:
  - Reduces cooldown (minimum 4 seconds)
  - Increases reflected damage multiplier linearly
  - All stacks share the same cooldown timer

## C# Pseudocode
```csharp
// Simplified constructor logic
public ItemMirror(ItemInventory itemInventoryRef) {
    // Base properties
    cooldown = 8.0f;
    minCooldown = 4.0f;
    damageMultiplier = 1.0f;
    damagePerAmount = 0.25f;
    damageSource = "Mirror";

    // Create reusable damage container
    reuseDc = new DamageContainer(1.0f, damageSource);

    // Initialize base class
    base(itemInventoryRef);
}

// Update scaling values when amount changes
protected override void OnInitOrAmountChanged() {
    cooldown = Math.Max(8.0f - amount, minCooldown);
    damageMultiplier = (amount * damagePerAmount) + 1.0f;

    // Notify that mirror is ready
    A_MirrorReady?.Invoke(true);
}

// Check if reflection is available each tick
public override void Tick() {
    if (!canReflect && MyTime.time > lastReflectedTime + cooldown) {
        canReflect = true;
        A_MirrorReady?.Invoke(true);
    }
}

// Handle damage events from PlayerHealth
private void OnCheckStopDamage(DamageContainer dc, bool shieldDamage) {
    if (canReflect && ReflectDamage(dc)) {
        canReflect = false;
        lastReflectedTime = MyTime.time;
        A_MirrorReady?.Invoke(false);
    }
}
```

## Technical Notes
- Uses a reusable DamageContainer to avoid allocation overhead during reflections
- The reflection system is tied to the PlayerHealth component's damage processing pipeline
- Static A_MirrorReady event allows UI elements to show mirror availability state
- Event subscription/unsubscription is properly handled in Init/Cleanup to prevent memory leaks
- The item uses EItem enum value 48 for its damage source string generation

## Related Items
- **SpikyShield**: Another defensive item that deals retaliation damage
- **QuinsMask**: Provides thorns damage which is similar to damage reflection
- **PhantomShroud**: Defensive item with evasion mechanics

---

*Data extracted from decompiled IL2CPP constructors via IDA Pro MCP and Assembly-CSharp decompilation*