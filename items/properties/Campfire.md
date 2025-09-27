# Campfire

## Overview
- **Item ID**: Not specified (from decompiled C#)
- **Constructor Address**: 0x1804048B0
- **Category**: Utility/Healing
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| healthRegenPerMinutePerAmount | float | 1100.0 | Health regen per minute per stack |
| setupTime | float | 0.6 | Setup time in seconds |
| distThreshold | float | 1.75 | Activation distance threshold |
| healthRegen | float | Dynamic | Calculated regen amount |
| campfire | Campfire | Reference | Visual campfire object |
| campfirePos | Vector3 | Dynamic | Player position when campfire placed |
| startCampfireTime | float | Dynamic | Time when campfire timer started |
| isCampActive | bool | Dynamic | Whether campfire is currently active |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| 1 | HP Regeneration | amount * 1100.0 / 60.0 | Linear per stack (when active) |

## Special Mechanics
The Campfire item implements a proximity-based health regeneration system that requires the player to remain stationary within a specific area:

### Activation Sequence
1. **Position Recording**: When player moves outside the distance threshold (1.75 units), the current position is recorded as `campfirePos`
2. **Setup Timer**: A setup timer (`startCampfireTime`) is set to current time + 0.6 seconds
3. **Proximity Check**: Player must remain within 1.75 units of the recorded position
4. **Activation**: After 0.6 seconds of remaining within range, the campfire activates

### Active State Behavior
- Creates a visual `Campfire` object with particle effects and audio
- Applies health regeneration stat modifier (EStat ID 1)
- Continues regenerating as long as player stays within range

### Deactivation
- Moving outside the 1.75 unit threshold immediately deactivates the campfire
- Removes the visual campfire object via `EndFire()`
- Removes the health regeneration stat modifier

## Formulas
- **Health Regeneration Rate**: `amount * 1100.0` health per minute
- **Per-Second Regeneration**: `(amount * 1100.0) / 60.0` ≈ `amount * 18.33` health per second
- **Distance Check**: `sqrt((pos.x - campfirePos.x)² + (pos.y - campfirePos.y)² + (pos.z - campfirePos.z)²) <= 1.75`

## Implementation Details
- **Update Frequency**: Every tick (frame-based via `Tick()` method)
- **Event Subscriptions**: None (purely tick-based)
- **Stack Behavior**: Each stack increases regeneration linearly by 1100 HP/minute

### State Management
The item tracks several internal states:
- **Position Recording**: Updates `campfirePos` when player moves too far
- **Timer Management**: Uses `MyTime.time` for setup delay
- **Visual Management**: Creates/destroys `Campfire` GameObject as needed

## C# Pseudocode
```csharp
public class ItemCampfire : ItemBase
{
    // Constructor sets base properties
    public ItemCampfire()
    {
        healthRegenPerMinutePerAmount = 1100.0f;
        setupTime = 0.6f;
        distThreshold = 1.75f;
    }

    public override void Tick()
    {
        Vector3 playerPos = MyPlayer.Instance.transform.position;
        float distance = Vector3.Distance(playerPos, campfirePos);

        if (distance <= distThreshold)
        {
            // Player is within range
            if (!isCampActive && MyTime.time > startCampfireTime)
            {
                // Activate campfire after setup time
                CreateCamp();
                SetStat(new StatModifier(EStat.HPRegen, amount * healthRegenPerMinutePerAmount / 60f));
                isCampActive = true;
            }
        }
        else
        {
            // Player moved too far - reset
            campfirePos = playerPos;
            startCampfireTime = MyTime.time + setupTime;

            if (isCampActive)
            {
                // Deactivate campfire
                campfire.EndFire();
                SetStat(new StatModifier(EStat.HPRegen, 0)); // Remove regen
                isCampActive = false;
            }
        }
    }
}
```

## Technical Notes
- Uses Unity's `MonoBehaviour.Update()` pattern via the `Tick()` method
- Position tracking uses 3D distance calculation with `sqrt()` operation
- Visual campfire includes particle effects (`ParticleSystem`) and audio (`AudioSource`)
- The campfire object has animation scaling and random sound effects
- Implementation encourages defensive/stationary playstyles by requiring the player to stay in one location

## Related Items
- **IdleJuice**: Similar proximity-based mechanics but for damage accumulation
- **Medkit**: Direct HP regeneration without proximity requirements
- **HolyBook**: Provides HP regeneration as a passive stat bonus
- **Beacon**: Has proximity-based healing mechanics but for shrines

---

*Data sources: megabonk_research/items.md, extracted_constructors/items/Campfire.c, decompiled C# classes ItemCampfire.cs and Campfire.cs*