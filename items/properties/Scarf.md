# Scarf

## Overview
- **Item ID**: EItem.Scarf
- **Constructor Address**: 0x180463DC0
- **Category**: Damage/Power
- **Rarity**: Common

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| damageAddPerAmount | float | 0.5 | 50% damage increase per stack |
| damageAdd | float | Calculated | Current active damage bonus |
| lastValueSet | float | Tracked | Last damage value applied to prevent redundant updates |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 12 | Power/Damage | damageAdd (conditional) | Linear per stack, conditional on grounded state |

## Special Mechanics
- **Grounded State Dependency**: Only provides damage bonus when the player is **NOT** grounded (airborne)
- **Event-Driven Updates**: Subscribes to PlayerMovement.A_Grounded events to track grounded state changes
- **Conditional Stat Application**: Damage bonus is dynamically applied/removed based on grounded state
- **Performance Optimization**: Uses lastValueSet tracking to prevent redundant stat updates

## Formulas
- **Damage Calculation**: `damageAdd = amount * damageAddPerAmount * (grounded ? 0 : 1)`
- **Final Damage Bonus**: `amount * 0.5` when airborne, `0` when grounded
- **Per Stack Value**: 50% damage increase per stack (only when airborne)

## Implementation Details
- **Update Frequency**: Event-driven (on grounded state change)
- **Event Subscriptions**: PlayerMovement.A_Grounded (bool grounded)
- **Stack Behavior**: Linear scaling - each stack adds 50% damage when airborne
- **Stat Management**: Uses EStat 12 (Power/Damage) with conditional application

## C# Pseudocode
```csharp
// Constructor logic
public ItemScarf(ItemInventory itemInventoryRef) : base(itemInventoryRef)
{
    damageAddPerAmount = 0.5f;
}

// Initialization - subscribe to grounded events
public override void Init()
{
    PlayerMovement.A_Grounded += OnGroundedChange;
}

// Cleanup - unsubscribe from events
public override void Cleanup()
{
    PlayerMovement.A_Grounded -= OnGroundedChange;
}

// Amount change handler
protected override void OnInitOrAmountChanged()
{
    damageAdd = amount * damageAddPerAmount;
    if (MyPlayer.Instance?.playerMovement != null)
        UpdateDamage(MyPlayer.Instance.playerMovement.grounded);
}

// Grounded state change handler
private void OnGroundedChange(bool grounded)
{
    UpdateDamage(grounded);
}

// Core damage update logic
private void UpdateDamage(bool grounded)
{
    float newDamageValue = grounded ? 0f : damageAdd;

    if (newDamageValue != lastValueSet)
    {
        lastValueSet = newDamageValue;
        SetStat(new StatModifier
        {
            statType = EStat.Power, // EStat 12
            flatAdditive = newDamageValue
        });
    }
}
```

## Technical Notes
- **Memory Efficiency**: Tracks lastValueSet to avoid redundant stat calculations and Unity event spam
- **Event Safety**: Properly subscribes/unsubscribes to prevent memory leaks
- **State Validation**: Checks for null player instances before accessing grounded state
- **IL2CPP Compatibility**: Uses native IL2CPP event system integration
- **Performance Impact**: Minimal - only updates when grounded state actually changes

## Related Items
- **FlappyFeathers**: Also provides airborne-related bonuses (jump height + speed on jump)
- **EagleClaw**: Damage bonus when enemies are airborne (inverse mechanic)
- **PhantomShroud**: Evasion-based item that can trigger when airborne due to mobility

## Usage Strategy
- **Best When**: Used by highly mobile characters who spend significant time airborne
- **Synergies**: Combines well with jump height increases, movement abilities, and platforming-heavy gameplay
- **Stacking Value**: Each additional stack provides consistent 50% damage increase when airborne
- **Risk/Reward**: Encourages risky aerial gameplay for damage bonus

---

**Data Sources**:
- megabonk_research/items.md (lines 923-934)
- extracted_constructors/items/Scarf.c (full constructor and method implementations)
- decompiled/Assembly-CSharp/Assets.Scripts.Inventory__Items__Pickups.Items.ItemImplementations/ItemScarf.cs
- IDA Pro MCP decompilation of OnGroundedChange (0x1804232C0) and UpdateDamage (0x1804233D0)