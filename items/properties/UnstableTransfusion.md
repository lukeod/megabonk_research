# UnstableTransfusion

## Overview
- **Item ID**: EItem.UnstableTransfusion
- **Constructor Address**: 0x180468A60
- **Category**: Status Effect / Bleeding
- **Rarity**: Unknown (determinable from game data)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| chanceToStackPerAmount | float | 0.35 | 35% base chance per stack |
| totalChance | float | Dynamic | Calculated as amount * chanceToStackPerAmount |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | No direct stat modifications | N/A | N/A |

## Special Mechanics
- **Bleeding Application**: Applies bleeding debuff (ID 64) for 5.0 seconds on successful proc
- **Stacking Logic**: Uses sophisticated floor calculation for guaranteed stacks plus probability-based remainder proc
- **Proc-Based**: Triggers on hit using ProcOnHitEffects with proc coefficient scaling
- **Dual Chance System**: First proc for floor(totalChance), then additional proc for remainder

### Bleeding Mechanics
- **Debuff Type**: 64 (Bleeding)
- **Duration**: 5.0 seconds
- **Stack Count**: Dynamic based on proc results (floor + potential remainder)
- **Target**: Enemy hit by the attack

## Formulas

### Total Chance Calculation
```
totalChance = amount * chanceToStackPerAmount
totalChance = amount * 0.35
```

### Stack Determination
```csharp
guaranteedStacks = (int)Math.Floor(totalChance);
remainderChance = totalChance - guaranteedStacks;

// Apply guaranteed stacks
if (guaranteedStacks > 0) {
    ApplyBleeding(enemy, guaranteedStacks);
}

// Roll for remainder stack
if (TryProc(procCoefficient, remainderChance)) {
    ApplyBleeding(enemy, 1);
}
```

### Examples
- **1 stack**: 35% chance for 1 bleeding stack
- **2 stacks**: 70% chance - guaranteed 0 stacks + 70% chance for 1 stack
- **3 stacks**: 105% chance - guaranteed 1 stack + 5% chance for 1 additional stack
- **4 stacks**: 140% chance - guaranteed 1 stack + 40% chance for 1 additional stack
- **8 stacks**: 280% chance - guaranteed 2 stacks + 80% chance for 1 additional stack

## Implementation Details
- **Update Frequency**: On-demand (ProcOnHitEffects)
- **Event Subscriptions**: None (purely reactive to hits)
- **Stack Behavior**: Each successful proc applies bleeding debuff to target enemy
- **Proc Mechanism**: Uses ItemUtility.TryProc with damage container's proc coefficient

## C# Pseudocode
```csharp
// Constructor logic
public void Constructor(ItemInventory itemInventoryRef) {
    chanceToStackPerAmount = 0.35f;
    base.Constructor(itemInventoryRef);
}

// Amount change handler
public void OnInitOrAmountChanged() {
    totalChance = (float)amount * chanceToStackPerAmount;
}

// Hit processing
public void ProcOnHitEffects(DamageContainer dc) {
    if (dc == null || dc.enemy == null) return;

    if (!ItemUtility.TryProc(dc.procCoefficient, totalChance)) return;

    int stacks = 0;

    // Calculate guaranteed stacks
    if (chanceToStackPerAmount > 0.0f) {
        int guaranteedStacks = (int)Math.Floor(totalChance);
        stacks = guaranteedStacks;

        // Roll for remainder chance
        float remainder = totalChance - guaranteedStacks;
        if (ItemUtility.TryProc(dc.procCoefficient, remainder)) {
            stacks++;
        }
    }

    // Apply bleeding debuff
    if (stacks > 0) {
        dc.enemy.AddDebuff(64, dc, 5.0f, stacks);
    }
}
```

## Technical Notes
- **Floating Point Precision**: Uses 0.34999999 in native code due to IEEE-754 representation
- **Math.Floor Usage**: Ensures deterministic guaranteed stack calculation
- **Proc Coefficient Scaling**: Both initial proc and remainder proc respect damage container's proc coefficient
- **Null Safety**: Includes checks for null damage container and enemy
- **Memory Management**: Uses IL2CPP garbage collection barriers for object references

## Related Items
- **BloodyCleaver**: Also applies bleeding debuff (50% chance per stack, 5.0s duration, lifesteal-triggered)
- **GlovesBlood**: Creates bleeding explosion effect (5s duration, area-based)
- **Items with similar proc mechanics**: Bonker, Dragonfire, EagleClaw (all use chance-based proc systems)

---

*Data sources:*
- *megabonk_research/items.md (lines 1098-1110)*
- *decompiled/Assembly-CSharp/Assets.Scripts.Inventory__Items__Pickups.Items.ItemImplementations/ItemUnstableTransfusion.cs*
- *extracted_constructors/items/UnstableTransfusion.c (constructor at 0x180468A60, OnInitOrAmountChanged at 0x18043C690, ProcOnHitEffects at 0x180468960)*
- *decompiled/Assembly-CSharp/Assets.Scripts.Inventory__Items__Pickups.Items/ItemBase.cs (base class understanding)*