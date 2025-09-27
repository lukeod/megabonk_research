# JoesDagger

## Overview
- **Item ID**: 49 (EItem.JoesDagger)
- **Constructor Address**: 0x1804144D0
- **Category**: Execution/Damage Scaling Item
- **Rarity**: Unknown (not determinable from constructor)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| attackDamagePerProc | float | 0.01 | 1% attack damage per proc |
| executionChancePerAmount | float | 0.01 | 1% execution chance per stack |
| executionChance | float | 0.01 | Base execution chance (1%) |
| maxStacks | int | 999999 | Maximum damage accumulation stacks |
| accumulatedDamaged | float | Dynamic | Accumulates from enemy hits |
| stacks | int | Dynamic | Current damage accumulation stacks |
| lastUsedStacks | int | Dynamic | Last stack count when stat was updated |
| nextUpdateTime | float | Dynamic | Next time to update power stat |
| damageSource | string | "JoesDagger" | Damage source identifier |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 12 | Power/Damage | accumulatedDamaged | Dynamic based on hits |

## Special Mechanics

### Execution System
- **Execution Chance**: 1% per stack (amount × 0.01)
- **Execution Trigger**: On PreAttack phase
- **Execution Effect**: Calls `DamageUtility.ApplyExecute()` which instantly kills the target
- **Random Check**: Uses `MyRandom.random.NextDouble()` for proc determination

### Damage Accumulation System
- **Accumulation Trigger**: Subscribes to `Enemy.A_Damage` event via `OnEnemyDamage`
- **Stack Tracking**: Maintains internal stacks up to 999,999 maximum
- **Power Scaling**: Each accumulated damage point adds to power stat (EStat 12)
- **Update Frequency**: Checks every 1.0 second for stack changes
- **Stat Application**: Only updates power stat when stacks have increased

### Event Subscriptions
- **Init**: Subscribes to global enemy damage events
- **Cleanup**: Unsubscribes from enemy damage events
- **Event Handler**: `OnEnemyDamage(Enemy e, DamageContainer dc)`

## Formulas

### Execution Chance Calculation
```
executionChance = amount × 0.01
procCheck = Random.NextDouble() ≤ executionChance
```

### Attack Damage Scaling (OnInitOrAmountChanged)
```
attackDamagePerProc = amount × 0.01
```

### Power Stat Update (Tick)
```
if (currentTime ≥ nextUpdateTime && stacks > lastUsedStacks) {
    SetStat(EStat.Power, accumulatedDamaged, AdditiveBonusType.Additive)
    lastUsedStacks = stacks
    nextUpdateTime = currentTime + 1.0
}
```

## Implementation Details
- **Update Frequency**: 1.0 second intervals for stat updates
- **Event Subscriptions**: Global enemy damage tracking
- **Stack Behavior**: Unlimited accumulation up to 999,999 stacks
- **Execution Priority**: Processed in PreAttack phase before damage calculation
- **Damage Source**: Sets damage source to "JoesDagger" when executing

## C# Pseudocode
```csharp
// Constructor logic
public ItemJoesDagger() {
    attackDamagePerProc = 0.01f;
    executionChancePerAmount = 0.01f;
    executionChance = 0.01f;
    maxStacks = 999999;
    damageSource = "JoesDagger";
}

// On amount changed
protected override void OnInitOrAmountChanged() {
    attackDamagePerProc = amount * 0.01f;
}

// Pre-attack execution check
public override void PreAttack(DamageContainer dc, StatComponents itemAttackModifier) {
    if (MyRandom.random.NextDouble() <= executionChance) {
        dc.damageSource = damageSource;
        DamageUtility.ApplyExecute(dc);
    }
}

// Damage accumulation system
private void OnEnemyDamage(Enemy e, DamageContainer dc) {
    // Accumulates damage and increases stacks
    // Implementation details in native code
}

// Periodic stat updates
public override void Tick() {
    if (Time.time >= nextUpdateTime) {
        if (stacks > lastUsedStacks) {
            SetStat(EStat.Power, accumulatedDamaged, AdditiveBonusType.Additive);
            lastUsedStacks = stacks;
        }
        nextUpdateTime = Time.time + 1.0f;
    }
}
```

## Technical Notes
- **IL2CPP Limitation**: The actual damage accumulation logic in `OnEnemyDamage` is implemented in native code and not visible in the decompiled C#
- **Performance**: Uses 1-second update intervals to avoid per-frame stat recalculation
- **Execution Mechanics**: Leverages the game's built-in execution system via `DamageUtility.ApplyExecute()`
- **Stack Management**: Maintains separate tracking for current stacks vs. last applied stacks to minimize redundant stat updates
- **Event System**: Uses Unity's delegate system for global enemy damage tracking

## Related Items
- **DemonBlade**: Also has execution mechanics (25% heal chance per stack on execution)
- **DemonicSoul**: Similar stacking damage bonus system based on enemy deaths
- **DemonicBlood**: Parallel HP stacking system based on enemy deaths
- **GiantFork**: Alternative high-damage system with mega-crit mechanics

---

**Data Sources**:
- megabonk_research/items.md (Base properties)
- decompiled/Assembly-CSharp/Assets.Scripts.Inventory__Items__Pickups.Items.ItemImplementations/ItemJoesDagger.cs (Class structure)
- extracted_constructors/items/JoesDagger.c (Constructor implementation)
- decompiled/Assembly-CSharp/Assets.Scripts.Game.Combat/DamageUtility.cs (Execution system reference)