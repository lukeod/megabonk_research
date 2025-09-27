# FlappyFeathers

## Overview
- **Item ID**: EItem.FlappyFeathers
- **Constructor Address**: 0x18040BDF0
- **Category**: Movement/Speed Enhancement
- **Rarity**: Unknown (not determinable from code)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| speedBoostPerAmount | float | 1.8 | Speed boost multiplier per stack |
| jumpHeightAdditionPerAmount | float | 0.15 | Jump height increase per stack |
| speedBoost | float | Dynamic | Calculated value: amount * speedBoostPerAmount |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 26 | Jump Height | amount * 0.15 | Linear |

## Special Mechanics
FlappyFeathers provides a temporary speed boost when the player jumps. The item subscribes to the PlayerMovement jump event system and applies a calculated speed boost when the OnJumped callback is triggered.

### Key Features:
- **Event-Driven**: Responds to player jump actions via Unity's event system
- **Speed Boost**: Provides temporary movement speed increase on jump
- **Permanent Jump Height**: Permanently increases jump height through EStat modification
- **Stack Scaling**: Both speed boost and jump height scale linearly with item count

## Formulas
- **Jump Height Bonus**: `amount * 0.15`
- **Speed Boost on Jump**: `amount * 1.8`

## Implementation Details
- **Update Frequency**: Event-driven (no regular ticking)
- **Event Subscriptions**: PlayerMovement.A_Jumped (static event)
- **Stack Behavior**: Linear scaling for both speed boost and jump height

### Event Lifecycle:
1. **Init()**: Subscribes to PlayerMovement.A_Jumped event
2. **OnJumped()**: Applies temporary speed boost when event fires
3. **Cleanup()**: Unsubscribes from PlayerMovement.A_Jumped event

## C# Pseudocode
```csharp
// Constructor logic
public ItemFlappyFeathers(ItemInventory itemInventoryRef) {
    this.speedBoostPerAmount = 1.8f;
    this.jumpHeightAdditionPerAmount = 0.15f;
    base(itemInventoryRef);
}

// Stat initialization
protected override void OnInitOrAmountChanged() {
    // Set permanent jump height bonus
    SetStat(new StatModifier {
        statType = EStat.JumpHeight, // ID: 26
        value = amount * jumpHeightAdditionPerAmount
    });

    // Calculate speed boost for jump events
    speedBoost = amount * speedBoostPerAmount;
}

// Event subscription
public override void Init() {
    PlayerMovement.A_Jumped += OnJumped;
}

// Speed boost application
private void OnJumped(PlayerMovement pm) {
    // Apply temporary speed boost (implementation in native code)
    // Uses calculated speedBoost value
}

// Event cleanup
public override void Cleanup() {
    PlayerMovement.A_Jumped -= OnJumped;
}
```

## Technical Notes
- The IL2CPP decompiled C# shows event subscription/unsubscription patterns but actual speed boost implementation is in native code
- Speed boost appears to be temporary (applied on jump, likely with duration/decay)
- Jump height modification is permanent through the stat system
- Event system uses Unity's Action delegates for decoupled architecture
- The item does not override Tick(), PreAttack(), or ProcOnHitEffects() - purely event-driven

## Related Items
**Movement/Speed Category Items**:
- **CowardsCloak**: Speed boost on taking damage with stack mechanics
- **GoldenSneakers**: Movement-based gold generation
- **Rollerblades**: Attack speed scaling with movement speed
- **TurboSocks**: Direct movement speed stat bonus
- **PhantomShroud**: Speed boost on evasion with timeout mechanics

## Performance Considerations
- Minimal performance impact as it's event-driven rather than tick-based
- Event subscription/unsubscription handled properly in Init/Cleanup
- No continuous calculations or updates required

---
*Data extracted from decompiled IL2CPP constructors and C# interop code via IDA Pro MCP*