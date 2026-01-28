# Gasmask

## Overview
- **Item ID**: EItem.Gasmask
- **Constructor Address**: 0x180457880
- **Category**: Defensive/Adaptive
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| armorPerStack | float | 0.005 | Armor gained per poisoned enemy |
| overhealPerStack | float | 0.005 | Overheal gained per poisoned enemy |
| maxArmorPerAmount | float | 0.4 | Maximum armor per item stack |
| maxOverhealPerAmount | float | 0.25 | Maximum overheal per item stack |
| updateInterval | float | 1.0 | Update frequency in seconds |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 4 | Armor | min(poisonedEnemies * 0.005, amount * 0.4) | Dynamic |
| 47 | Overheal | min(poisonedEnemies * 0.005, amount * 0.25) | Dynamic |

## Special Mechanics
The Gasmask provides dynamic defensive bonuses based on the number of poisoned enemies in the area. The item continuously monitors poisoned enemies and adjusts the player's armor and overheal stats accordingly.

- **Poison Detection**: Tracks the global count of poisoned enemies via `DebuffPoison.numPoisonedEnemies`
- **Dynamic Scaling**: Both armor and overheal scale with the number of poisoned enemies, up to the maximum cap per stack
- **Real-time Updates**: Stats are recalculated every 1 second when the poison count changes
- **Event Subscription**: Subscribes to stage start events for initialization

## Formulas

### Armor Calculation
```
currentArmor = min(numPoisonedEnemies * armorPerStack, amount * maxArmorPerAmount)
currentArmor = min(numPoisonedEnemies * 0.005, amount * 0.4)
```

### Overheal Calculation
```
currentOverheal = min(numPoisonedEnemies * overhealPerStack, amount * maxOverhealPerAmount)
currentOverheal = min(numPoisonedEnemies * 0.005, amount * 0.25)
```

### Maximum Effective Poisoned Enemies
```
maxEffectivePoison_Armor = amount * maxArmorPerAmount / armorPerStack = amount * 80
maxEffectivePoison_Overheal = amount * maxOverhealPerAmount / overhealPerStack = amount * 50
```

## Implementation Details
- **Update Frequency**: 1.0 second intervals
- **Event Subscriptions**: GameManager.A_StageStarted for initialization
- **Stack Behavior**: Linear scaling of maximum caps with item amount
- **State Tracking**: Uses `lastStoredStacks` to avoid unnecessary stat updates

## C# Pseudocode
```csharp
// Constructor logic
public ItemGasmask(ItemInventory itemInventoryRef) {
    armorPerStack = 0.005f;
    overhealPerStack = 0.005f;
    maxArmorPerAmount = 0.4f;
    maxOverhealPerAmount = 0.25f;
    updateInterval = 1.0f;
}

// Update logic (called every 1 second)
public void Tick() {
    if (MyTime.time >= nextUpdateTime) {
        nextUpdateTime = MyTime.time + updateInterval;

        int numPoisonedEnemies = DebuffPoison.numPoisonedEnemies;

        if (lastStoredStacks != numPoisonedEnemies) {
            // Update Overheal (EStat 47)
            float overhealValue = Mathf.Min(
                numPoisonedEnemies * overhealPerStack,
                maxOverheal
            );
            SetStat(new StatModifier {
                statType = 47,
                modifyType = 2, // Set value
                value = overhealValue
            });

            // Update Armor (EStat 4)
            float armorValue = Mathf.Min(
                numPoisonedEnemies * armorPerStack,
                maxArmor
            );
            SetStat(new StatModifier {
                statType = 4,
                modifyType = 2, // Set value
                value = armorValue
            });

            lastStoredStacks = numPoisonedEnemies;
        }
    }
}

// Initialization
public void OnInitOrAmountChanged() {
    maxArmor = amount * maxArmorPerAmount;
    maxOverheal = amount * maxOverhealPerAmount;
    nextUpdateTime = MyTime.time + updateInterval;
}
```

## Technical Notes
- **Performance Optimization**: Only updates stats when the poison count changes, avoiding unnecessary calculations
- **Global State Dependency**: Relies on the global poison debuff system to track enemy states
- **Memory Management**: Uses efficient field access patterns for IL2CPP interop
- **Update Pattern**: Uses time-based updates rather than frame-based for consistent behavior

## Related Items
- **MoldyCheese**: Applies poison debuffs that this item benefits from
- **GlovesPoison**: Creates poisoned areas that boost Gasmask effectiveness
- **ToxicBarrel**: Poison area effects that work synergistically with Gasmask
- **Chonkplate**: Another overheal provider, but with fixed values
- **SpikyShield**: Another armor-based defensive item with different mechanics

---

*Data extracted from IL2CPP constructor analysis and decompiled C# source code*