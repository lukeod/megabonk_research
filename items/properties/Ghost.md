# Ghost

## Overview
- **Item ID**: 28 (EItem.Ghost)
- **Constructor Address**: 0x180457DF0
- **Category**: Utility/Special
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| numGhostsPerAmount | int | 6 | Number of ghosts spawned per item stack |
| damageSource | string | "Ghost" | Damage attribution string for ghost projectiles |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | N/A | N/A | No direct stat modifications |

## Special Mechanics
- **Interaction-Based**: Triggers ghost spawning when player interacts with interactable objects
- **Event Subscription**: Subscribes to `DetectInteractables.A_Interacted` event during Init()
- **Ghost Spawning**: Calls `SpawnGhost()` method when interaction events fire
- **Dynamic Ghost Count**: Total ghosts = `amount * numGhostsPerAmount` (calculated in OnInitOrAmountChanged)

## Formulas
- **Total Ghosts**: `amount * 6`
- **Damage**: Calculated by `GetDamage()` method (implementation in native code)
- **Duration**: Calculated by `GetDuration()` method (implementation in native code)

## Implementation Details
- **Update Frequency**: Event-driven (no periodic updates)
- **Event Subscriptions**: DetectInteractables interaction events
- **Stack Behavior**: Linear scaling - each stack adds 6 more ghosts
- **Cleanup**: Properly unsubscribes from interaction events on cleanup

## C# Pseudocode
```csharp
// Simplified constructor logic
public ItemGhost(ItemInventory itemInventoryRef) : base(itemInventoryRef)
{
    this.numGhostsPerAmount = 6;
    this.damageSource = "Ghost"; // EItem.Ghost.ToString()
}

// Amount change handler
protected override void OnInitOrAmountChanged()
{
    this.numGhosts = this.amount * this.numGhostsPerAmount;
}

// Initialization - subscribe to interaction events
public override void Init()
{
    DetectInteractables.A_Interacted += OnInteracted;
}

// Cleanup - unsubscribe from events
public override void Cleanup()
{
    DetectInteractables.A_Interacted -= OnInteracted;
}

// Handle interaction events
private void OnInteracted(BaseInteractable interactable, bool success)
{
    if (success)
    {
        SpawnGhost();
    }
}
```

## Technical Notes
- Ghost spawning mechanics, damage calculation, and duration are implemented in native IL2CPP code
- The item has minimal constructor setup compared to other items
- Uses Unity's event system for reactive behavior rather than tick-based updates
- No stat modifications or passive effects - purely interaction-driven
- Ghost projectiles likely have their own damage, speed, and behavior properties defined elsewhere

## Related Items
- **BobDead**: Also spawns ghost projectiles but based on movement rather than interactions
- **SoulHarvester**: Another projectile-spawning item triggered by enemy deaths

---

**Data Sources**:
- megabonk_research/items.md
- decompiled/Assembly-CSharp/Assets.Scripts.Inventory__Items__Pickups.Items.ItemImplementations/ItemGhost.cs
- extracted_constructors/items/Ghost.c
- decompiled/Assembly-CSharp/Assets.Scripts.Inventory__Items__Pickups.Items/EItem.cs