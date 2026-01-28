# CreditCardRed

## Overview
- **Item ID**: EItem.CreditCardRed (ID: 63)
- **Constructor Address**: 0x18043FD80
- **Category**: Economy/Damage
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| damagePerChestAmount | float | 0.025 | Base damage per chest opened per stack |
| damagePerChest | float | Calculated | Derived from damagePerChestAmount * amount |
| accumulatedDamage | float | Dynamic | Running total of damage bonus |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 12 | Power/Damage | accumulatedDamage | Dynamic accumulation |

## Special Mechanics
- **Event-Driven**: Responds to chest opening events through ChestWindowUi.A_Open
- **Accumulative**: Each chest opened permanently increases damage by `damagePerChest`
- **Persistent**: Damage accumulation persists throughout the run
- **Stack Scaling**: Each stack increases damage per chest by 0.025 (2.5%)

## Formulas
```
damagePerChest = damagePerChestAmount * amount
accumulatedDamage += damagePerChest (on each chest opened)
Final Power Bonus = accumulatedDamage
```

## Implementation Details
- **Update Frequency**: Triggered on chest opening events
- **Event Subscriptions**:
  - Init: Subscribes to `ChestWindowUi.A_Open` event
  - Cleanup: Unsubscribes from `ChestWindowUi.A_Open` event
- **Stack Behavior**: Linear scaling - each stack adds 2.5% damage per chest
- **Moving Stat**: Uses `StatInventory.ChangeMovingStat()` with source name from EItem enum

## C# Pseudocode
```csharp
public class ItemCreditCardRed : ItemBase
{
    public float damagePerChestAmount = 0.025f;
    public float damagePerChest;
    public float accumulatedDamage;

    public override void Init()
    {
        // Subscribe to both chest opening events
        InteractableChest.A_ChestOpened += OnChestWindowOpen;
        OpenChest.A_Open += OnChestWindowOpen;
    }

    public override void Cleanup()
    {
        // Unsubscribe from chest opening events
        InteractableChest.A_ChestOpened -= OnChestWindowOpen;
        OpenChest.A_Open -= OnChestWindowOpen;
    }

    private void OnChestWindowOpen()
    {
        // Calculate damage per chest based on stacks
        damagePerChest = damagePerChestAmount * amount;

        // Accumulate damage permanently
        accumulatedDamage += damagePerChest;

        // Apply as moving stat modifier
        var modifier = new StatModifier
        {
            stat = EStat.Power, // ID 12
            value = accumulatedDamage
        };

        Player.Instance.inventory.statInventory.ChangeMovingStat(
            EItem.CreditCardRed.ToString(),
            modifier
        );
    }
}
```

## Technical Notes
- Uses Unity's delegate system for event handling
- The moving stat system allows dynamic updates without recreating stat modifiers
- Constructor only sets the base `damagePerChestAmount` value
- All damage calculation happens in the event handler
- EItem enum value 63 is used as the damage source identifier

## Related Items
- **CreditCardGreen**: Similar chest-based mechanics but affects luck instead of damage
- **GoldenShield**: Another economy item that generates gold from taking damage
- **Items with accumulative mechanics**: DemonicBlood, DemonicSoul, JoesDagger

## Synergies
- **High chest frequency runs**: More valuable when many chests are encountered
- **Luck items**: Increased chest quality may provide more opportunities to trigger
- **Economy items**: Can be part of economic builds focusing on chest interactions

---

*Data extracted from decompiled IL2CPP constructors and IDA Pro analysis*