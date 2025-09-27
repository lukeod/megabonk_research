# LightningOrb

## Overview
- **Item ID**: EItem.LightningOrb (17)
- **Constructor Address**: 0x180415900
- **Category**: Elemental/Status (Lightning)
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| procChancePerAmount | float | 0.25 | 25% base proc chance per stack |
| stunChancePerAmount | float | 0.25 | 25% base stun chance per stack |
| baseRadius | float | 40.0 | Fixed detection radius for enemies |
| damageRatio | float | 0.4 | Base damage ratio (40% of original damage) |
| damageRatioPerAmount | float | 0.4 | Additional 40% damage ratio per stack |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| N/A | No direct stat modifications | N/A | Item uses proc-based mechanics |

## Special Mechanics

### Lightning Strike System
- **Trigger**: Procs on hit based on `procChance`
- **Target Selection**: Random enemy within `baseRadius` (40.0 units) of player
- **Damage Calculation**: `originalDamage * damageRatio`
- **Element**: Lightning (element = 1)
- **Visual Effect**: Lightning strike with area scaling based on EStat 9 (Area/Radius Multiplier)

### Stun Mechanics
- **Dual Stun Chances**:
  - Direct hit stun: Applied when original attack has lightning element (element == 1)
  - Lightning strike stun: Applied when lightning proc occurs
- **Stun Duration**: 3.0 seconds
- **Debuff Type**: Lightning/Stun (ID: 8)

### Enemy Caching System
- **Cache Duration**: Enemies found within radius are cached until `foundEnemiesAtTime` expires
- **Performance Optimization**: Avoids recalculating enemy list on every proc
- **Dead Enemy Handling**: Removes dead enemies from cache using efficient list swapping

## Formulas

### Proc Chance (Hyperbolic Scaling)
```
procChance = (amount * procChancePerAmount) / (amount * procChancePerAmount + 0.6) * 0.9
```
- **Factor**: 0.6
- **Cap**: 90% (0.9 multiplier)
- **Per Stack**: 25% base contribution

### Stun Chance (Hyperbolic Scaling)
```
stunChance = (amount * stunChancePerAmount) / (amount * stunChancePerAmount + 0.6) * 0.9
```
- **Factor**: 0.6
- **Cap**: 90% (0.9 multiplier)
- **Per Stack**: 25% base contribution

### Damage Calculation
```
finalDamage = originalDamage * (amount * damageRatioPerAmount)
lightningDamage = finalDamage * (EStat9 * 8.0)
```
- **Base Multiplier**: 40% per stack
- **Area Scaling**: EStat 9 (Area/Radius Multiplier) * 8.0

## Implementation Details
- **Update Frequency**: On-hit (event-driven)
- **Event Subscriptions**: ProcOnHitEffects virtual method
- **Stack Behavior**: Linear scaling for damage, hyperbolic for chances
- **Proc Coefficient**: Respects weapon proc coefficient for balancing

## C# Pseudocode
```csharp
// Constructor logic
public ItemLightningOrb(ItemInventory itemInventoryRef) {
    procChancePerAmount = 0.25f;
    stunChancePerAmount = 0.25f;
    baseRadius = 40.0f;
    damageRatio = 0.4f;
    damageRatioPerAmount = 0.4f;
    damageSource = "LightningOrb";
    yepDc = new DamageContainer(0.0f, "LightningOrb");
}

// Scaling calculation
protected override void OnInitOrAmountChanged() {
    // Hyperbolic scaling for proc chance
    procChance = (amount * procChancePerAmount) /
                 (amount * procChancePerAmount + 0.6f) * 0.9f;

    // Hyperbolic scaling for stun chance
    stunChance = (amount * stunChancePerAmount) /
                 (amount * stunChancePerAmount + 0.6f) * 0.9f;

    // Linear scaling for damage
    damageRatio = amount * damageRatioPerAmount;
}

// Main proc logic
public override void ProcOnHitEffects(DamageContainer dc) {
    // Direct stun on lightning attacks
    if (dc.element == 1 && TryProc(dc.procCoefficient, stunChance)) {
        dc.enemy.AddDebuff(DebuffType.Lightning, dc, 3.0f);
    }

    // Lightning strike proc
    if (TryProc(dc.procCoefficient, procChance)) {
        // Find random enemy in radius
        Enemy target = GetRandomEnemyInRadius(baseRadius);
        if (target != null && !target.IsDead()) {
            // Calculate damage
            float finalDamage = damageRatio * dc.damage;
            float areaMultiplier = PlayerStats.GetStat(EStat.AreaMultiplier) * 8.0f;

            // Lightning strike
            WeaponUtility.LightningStrike(target, yepDc, areaMultiplier);

            // Apply stun chance
            if (TryProc(1.0f, stunChance)) {
                target.AddDebuff(DebuffType.Lightning, yepDc, 3.0f);
            }
        }
    }
}
```

## Technical Notes
- **Performance**: Uses efficient enemy caching to avoid repeated radius searches
- **Memory Management**: Properly handles IL2CPP garbage collection with `j_j_GC_end_stubborn_change`
- **Dead Enemy Optimization**: Swaps dead enemies to end of list before removal to maintain O(1) removal
- **Lightning Element**: Sets damage container element to 1 (Lightning) for consistent theming
- **Area Scaling**: Lightning visual effects scale with EStat 9 multiplied by 8.0 for dramatic effect

## Related Items
- **ElectricPlug**: Chain lightning on taking damage
- **GlovesLightning**: Lightning AOE gloves with similar mechanics
- **All Hyperbolic Items**: IceCrystal, Dragonfire, SluttyCannon (similar scaling pattern)

---

**Data Sources:**
- Constructor: `D:\dev\megabonk\extracted_constructors\items\LightningOrb.c`
- Decompiled C#: `D:\dev\megabonk\decompiled\Assembly-CSharp\Assets.Scripts.Inventory__Items__Pickups.Items.ItemImplementations\ItemLightningOrb.cs`
- Item Database: `D:\dev\megabonk\megabonk_research\items.md`