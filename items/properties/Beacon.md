# Beacon

## Overview
- **Item ID**: EItem.Beacon (from enum)
- **Constructor Address**: 0x180400920
- **Category**: Utility/Environmental Enhancement
- **Rarity**: Unknown (not determinable from constructor code)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| extraShrinesPerAmount | int | 2 | Extra shrines spawned per stack |
| healingRadiusPerAmount | float | 2.0 | Healing radius increase per stack |
| healingFractionPerInterval | float | 0.025 | Base healing fraction (2.5%) per interval |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | N/A | N/A | No direct stat modifications |

## Special Mechanics

### Shrine Generation
- **Extra Shrines**: Each stack of Beacon adds 2 additional shrines to level generation
- **Function**: `GetExtraShrines()` returns `amount * extraShrinesPerAmount` (amount * 2)
- **Impact**: Increases shrine encounter opportunities for stat upgrades and rewards

### Healing Aura
- **Healing Radius**: Area of effect healing around the player
- **Radius Calculation**: `GetRadius()` returns `amount * healingRadiusPerAmount` (amount * 2.0)
- **Healing Rate**: `GetHealingPerInterval()` returns `healingFractionPerInterval` (0.025 or 2.5%)
- **Mechanism**: Likely provides periodic healing to the player or allies within the radius

## Formulas

### Shrine Generation
```
extraShrines = amount * 2
```

### Healing Mechanics
```
healingRadius = amount * 2.0
healingPerInterval = 0.025 (constant, independent of stacks)
```

## Implementation Details
- **Update Frequency**: Implements `Tick()` method suggesting periodic updates
- **Event Subscriptions**: Inherits standard ItemBase event system
- **Stack Behavior**: Linear scaling for both shrine generation and healing radius
- **Healing Type**: Fractional healing (percentage-based rather than flat amount)

## C# Pseudocode
```csharp
public class ItemBeacon : ItemBase
{
    private int extraShrinesPerAmount = 2;
    private float healingRadiusPerAmount = 2.0f;
    private float healingFractionPerInterval = 0.025f; // 2.5%

    public int GetExtraShrines()
    {
        return amount * extraShrinesPerAmount; // amount * 2
    }

    public float GetRadius()
    {
        return amount * healingRadiusPerAmount; // amount * 2.0
    }

    public float GetHealingPerInterval()
    {
        return healingFractionPerInterval; // Always 0.025 (2.5%)
    }

    // Periodic healing logic likely implemented in Tick()
    public override void Tick()
    {
        // Implementation handles healing aura mechanics
        // Heals player/allies within GetRadius() range
        // by GetHealingPerInterval() fraction of max health
    }
}
```

## Technical Notes

### Shrine Integration
- Beacon modifies level generation to include additional shrine encounters
- Shrines provide stat upgrade opportunities through the EncounterUtility system
- Each stack adds exactly 2 more shrines, providing significant progression value

### Healing Implementation
- Uses fractional healing (2.5% of max health) rather than flat healing amounts
- Healing rate is constant regardless of stack count - only radius increases
- Likely operates on a fixed interval (implementation hidden in native IL2CPP code)

### Performance Considerations
- Tick-based implementation may have performance impact with multiple Beacon stacks
- Healing radius collision detection scales with stack count
- Shrine generation occurs during level creation, not during gameplay

## Related Items

### Similar Utility Items
- **Campfire**: Also provides area-based healing with setup mechanics
- **HolyBook**: Provides direct HP regeneration stat bonuses
- **Key**: Another item that affects encounter/reward generation

### Synergistic Items
- **Items with HP scaling**: Beacon's healing benefits scale with maximum health
- **Area of effect items**: May benefit from increased shrine opportunities for radius modifiers
- **Healing enhancement items**: Could potentially amplify Beacon's healing effectiveness

---

**Data Sources:**
- D:\dev\megabonk\megabonk_research\items.md (Beacon section lines 56-66)
- D:\dev\megabonk\extracted_constructors\items\Beacon.c (Constructor implementation)
- D:\dev\megabonk\decompiled\Assembly-CSharp\Assets.Scripts.Inventory__Items__Pickups.Items.ItemImplementations\ItemBeacon.cs (Class structure)
- D:\dev\megabonk\decompiled\Assembly-CSharp\Assets.Scripts.Inventory__Items__Pickups.Items\ItemBase.cs (Base class architecture)