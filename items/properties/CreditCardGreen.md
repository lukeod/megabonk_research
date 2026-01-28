# CreditCardGreen

## Overview
- **Item ID**: EItem.CreditCardGreen
- **Constructor Address**: 0x18043F5F0
- **Category**: Economy/Utility Item
- **Rarity**: Unknown (requires game data analysis)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| luckPerChestPerAmount | float | 0.02 | Luck gained per chest opened per stack |
| chestPriceIncreasePerAmount | float | 0.1 | Chest price increase per stack (10%) |
| luckPerChest | float | Runtime | Calculated value based on stack count |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 34 | ChestPriceIncrease | chestPriceIncreasePerAmount * amount | Additive (type 2) |

## Special Mechanics
- **Event-Driven Luck Accumulation**: Subscribes to chest opening events (`ChestWindowUi.A_Open`)
- **Temporary Luck Boost**: Provides luck bonus when chests are opened
- **No Direct Stat Modifications**: Does not modify base luck stat directly, only accumulates on chest events
- **Event Subscription Management**: Properly subscribes on Init() and unsubscribes on Cleanup()

## Formulas
```
luckPerChest = amount * luckPerChestPerAmount
luckPerChest = amount * 0.02

chestPriceIncrease = amount * chestPriceIncreasePerAmount
chestPriceIncrease = amount * 0.1
```

## Implementation Details
- **Update Frequency**: Event-driven (only on chest opening)
- **Event Subscriptions**: `ChestWindowUi.A_Open` - Chest window opening event
- **Stack Behavior**: Linear scaling - each stack adds 2% luck per chest and 10% chest price increase
- **Memory Management**: Uses IL2CPP garbage collection barriers for event subscription

## C# Pseudocode
```csharp
public class ItemCreditCardGreen : ItemBase
{
    private float luckPerChestPerAmount = 0.02f;
    private float chestPriceIncreasePerAmount = 0.1f;
    private float luckPerChest;

    public override void Init()
    {
        // Subscribe to chest opening events
        InteractableChest.A_ChestOpened += OnChestWindowOpen;
        OpenChest.A_Open += OnChestWindowOpen;
    }

    public override void Cleanup()
    {
        // Unsubscribe from chest opening events
        InteractableChest.A_ChestOpened -= OnChestWindowOpen;
        OpenChest.A_Open -= OnChestWindowOpen;
    }

    protected override void OnInitOrAmountChanged()
    {
        // Update calculated values when stack amount changes
        luckPerChest = amount * luckPerChestPerAmount;

        // Apply chest price increase stat modifier
        var modifier = new StatModifier
        {
            operationType = 2,  // Additive
            stat = 34,          // ChestPriceIncrease
            value = chestPriceIncreasePerAmount * amount
        };
        SetStat(modifier);
    }
}
```

## Technical Notes
- **IL2CPP Implementation**: Uses native method invocation for performance
- **Event System**: Leverages Unity's delegate system for chest interaction detection
- **Thread Safety**: Event subscription uses proper IL2CPP garbage collection barriers
- **Memory Efficiency**: No continuous update loop - only activates on chest events
- **Cross-Reference**: Similar pattern to CreditCardRed (damage-based chest bonus)

## Implementation Quirks
- **Native Wrapper**: C# code is mostly IL2CPP interop wrappers around native implementations
- **Event Method Caching**: Uses method pointer caching for performance optimization
- **Static Initialization**: Class initialization includes metadata setup for IL2CPP runtime
- **Virtual Method Dispatch**: Inherits standard ItemBase lifecycle methods

## Related Items
- **CreditCardRed**: Sister item providing damage bonus per chest opened (0.025 damage per chest)
- **GoldenShield**: Another economy-focused item that generates gold
- **GoldenSneakers**: Movement-based gold generation item
- **Key**: Utility item affecting chest mechanics with diminishing returns

## Usage Patterns
- **Early Game**: Provides consistent luck scaling for improved loot quality
- **Chest-Heavy Runs**: Synergizes well with builds focused on finding many chests
- **Luck Builds**: Complements other luck-boosting items like Clover (0.075 luck per stack)
- **Economic Strategy**: Part of economic item synergy with other gold/loot enhancement items

---

**Data Sources:**
- `megabonk_research/items.md` - Base properties and mechanics overview
- `extracted_constructors/items/CreditCardGreen.c` - Native constructor implementation (0x18043F5F0)
- `decompiled/Assembly-CSharp/.../ItemCreditCardGreen.cs` - C# class structure and method signatures
- `decompiled/Assembly-CSharp/.../ItemBase.cs` - Base class understanding