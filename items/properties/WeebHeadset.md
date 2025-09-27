# WeebHeadset

## Overview
- **Item ID**: ItemWeebHeadset
- **Constructor Address**: 0x1804273A0
- **Category**: Elemental/Status (Charm Effect)
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| charmChancePerAmount | float | 0.02 | 2% charm chance per stack |
| durationPerAmount | float | 1.5 | 1.5 seconds duration per stack |
| maxProcsPerTick | int | 100 | Maximum procs per tick |
| charmDuration | float | Dynamic | Calculated on init/amount change |
| charmChance | float | Dynamic | Calculated with hyperbolic scaling |
| numProcsThisTick | int | 0 | Reset each tick |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | N/A | N/A | No direct stat modifications |

## Special Mechanics
- **Charm Effect**: Applies charm debuff (ID 32) to enemies on hit
- **Proc Limiting**: Limited to 100 procs per tick to prevent performance issues
- **Hyperbolic Scaling**: Charm chance uses hyperbolic scaling to prevent 100% rates
- **Debuff Check**: Only procs if enemy doesn't already have charm debuff
- **Proc Coefficient**: Uses damage container's proc coefficient for chance calculation

## Formulas

### Charm Duration
```
charmDuration = (amount * durationPerAmount) + 5.0
charmDuration = (amount * 1.5) + 5.0
```

### Charm Chance (Hyperbolic Scaling)
```
baseChance = amount * charmChancePerAmount
baseChance = amount * 0.02

charmChance = HyperbolicScaling(baseChance, 0.1, 4.0)
if (charmChance < 0.02) charmChance = 0.02  // Minimum 2%
```

### Hyperbolic Scaling Function
```
// Parameters: value=baseChance, factor=0.1, steepness=4.0
scaledValue = value / (value + factor * steepness)
```

## Implementation Details
- **Update Frequency**: Every tick (resets numProcsThisTick)
- **Event Subscriptions**: ProcOnHitEffects hook
- **Stack Behavior**: Linear scaling for duration, hyperbolic scaling for chance
- **Minimum Chance**: Always maintains at least 2% charm chance

## C# Pseudocode
```csharp
// Constructor logic
public ItemWeebHeadset(ItemInventory itemInventoryRef) {
    this.charmChancePerAmount = 0.02f;
    this.durationPerAmount = 1.5f;
    this.maxProcsPerTick = 100;
    base(itemInventoryRef);
}

// OnInitOrAmountChanged
protected override void OnInitOrAmountChanged() {
    // Calculate charm duration
    this.charmDuration = (this.amount * this.durationPerAmount) + 5.0f;

    // Calculate charm chance with hyperbolic scaling
    float baseChance = this.amount * this.charmChancePerAmount;
    this.charmChance = HyperbolicScaling(baseChance, 0.1f, 4.0f);

    // Ensure minimum charm chance
    if (this.charmChance < 0.02f) {
        this.charmChance = 0.02f;
    }
}

// ProcOnHitEffects
public override void ProcOnHitEffects(DamageContainer dc) {
    if (dc?.enemy != null && !dc.enemy.IsDead() &&
        !dc.enemy.HasDebuff(32) && // Not already charmed
        numProcsThisTick < maxProcsPerTick &&
        ItemUtility.TryProc(dc.procCoefficient, charmChance)) {

        dc.enemy.Charm(dc, charmDuration);
        numProcsThisTick++;
    }
}

// Tick
public override void Tick() {
    numProcsThisTick = 0; // Reset proc counter
}
```

## Technical Notes
- **Performance Optimization**: Proc limiting prevents excessive charm applications per frame
- **Debuff ID**: Uses debuff type 32 (Charm) from the game's debuff system
- **Proc Coefficient**: Respects weapon/attack proc coefficients for balanced scaling
- **Dead Enemy Check**: Prevents wasted procs on already dead enemies
- **Duplicate Prevention**: Won't charm already charmed enemies

## Related Items
- **Other Charm Items**: None found in current item list
- **Debuff Items**: MoldyCheese (poison), BloodyCleaver (bleeding), UnstableTransfusion (bleeding)
- **Hyperbolic Scaling Items**: Dragonfire, EagleClaw, IceCrystal, IceCube, LightningOrb, SluttyCannon

## Scaling Analysis
With hyperbolic scaling (factor=0.1, steepness=4.0):

| Stacks | Base Chance | Effective Chance | Duration |
|--------|-------------|------------------|----------|
| 1 | 2.0% | 4.76% | 6.5s |
| 5 | 10.0% | 20.0% | 12.5s |
| 10 | 20.0% | 33.33% | 20.0s |
| 20 | 40.0% | 50.0% | 35.0s |
| 50 | 100.0% | 71.43% | 80.0s |

---

*Data extracted from decompiled IL2CPP constructors and C# interop layer via IDA Pro MCP*