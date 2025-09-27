# DemonBlade

## Overview
- **Item ID**: DemonBlade (from EItem enum)
- **Constructor Address**: 0x180407420
- **Category**: Hybrid Damage/Healing Item
- **Rarity**: Unknown (no specific rarity data found in constructor)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| critChance | float | 0.01 (1%) | Base critical hit chance |
| healChancePerStack | float | 0.25 (25%) | Heal chance per stack |
| totalHealChance | float | Dynamic | Calculated as `amount * healChancePerStack` |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 18 | Critical Chance | 0.01 | Flat (no scaling) |

## Special Mechanics
- **Healing on Enemy Damage**: Subscribes to the global enemy damage event (`Enemy.A_Damage`)
- **Stack-based Heal Chance**: Each stack increases healing chance by 25%
- **Event-driven Behavior**: Uses Unity's event system to respond to enemy damage across the game
- **Calculated Total Heal Chance**: Updates dynamically when item amount changes

## Formulas
- **Total Heal Chance**: `amount * 0.25` (25% per stack)
- **Critical Chance**: Fixed at 1% regardless of stack count
- **Heal Trigger**: Occurs on any enemy taking damage (global event subscription)

## Implementation Details
- **Update Frequency**: On amount change only (via `OnInitOrAmountChanged`)
- **Event Subscriptions**: `Assets.Scripts.Actors.Enemies.Enemy.A_Damage`
- **Stack Behavior**: Linear scaling for heal chance only
- **StatModifier Type**: Uses type 2 (likely additive) for critical chance stat

## C# Pseudocode
```csharp
// Simplified constructor logic
public ItemDemonBlade(ItemInventory itemInventoryRef) : base(itemInventoryRef)
{
    this.critChance = 0.01f;                // 1% crit chance
    this.healChancePerStack = 0.25f;        // 25% heal chance per stack
}

// OnInitOrAmountChanged logic
protected override void OnInitOrAmountChanged()
{
    // Update total heal chance based on current stack count
    this.totalHealChance = this.amount * this.healChancePerStack;

    // Set critical chance stat modifier
    StatModifier critMod = new StatModifier();
    critMod.type = 2;                       // Additive type
    critMod.stat = EStat.CriticalChance;    // EStat 18
    critMod.value = this.critChance;        // 1%
    SetStat(critMod);
}

// Init logic - Subscribe to global enemy damage events
public override void Init()
{
    Enemy.A_Damage += OnEnemyDamaged;
}

// Cleanup logic - Unsubscribe from events
public override void Cleanup()
{
    Enemy.A_Damage -= OnEnemyDamaged;
}

// Event handler - Called whenever any enemy takes damage
private void OnEnemyDamaged(Enemy enemy, DamageContainer damageContainer)
{
    // Implementation details not visible in constructor
    // Likely checks totalHealChance against random roll
    // and heals player if successful
}
```

## Technical Notes
- **Performance Consideration**: Event subscription is global, so this item responds to ALL enemy damage in the game
- **Memory Management**: Properly subscribes in Init() and unsubscribes in Cleanup() to prevent memory leaks
- **StatModifier Usage**: Uses the game's stat system to apply critical chance bonus
- **IL2CPP Compatibility**: Designed to work with Unity's IL2CPP compilation system

## Related Items
- **BloodyCleaver**: Also has healing mechanics tied to enemy interactions
- **GiantFork**: Another critical chance item (15% per stack vs 1% flat)
- **Chonkplate**: Has lifesteal mechanics (different from proc-based healing)
- **GlovesBlood**: Has healing component with damage reflection

## Synergies
- **Critical Chance Items**: GiantFork, GrandmasSecretTonic - stack critical chance
- **Healing Items**: HolyBook, Medkit, Borgor - create healing-focused builds
- **Damage Items**: Items that increase damage dealt to enemies increase healing opportunities

## Strategy Notes
- **Early Game**: Provides steady healing and minor crit boost
- **Late Game**: Heal chance scales well with multiple stacks (5 stacks = 125% heal chance, likely capped at 100%)
- **Playstyle**: Best for aggressive players who deal damage frequently
- **Risk/Reward**: Healing depends on dealing damage to enemies, encouraging offensive play

---

*Data extracted from IL2CPP constructor at 0x180407420 and decompiled C# class structure*