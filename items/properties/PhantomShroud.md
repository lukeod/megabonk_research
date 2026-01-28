# PhantomShroud

## Overview
- **Item ID**: PhantomShroud (from EItem enumeration)
- **Constructor Address**: 0x180462710
- **Category**: Movement/Defensive
- **Rarity**: Unknown (not determinable from decompiled code)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| evasionPerAmount | float | 0.05 | 5% evasion per stack |
| damageMultiplierBase | float | 2.0 | Base damage multiplier when evading |
| damageMultiplierPerAmount | float | 0.5 | Additional damage multiplier per stack |
| speedAdditionBase | float | 0.5 | Base speed addition when evading |
| speedAdditionPerAmount | float | 0.15 | Additional speed per stack |
| timeout | float | 2.0 | Initial timeout value (dynamically calculated) |
| attackSpeedPerStack | float | 0.25 | Attack speed bonus per phantom stack |
| damagePerStack | float | 0.5 | Damage bonus per phantom stack |
| maxStacks | int | amount * 4 | Maximum phantom stacks (4 per item) |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 5 | Evasion | amount * 0.05 | Linear per stack |

## Special Mechanics

### Evasion-Based Activation
- Subscribes to the PlayerHealth.A_Evaded event
- When player evades an attack, triggers OnEvade() method
- Sets hasEvaded flag to true for next attack

### Phantom Stacks System
- Gains phantom stacks when evading (up to maxStacks = amount * 4)
- Each phantom stack provides temporary bonuses
- Stacks decay over time based on timeout value

### Temporary Speed Boost
- On evade, grants movement speed bonus for limited duration
- Speed boost = speedAdditionBase + (amount * speedAdditionPerAmount)
- Speed boost duration controlled by timeout value
- Automatically removes speed boost when timer expires

### Damage Amplification
- Next attack after evading gets damage multiplier
- Damage multiplier = damageMultiplierBase + ((amount - 1) * damageMultiplierPerAmount)
- Formula: 2.0 + ((stacks - 1) * 0.5)

## Formulas

### Timeout Calculation
```
timeout = (amount - 1) * 0.5 + 3.0
if (timeout > 6.0) timeout = 6.0
```
- Minimum: 3.0 seconds (1 stack)
- Maximum: 6.0 seconds (capped)

### Evasion Stat
```
evasion = amount * 0.05
```
- Linear scaling: 5% per stack

### Damage Multiplier on Evade
```
damageMultiplier = 2.0 + ((amount - 1) * 0.5)
```
- 1 stack: 2.0x damage
- 2 stacks: 2.5x damage
- 3 stacks: 3.0x damage

### Speed Addition on Evade
```
speedAddition = 0.5 + (amount * 0.15)
```
- Dynamic speed boost based on stacks

## Implementation Details
- **Update Frequency**: Tick() method called every frame
- **Event Subscriptions**: PlayerHealth.A_Evaded (subscribed in Init, unsubscribed in Cleanup)
- **Stack Behavior**: Phantom stacks are temporary and decay over time
- **State Management**: Uses hasEvaded, hasSpeed, and speedResetAtTime flags

## C# Pseudocode
```csharp
// Constructor logic
public ItemPhantomShroud(ItemInventory itemInventoryRef) : base(itemInventoryRef)
{
    evasionPerAmount = 0.05f;
    damageMultiplierBase = 2.0f;
    damageMultiplierPerAmount = 0.5f;
    speedAdditionBase = 0.5f;
    speedAdditionPerAmount = 0.15f;
    timeout = 2.0f;
    attackSpeedPerStack = 0.25f;
    damagePerStack = 0.5f;
}

// On amount change
protected override void OnInitOrAmountChanged()
{
    maxStacks = amount * 4;
    timeout = Math.Min((amount - 1) * 0.5f + 3.0f, 6.0f);
    SetStat(EStat.Evasion, amount * evasionPerAmount);
}

// Event handler for evasion
private void OnEvade(Enemy enemy)
{
    hasEvaded = true;
    stacks = Math.Min(stacks + 1, maxStacks);

    // Apply speed boost
    hasSpeed = true;
    speedResetAtTime = MyTime.time + timeout;

    // Apply temporary movement speed stat
    SetStat(EStat.MovementSpeed, speedAdditionBase + (amount * speedAdditionPerAmount));

    // Apply temporary attack speed and damage from phantom stacks
    SetStat(EStat.AttackSpeed, stacks * attackSpeedPerStack);
    SetStat(EStat.Power, stacks * damagePerStack);
}

// Frame update
public override void Tick()
{
    if (hasSpeed && MyTime.time > speedResetAtTime)
    {
        // Remove temporary bonuses
        SetStat(EStat.MovementSpeed, 0);
        SetStat(EStat.Power, 0);
        SetStat(EStat.AttackSpeed, 0);
        hasSpeed = false;
        stacks = 0;
    }
}

// Pre-attack modifier
public override void PreAttack(DamageContainer dc, StatComponents itemAttackModifier)
{
    if (hasEvaded)
    {
        hasEvaded = false;
        float damageMultiplier = damageMultiplierBase + ((amount - 1) * damageMultiplierPerAmount);
        itemAttackModifier.AddMultiplier(damageMultiplier);
    }
}
```

## Technical Notes
- Uses Unity's event system for evasion detection
- Implements time-based temporary effects that auto-expire
- Phantom stacks provide temporary attack speed and damage bonuses
- Next attack after evading gets significant damage amplification
- Speed boost duration scales with item stacks but caps at 6 seconds
- Clean event subscription/unsubscription in Init/Cleanup methods

## Performance Considerations
- Tick() method runs every frame when speed boost is active
- Event-driven evasion detection is efficient
- Temporary stat modifications are properly cleaned up

## Related Items
- **CowardsCloak**: Also provides speed boost when taking damage
- **TurboSocks**: Provides movement speed (but permanent, not triggered)
- **Mirror**: Another defensive item that responds to taking damage
- **SpikyShield**: Defensive item with retaliation mechanics

---

**Data Sources Used:**
- D:\dev\megabonk\megabonk_research\items.md (PhantomShroud section)
- D:\dev\megabonk\extracted_constructors\items\PhantomShroud.c (decompiled constructor)
- D:\dev\megabonk\decompiled\Assembly-CSharp\Assets.Scripts.Inventory__Items__Pickups.Items.ItemImplementations\ItemPhantomShroud.cs
- D:\dev\megabonk\decompiled\Assembly-CSharp\Assets.Scripts.Inventory__Items__Pickups.Items\ItemBase.cs