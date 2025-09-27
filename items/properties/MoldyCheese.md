# MoldyCheese

## Overview
- **Item ID**: EItem.MoldyCheese
- **Constructor Address**: 0x180421870
- **Category**: Status Effect (Poison)
- **Rarity**: Common

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| chanceToStackPerAmount | float | 0.4 | 40% base chance to apply poison per stack |
| totalChance | float | Dynamic | Calculated during runtime |
| procsSinceLastTick | int | Dynamic | Statistics tracking |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | - | - | No direct stat modifications |

## Special Mechanics

### Poison Application
- **Debuff Type**: Poison (ID: 1)
- **Duration**: 5.0 seconds
- **Proc Chance**: 40% per stack (chanceToStackPerAmount)
- **Stacking Logic**: Uses floor calculation for guaranteed stacks plus probabilistic remainder

### Stack Calculation System
The item uses a sophisticated stacking system:

1. **Base Calculation**: `totalChance` is calculated during `OnInitOrAmountChanged()`
2. **Guaranteed Stacks**: `floor(totalChance)` poison stacks are always applied
3. **Probabilistic Stack**: The remainder (`totalChance - floor(totalChance)`) has a chance to add one more stack
4. **Proc Coefficient**: Uses the damage container's `procCoefficient` for scaling

### Statistics Tracking
- Tracks poison applications in `procsSinceLastTick`
- Reports to MyStats system (stat ID 35) every tick
- Resets counter after reporting

## Formulas

### Guaranteed Stacks
```
guaranteedStacks = floor(totalChance)
```

### Probabilistic Stack
```
if (remainder > 0 && TryProc(procCoefficient, remainder)) {
    stacks += 1
}
where remainder = totalChance - guaranteedStacks
```

### Total Poison Stacks Applied
```
finalStacks = guaranteedStacks + (probabilisticStack ? 1 : 0)
```

## Implementation Details

- **Update Frequency**: On hit events (ProcOnHitEffects)
- **Event Subscriptions**: Responds to damage dealt to enemies
- **Stack Behavior**: Each item instance contributes independently to poison chance
- **Debuff Duration**: Fixed 5.0 seconds regardless of stack count

## C# Pseudocode
```csharp
// Constructor logic
public ItemMoldyCheese(ItemInventory itemInventoryRef) : base(itemInventoryRef) {
    chanceToStackPerAmount = 0.4f; // 40% per stack
}

// On hit processing
public override void ProcOnHitEffects(DamageContainer dc) {
    if (totalChance > 0) {
        int guaranteedStacks = (int)Math.Floor(totalChance);
        float remainder = totalChance - guaranteedStacks;

        // Check for probabilistic additional stack
        if (remainder > 0 && ItemUtility.TryProc(dc.procCoefficient, remainder)) {
            guaranteedStacks++;
        }

        if (guaranteedStacks > 0 && dc.enemy != null) {
            // Apply poison debuff (type 1) for 5 seconds
            dc.enemy.AddDebuff(1, dc, 5.0f, guaranteedStacks);
            procsSinceLastTick += guaranteedStacks;
        }
    }
}

// Statistics reporting
public override void Tick() {
    if (procsSinceLastTick > 0) {
        MyStats.AddValue(35, procsSinceLastTick); // Report poison applications
        procsSinceLastTick = 0; // Reset counter
    }
}
```

## Technical Notes

- **IL2CPP Compatibility**: Fully integrated with IL2CPP runtime system
- **Memory Management**: Uses proper IL2CPP object lifecycle management
- **Performance**: Minimal overhead as only processes on successful hits
- **Null Safety**: Includes proper null checks for enemy references
- **Statistics Integration**: Contributes to global game statistics tracking

## Related Items

### Similar Poison Items
- **ToxicBarrel**: Area poison on player damage
- **GlovesPoison**: Area poison explosion on cooldown

### Status Effect Items
- **BloodyCleaver**: Applies bleeding debuff
- **UnstableTransfusion**: Also applies bleeding with different mechanics
- **Dragonfire**: Applies burn debuff

### Proc-Based Items
- **Bonker**: Uses similar probabilistic proc system
- **EagleClaw**: Similar knockup proc mechanics
- **SluttyCannon**: Comparable proc chance scaling

---

*Data extracted from IL2CPP decompiled constructor at 0x180421870 and C# class definition.*