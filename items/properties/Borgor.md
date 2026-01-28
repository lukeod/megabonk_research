# Borgor

## Overview
- **Item ID**: EItem.Borgor
- **Constructor Address**: 0x18043C4F0
- **Category**: Healing/Survival
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| baseChance | float | 0.02 | 2% base proc chance |
| chancePerAmount | float | 0.01 | 1% proc chance per stack |
| ratioHeal | float | 0.08 | 8% ratio heal component |
| flatHealPerAmount | int | 2 | Flat healing per stack |
| flatHeal | int | 10 | Base flat heal (calculated) |
| chance | float | calculated | Total proc chance (calculated) |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | No direct stat modifiers | - | - |

## Special Mechanics
- **Trigger**: Enemy death events (subscribes to A_EnemyDied)
- **Healing Formula**: Combination of flat healing and ratio-based healing
- **Proc Chance**: Linear scaling with stack count
- **Event-Driven**: Responds to enemy death through Unity's event system

## Formulas

### Proc Chance Calculation
```
chance = baseChance + (amount - 1) * chancePerAmount
chance = 0.02 + (stacks - 1) * 0.01
```

### Healing Calculation
```
flatHeal = amount * flatHealPerAmount + 8
flatHeal = stacks * 2 + 8
```

### Total Healing (Estimated)
```
totalHeal = flatHeal + (maxHealth * ratioHeal)
totalHeal = (stacks * 2 + 8) + (maxHealth * 0.08)
```

## Implementation Details
- **Update Frequency**: Event-driven (no tick-based updates)
- **Event Subscriptions**:
  - Init: Subscribes to Enemy.A_EnemyDied
  - Cleanup: Unsubscribes from Enemy.A_EnemyDied
- **Stack Behavior**:
  - Proc chance scales linearly
  - Flat healing scales linearly
  - Ratio healing remains constant per proc

## C# Pseudocode
```csharp
// Constructor logic
public ItemBorgor(ItemInventory itemInventoryRef) : base(itemInventoryRef, 0)
{
    this.baseChance = 0.02f;
    this.chancePerAmount = 0.01f;
    this.ratioHeal = 0.08f;
    this.flatHealPerAmount = 2;
    this.flatHeal = 10;
}

// Amount change recalculation
protected override void OnInitOrAmountChanged()
{
    float stackBonus = (float)(this.amount - 1);
    this.flatHeal = this.amount * this.flatHealPerAmount + 8;
    this.chance = stackBonus * this.chancePerAmount + this.baseChance;
}

// Event subscription
public override void Init()
{
    Enemy.A_EnemyDied += OnEnemyDied;
}

// Event cleanup
public override void Cleanup()
{
    Enemy.A_EnemyDied -= OnEnemyDied;
}

// Healing logic (estimated implementation)
private void OnEnemyDied(Enemy enemy, DamageContainer deathSource)
{
    if (UnityEngine.Random.value <= this.chance)
    {
        float healAmount = this.flatHeal + (player.maxHealth * this.ratioHeal);
        player.Heal(healAmount);
    }
}
```

## Technical Notes
- Uses IL2CPP delegate system for event subscription/unsubscription
- Event handlers are properly managed in Init() and Cleanup() methods
- Healing calculation includes both flat and percentage-based components
- Linear scaling prevents exponential power growth
- No cooldown or maximum proc limit per death event

## Related Items
- **DemonicBlood**: Also triggers on enemy death, provides HP stat bonuses
- **DemonicSoul**: Similar enemy death trigger, provides attack damage bonuses
- **SoulHarvester**: Enemy death trigger with projectile spawning
- **CursedDoll**: Enemy death mechanics with damage dealing

## Scaling Analysis

### Single Stack (Amount = 1)
- Proc Chance: 2%
- Flat Heal: 10 HP
- Ratio Heal: 8% max HP

### Five Stacks (Amount = 5)
- Proc Chance: 6% (2% + 4 * 1%)
- Flat Heal: 18 HP (5 * 2 + 8)
- Ratio Heal: 8% max HP

### Ten Stacks (Amount = 10)
- Proc Chance: 11% (2% + 9 * 1%)
- Flat Heal: 28 HP (10 * 2 + 8)
- Ratio Heal: 8% max HP

---

*Data extracted from decompiled IL2CPP constructors and C# interface analysis*