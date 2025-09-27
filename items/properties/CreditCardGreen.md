# CreditCardGreen

## Overview
- **Item ID**: EItem.CreditCardGreen
- **Constructor Address**: 0x180405AE0
- **Category**: Economy/Utility Item
- **Rarity**: Unknown (requires game data analysis)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| luckPerChestPerAmount | float | 0.02 | Luck gained per chest opened per stack |
| luckPerChest | float | Runtime | Calculated value based on stack count |
| accumulatedLuck | float | Runtime | Total accumulated luck from chest openings |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 30 | Luck | Dynamic | Accumulated through chest opening events |

## Special Mechanics
- **Event-Driven Luck Accumulation**: Subscribes to chest opening events (`ChestWindowUi.A_Open`)
- **Temporary Luck Boost**: Provides luck bonus when chests are opened
- **No Direct Stat Modifications**: Does not modify base luck stat directly, only accumulates on chest events
- **Event Subscription Management**: Properly subscribes on Init() and unsubscribes on Cleanup()

## Formulas
```
luckPerChest = amount * luckPerChestPerAmount
luckPerChest = amount * 0.02

Total luck gain per chest = amount * 0.02
```

## Implementation Details
- **Update Frequency**: Event-driven (only on chest opening)
- **Event Subscriptions**: `ChestWindowUi.A_Open` - Chest window opening event
- **Stack Behavior**: Linear scaling - each stack adds 2% luck per chest opened
- **Memory Management**: Uses IL2CPP garbage collection barriers for event subscription

## C# Pseudocode
```csharp
public class ItemCreditCardGreen : ItemBase
{
    private float luckPerChestPerAmount = 0.02f;
    private float luckPerChest;
    private float accumulatedLuck;

    public override void Init()
    {
        // Subscribe to chest opening events
        ChestWindowUi.A_Open += OnChestWindowOpen;
    }

    public override void Cleanup()
    {
        // Unsubscribe from chest opening events
        ChestWindowUi.A_Open -= OnChestWindowOpen;
    }

    private void OnChestWindowOpen()
    {
        // Calculate luck bonus based on current stack amount
        float luckBonus = amount * luckPerChestPerAmount;
        accumulatedLuck += luckBonus;

        // Apply temporary luck boost to player stats
        // (Implementation details not visible in decompiled code)
    }

    protected override void OnInitOrAmountChanged()
    {
        // Update calculated values when stack amount changes
        luckPerChest = amount * luckPerChestPerAmount;
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
- `extracted_constructors/items/CreditCardGreen.c` - Native constructor implementation
- `decompiled/Assembly-CSharp/.../ItemCreditCardGreen.cs` - C# class structure and method signatures
- `decompiled/Assembly-CSharp/.../ItemBase.cs` - Base class understanding