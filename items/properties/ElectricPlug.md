# ElectricPlug

## Overview
- **Item ID**: 45 (EItem.ElectricPlug)
- **Constructor Address**: 0x180455FB0
- **Category**: Elemental/Status (Lightning-based defensive item)
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| damageSource | string | "ElectricPlug" | Damage source identifier |
| reuseDc | DamageContainer | 0.5 damage, "ElectricPlug" source | Reusable damage container |
| radius | float | 13.0 | Base chain lightning radius |
| radiusPerAmount | float | 4.0 | Radius increase per stack |
| targets | int | 15 | Base number of targets |
| targetsPerAmount | int | 4 | Additional targets per stack |
| targetsDefault | int | 6 | Default target count at 1 stack |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| N/A | N/A | N/A | No direct stat modifications |

## Special Mechanics
- **Chain Lightning on Damage**: Triggers when the player takes damage
- **Area Lightning**: Creates chain lightning effect that hits multiple enemies within radius
- **Event-Driven**: Subscribes to `PlayerHealth.A_DamagePlayerCalled` event
- **Multi-Target**: Can hit up to `targets` number of enemies per activation

## Formulas
- **Final Radius**: `(EStat9 * radiusPerAmount) + 12.0`
- **Final Targets**: `targetsDefault + targetsPerAmount * (amount - 1)` = `6 + 4 * (amount - 1)`
- **Base Damage**: 0.5 (from reuseDc container)

## Implementation Details
- **Update Frequency**: Event-driven (triggers on player damage)
- **Event Subscriptions**: `PlayerHealth.A_DamagePlayerCalled`
- **Stack Behavior**: Radius and target count scale linearly with amount

## C# Pseudocode
```csharp
// Simplified constructor logic
public ItemElectricPlug(ItemInventory itemInventoryRef)
{
    // Set damage source
    damageSource = "ElectricPlug";

    // Create reusable damage container
    reuseDc = new DamageContainer(0.5f, "ElectricPlug");

    // Set base properties
    radius = 13.0f;
    radiusPerAmount = 4.0f;
    targets = 15;
    targetsPerAmount = 4;
    targetsDefault = 6;

    base(itemInventoryRef);
}

// OnInitOrAmountChanged logic
protected override void OnInitOrAmountChanged()
{
    float radiusMultiplier = PlayerStats.GetStat(EStat.AreaMultiplier); // EStat 9
    radius = (radiusMultiplier * radiusPerAmount) + 12.0f;
    targets = targetsDefault + targetsPerAmount * (amount - 1);
}

// Init logic
public override void Init()
{
    // Subscribe to player damage event
    PlayerHealth.A_DamagePlayerCalled += OnPlayerHit;
}

// Cleanup logic
public override void Cleanup()
{
    // Unsubscribe from player damage event
    PlayerHealth.A_DamagePlayerCalled -= OnPlayerHit;
}

// OnPlayerHit logic (inferred)
private void OnPlayerHit()
{
    // Create chain lightning effect
    // Hit up to 'targets' enemies within 'radius'
    // Deal damage using reuseDc (0.5 base damage)
}
```

## Technical Notes
- **Defensive Item**: Activates in response to taking damage, providing retaliation
- **EStat 9 Scaling**: Radius scales with Area/Radius Multiplier stat
- **Lightning Debuff**: Likely applies lightning debuff (ID 8) to affected enemies
- **Target Scaling**: Formula is `targetsDefault + targetsPerAmount * (amount - 1)` which gives 6 targets at 1 stack, 10 at 2, 14 at 3, etc.
- **Event System**: Uses Unity's delegate system for event handling

## Related Items
- **LightningOrb**: Another lightning-based offensive item
- **GlovesLightning**: Lightning-based gloves with area damage
- **Mirror**: Similar defensive retaliation mechanic
- **Cactus**: Counter-attack item that triggers on player damage

---

*Documentation generated from extracted IL2CPP constructor at 0x180455FB0 and decompiled C# class*