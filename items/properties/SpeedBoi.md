# SpeedBoi

## Overview
- **Item ID**: 56 (EItem.SpeedBoi)
- **Constructor Address**: 0x180425570
- **Category**: Time Manipulation / Defensive Survival
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| damageMultiplierDuringFreeze | float | 2.0 | Damage multiplier during time slow |
| durationPerAmount | float | 2.0 | Duration scaling per stack |
| normalCooldown | float | 10.0 | Base cooldown between activations |
| slowdownHpRatio | float | 0.5 | HP threshold to trigger (50%) |
| duration | float | calculated | Final duration based on amount |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | - | - | No direct stat modifications |

## Special Mechanics
- **Time Slowdown Trigger**: Automatically activates when player HP drops below 50%
- **Duration Calculation**: `duration = clamp(amount * 2.0 + 8.0, 0, 15.0)`
  - Base duration: 8 seconds
  - Per stack: +2 seconds
  - Maximum: 15 seconds
- **Damage Boost**: 2x damage multiplier during time slowdown effect
- **Cooldown System**: 10 second cooldown between activations

## Formulas
- **Slowdown Duration**: `min(15, max(0, amount * 2 + 8))`
- **HP Threshold**: `currentHP / maxHP < 0.5`
- **Damage During Slowdown**: `baseDamage * 2.0`

## Implementation Details
- **Update Frequency**: Event-driven (responds to damage taken)
- **Event Subscriptions**:
  - PlayerHealth.A_TakeDamage (OnTakeDamage)
  - MyTime.A_TimeScaleChange (RefreshTimeScale)
  - GameManager.A_StageStarted (ResetStats)
- **Stack Behavior**: Duration increases linearly with stacks, capped at 15 seconds

## C# Pseudocode
```csharp
// Constructor logic
public ItemSpeedBoi(ItemInventory itemInventoryRef) : base(itemInventoryRef)
{
    damageMultiplierDuringFreeze = 2.0f;
    durationPerAmount = 2.0f;
    normalCooldown = 10.0f;
    slowdownHpRatio = 0.5f;
}

// Duration calculation on amount change
protected override void OnInitOrAmountChanged()
{
    float calculatedDuration = amount * durationPerAmount + 8.0f;
    duration = Mathf.Clamp(calculatedDuration, 0f, 15.0f);
}

// Trigger condition
private void OnTakeDamage(PlayerHealth ph, DamageContainer dc, bool shieldDamage)
{
    if (ph.currentHP / ph.maxHP < slowdownHpRatio &&
        Time.time >= slowdownReadyAtTime)
    {
        Slowdown();
        slowdownReadyAtTime = Time.time + normalCooldown;
    }
}

// Damage modification during slowdown
public override void PreAttack(DamageContainer dc, StatComponents itemAttackModifier)
{
    if (timeSlowActive)
    {
        dc.damage *= damageMultiplierDuringFreeze;
    }
}
```

## Technical Notes
- Uses Unity's event system for reactive behavior
- Time scale manipulation affects game-wide time flow
- Subscribes to multiple event systems during Init(), unsubscribes during Cleanup()
- Static Action field A_Slowdown suggests coordination with other time-affecting systems
- RefreshTimeScale method indicates dynamic time scale adjustments

## Related Items
- **ZaWarudo**: Also manipulates time (though implementation differs)
- **TimeBracelet**: Time-based damage mechanics
- **Defensive Items**: Complements other survival items like shields and armor

---

*Data extracted from decompiled IL2CPP constructor at address 0x180425570 and C# interface analysis*