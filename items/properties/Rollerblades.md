# Rollerblades

## Overview
- **Item ID**: EItem.Rollerblades
- **Constructor Address**: 0x180422F10
- **Category**: Movement/Attack Speed
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| maxAttackSpeedPerAmount | float | 0.4 | 40% max attack speed per stack |
| updateStatsInterval | float | 0.25 | Update frequency in seconds |
| cap | float | Dynamic | Calculated as amount * maxAttackSpeedPerAmount |
| nextUpdateTime | float | Dynamic | Next scheduled update time |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 15 | Attack Speed | amount * cap | Dynamic scaling based on movement |

## Special Mechanics
- **Movement-Based Attack Speed**: Attack speed bonus is dynamically calculated based on the player's current movement speed relative to their base movement speed
- **Speed Ratio Formula**: `speedRatio = (currentHorizontalSpeed / baseMovementSpeed) - 1.0`
- **Clamping**: Speed ratio is clamped between 0.0 and the item's cap value
- **Final Attack Speed**: `finalAttackSpeed = amount * clamp(speedRatio, 0.0, cap)`

## Formulas
### Speed Ratio Calculation
```
speedRatio = (PlayerMovement.GetSpeedHorizontal() / Player.baseMovementSpeed) - 1.0
speedRatio = clamp(speedRatio, 0.0, cap)
cap = amount * maxAttackSpeedPerAmount
```

### Final Attack Speed Bonus
```
attackSpeedBonus = amount * speedRatio
```

## Implementation Details
- **Update Frequency**: 0.25 seconds (4 times per second)
- **Event Subscriptions**: None - uses Tick() for periodic updates
- **Stack Behavior**: Each stack increases both the maximum possible attack speed bonus (cap) and the scaling factor

## C# Pseudocode
```csharp
// Constructor logic
public ItemRollerblades(ItemInventory itemInventoryRef) : base(itemInventoryRef)
{
    maxAttackSpeedPerAmount = 0.4f;
    updateStatsInterval = 0.25f;
}

// OnInitOrAmountChanged - called when item amount changes
protected override void OnInitOrAmountChanged()
{
    cap = amount * maxAttackSpeedPerAmount;
}

// Tick - called every frame, but only updates every updateStatsInterval
public override void Tick()
{
    if (nextUpdateTime <= MyTime.time)
    {
        nextUpdateTime = MyTime.time + updateStatsInterval;

        float currentSpeed = Player.Instance.orientation.GetSpeedHorizontal();
        float baseSpeed = Player.Instance.baseMovementSpeed;

        float speedRatio = (currentSpeed / baseSpeed) - 1.0f;
        speedRatio = Mathf.Clamp(speedRatio, 0.0f, cap);

        StatModifier attackSpeedMod = new StatModifier();
        attackSpeedMod.eStatType = EStat.AttackSpeed; // ID 15
        attackSpeedMod.modificationType = ModificationType.Additive; // Type 2
        attackSpeedMod.value = amount * speedRatio;

        SetStat(attackSpeedMod);
    }
}
```

## Technical Notes
- **Performance Consideration**: Updates only 4 times per second to avoid excessive calculations
- **Movement Detection**: Uses PlayerMovement.GetSpeedHorizontal() to measure current movement speed
- **Base Speed Reference**: Compares against Player.baseMovementSpeed (stored at offset 33 in player data)
- **Stat Application**: Uses EStat ID 15 (Attack Speed) with additive modification type (2)

## Related Items
- **TurboSocks**: Provides flat movement speed bonus (EStat 25)
- **FlappyFeathers**: Provides jump-based speed boost and jump height
- **PhantomShroud**: Provides evasion-based speed boost
- **CowardsCloak**: Provides damage-triggered speed boost

---

*Generated from decompiled IL2CPP constructor at 0x180422F10 and C# class definition*