# ZaWarudo

## Overview
- **Item ID**: EItem.ZaWarudo
- **Constructor Address**: 0x1803FFD80
- **Static Constructor Address**: 0x180427640
- **Category**: Time Manipulation/Utility
- **Rarity**: Legendary (inferred from unique time-stop mechanic and JoJo reference)

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| freezeTime | static float | 15.0 | Duration of time freeze effect in seconds |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | N/A | N/A | No direct stat modifications |

## Special Mechanics
ZaWarudo is a unique item that implements time manipulation through a global static freeze time system rather than traditional item mechanics. Unlike other items, it has no constructor properties and all its functionality is handled through external systems and static state.

The item's name is a reference to Dio Brando's Stand "The World" from JoJo's Bizarre Adventure, which has the ability to stop time.

## Formulas
- **Freeze Duration**: Fixed at `15.0 seconds` (static value)
- **Activation**: Unknown mechanism (likely external trigger system)
- **Stacking Behavior**: Unknown (no amount-based scaling implemented)

## Implementation Details
- **Update Frequency**: N/A - no dynamic updates
- **Event Subscriptions**: None in constructor
- **Stack Behavior**: No amount-based scaling (static implementation)
- **Static Field**: `freezeTime` is shared across all instances globally

## C# Pseudocode
```csharp
public class ItemZaWarudo : ItemBase
{
    public static float freezeTime = 15.0f; // Set in static constructor

    // Static constructor sets freeze time
    static ItemZaWarudo()
    {
        freezeTime = 15.0f; // 0x41700000 in hex
    }

    // Empty constructor - no instance properties
    public ItemZaWarudo(ItemInventory itemInventoryRef) : base(itemInventoryRef)
    {
        // No initialization logic
    }

    // All virtual methods are empty - implementation handled elsewhere
    protected override void OnInitOrAmountChanged() { }
    public override void Init() { }
    public override void Cleanup() { }
    public override void Tick() { }
    public override void PreAttack(DamageContainer dc, StatComponents itemAttackModifier) { }
    public override void ProcOnHitEffects(DamageContainer dc) { }
}
```

## Technical Notes
- **Unique Implementation**: ZaWarudo is the only item with completely empty method implementations
- **Static State Management**: Uses a global static `freezeTime` field rather than instance-based properties
- **External Integration**: The actual time-stop functionality is likely implemented in:
  - Game time management systems
  - External powerup/effect systems
  - Unity Time.timeScale manipulation
- **No Direct Scaling**: Unlike other items, stack count (`amount`) doesn't affect the freeze duration
- **Performance**: Minimal overhead as all methods are empty stubs

## Related Items
- **SpeedBoi**: Also manipulates time scale with damage multipliers during slow effects
- **TimeBracelet**: Time-based damage mechanics (EStat 32)
- **IceCrystal/IceCube**: Apply freeze debuffs to enemies (different from global time freeze)

## Research Notes
The implementation suggests that ZaWarudo's time-stop effect is handled by:
1. **Static Field Access**: External systems read `ItemZaWarudo.freezeTime` (15.0 seconds)
2. **Global Time Management**: Likely integrates with Unity's Time.timeScale or custom time manager
3. **Activation Trigger**: Unknown mechanism (possibly keybind, ability system, or automatic trigger)
4. **Item Detection**: Game systems probably check if player has ZaWarudo in inventory

## Speculation on Missing Implementation
Based on the pattern of other items and the static field, the likely implementation involves:
- External systems checking for ZaWarudo in player inventory
- On activation, setting game time scale to 0 or very low value
- Timer countdown using the static `freezeTime` value (15 seconds)
- Restoration of normal time flow after duration expires

---

*Data sources: extracted_constructors/items/ZaWarudo.c, decompiled C# class ItemZaWarudo.cs, IDA Pro static constructor decompilation, megabonk_research/items.md*