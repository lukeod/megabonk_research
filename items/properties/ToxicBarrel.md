# ToxicBarrel

## Overview
- **Item ID**: 53 (EItem.ToxicBarrel)
- **Constructor Address**: 0x180468420
- **Category**: Elemental/Status (Poison-based area denial)
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| radiusPerAmount | float | 1.0 | Radius increase per stack |
| poisonStacksPerAmount | int | 5 | Poison stacks applied per item stack |
| cooldown | float | 0.25 | Seconds between activations |
| poisonDuration | float | 5.0 | Duration of poison effect in seconds |
| damageSource | string | "ToxicBarrel" | Damage attribution |

## Stat Modifiers
This item does not directly modify any EStat values.

## Special Mechanics
- **Reactive Defense**: Triggers when player takes damage
- **Area Poison**: Creates poison field around player position on activation
- **Event-Driven**: Subscribes to PlayerHealth.A_TakeDamage event
- **Cooldown System**: 0.25 second cooldown between activations to prevent spam

### Poison Application
- **Status Effect**: Poison (EStatusEffect.Poison = 13)
- **Stacks Applied**: `amount * poisonStacksPerAmount` (5 per item stack)
- **Duration**: 5.0 seconds
- **Area of Effect**: Circular area around player

### Radius Calculation
```
final_radius = (amount * radiusPerAmount) + 7.0
```
- Base radius: 7.0 units
- Per stack: +1.0 units
- With 3 stacks: 10.0 units radius

## Formulas
| Calculation | Formula | Example (3 stacks) |
|-------------|---------|-------------------|
| Poison Stacks | `amount * 5` | 15 stacks |
| Effect Radius | `amount * 1.0 + 7.0` | 10.0 units |
| Poison Duration | Fixed | 5.0 seconds |
| Activation Cooldown | Fixed | 0.25 seconds |

## Implementation Details
- **Update Frequency**: Event-driven (on player damage)
- **Event Subscriptions**: PlayerHealth.A_TakeDamage
- **Stack Behavior**: Linear scaling for both radius and poison intensity
- **Damage Container**: Uses 0.5 base damage multiplier with "ToxicBarrel" source

### Constructor Logic
```csharp
// Base property initialization
radiusPerAmount = 1.0f;
poisonStacksPerAmount = 5;
cooldown = 0.25f;
poisonDuration = 5.0f;
damageSource = "ToxicBarrel";

// Damage container creation
dc = new DamageContainer(0.5f, "ToxicBarrel");
```

### Runtime Calculations
```csharp
// OnInitOrAmountChanged
poisonStacks = amount * poisonStacksPerAmount;
radius = (amount * radiusPerAmount) + 7.0f;

// On activation (when player takes damage)
if (Time.time >= readyAtTime) {
    // Apply poison in radius around player
    // Reset cooldown: readyAtTime = Time.time + cooldown
}
```

## C# Pseudocode
```csharp
public class ItemToxicBarrel : ItemBase
{
    private float radiusPerAmount = 1.0f;
    private int poisonStacksPerAmount = 5;
    private float cooldown = 0.25f;
    private float poisonDuration = 5.0f;
    private float readyAtTime;
    private DamageContainer dc;

    public override void Init()
    {
        // Subscribe to player damage events
        PlayerHealth.A_TakeDamage += OnTakeDamage;
    }

    public override void Cleanup()
    {
        // Unsubscribe from events
        PlayerHealth.A_TakeDamage -= OnTakeDamage;
    }

    protected override void OnInitOrAmountChanged()
    {
        poisonStacks = amount * poisonStacksPerAmount;
        radius = (amount * radiusPerAmount) + 7.0f;
    }

    private void OnTakeDamage(PlayerHealth ph, DamageContainer dc, bool shieldDamage)
    {
        if (Time.time >= readyAtTime)
        {
            Activate();
        }
    }

    private void Activate()
    {
        // Create poison field around player
        // Apply poison stacks to enemies in radius
        // Duration: poisonDuration (5.0 seconds)
        // Reset cooldown
        readyAtTime = Time.time + cooldown;
    }
}
```

## Technical Notes
- **Memory Management**: Uses IL2CPP garbage collection barriers for object references
- **Performance**: Short cooldown (0.25s) prevents excessive poison field creation
- **Event System**: Properly subscribes/unsubscribes to prevent memory leaks
- **Damage Attribution**: All poison damage is attributed to "ToxicBarrel" source
- **Base Damage**: Uses 0.5 multiplier in damage container, suggesting poison damage is calculated elsewhere

## Related Items
- **MoldyCheese**: Also applies poison (40% chance per stack)
- **GlovesPoison**: Area poison effect with different trigger mechanism
- **UnstableTransfusion**: Applies bleeding instead of poison
- **Gasmask**: Benefits from poisoned enemies (armor/overheal scaling)

## Synergies
- **Defensive Strategy**: Excellent for players who take frequent damage
- **Area Control**: Large radius makes it effective against swarm enemies
- **Poison Builds**: Synergizes with other poison-applying items
- **Tank Builds**: Reactive nature suits high-health, damage-absorbing playstyles

---

*Data extracted from decompiled IL2CPP constructor at 0x180468420 and C# interface analysis*