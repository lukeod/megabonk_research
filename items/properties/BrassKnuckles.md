# BrassKnuckles

## Overview
- **Item ID**: EItem.BrassKnuckles
- **Constructor Address**: 0x18043C8F0
- **Category**: Melee Damage Enhancement
- **Rarity**: [Not determinable from available code]

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| damagePerAmount | float | 0.25 | Flat damage increase per stack |
| baseRadius | float | 8.0 | Base maximum distance for melee bonus to apply |
| radiusAddPerAmount | float | 2.0 | Additional radius per stack |
| radius | float | Calculated | baseRadius + (amount * radiusAddPerAmount) |
| flatValue | float | Calculated | amount * damagePerAmount |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None Direct | N/A | Conditional bonus applied via StatComponents.AddAdditive() | Conditional |

## Special Mechanics
- **Proximity-Based Damage**: Only applies damage bonus when enemy is within calculated radius of the player
- **Melee Range Check**: Uses enemy movement component to calculate distance to player
- **Conditional Application**: Damage bonus is applied by adding flatValue to StatComponents._baseValue_k__BackingField only when proximity condition is met
- **PreAttack Hook**: Implements the PreAttack virtual method to modify damage before it's applied

## Formulas
- **Radius Calculation**: `radius = baseRadius + (amount * radiusAddPerAmount) = 8.0 + (stack_count * 2.0)`
- **Damage Bonus**: `flatValue = damagePerAmount * amount = 0.25 * stack_count`
- **Range Check**: `enemy_distance <= radius`
- **Effective Damage Increase**: +0.25 flat damage per stack when within range, 0 otherwise

## Implementation Details
- **Update Frequency**: Evaluated on every attack (PreAttack hook)
- **Event Subscriptions**: PreAttack event from ItemBase
- **Stack Behavior**: Linear scaling - each stack adds +0.25 flat damage and +2.0 radius
- **Distance Calculation**: Uses enemy movement component's position data
- **Null Safety**: Includes comprehensive null checks for DamageContainer, Enemy, and EnemyMovement components

## C# Pseudocode
```csharp
public class ItemBrassKnuckles : ItemBase
{
    private float damagePerAmount = 0.25f;      // Flat damage per stack
    private float baseRadius = 8.0f;            // Base melee range
    private float radiusAddPerAmount = 2.0f;    // Additional radius per stack
    private float radius;                        // Calculated total radius
    private float flatValue;                     // Calculated damage bonus

    public override void PreAttack(DamageContainer dc, StatComponents itemAttackModifier)
    {
        // Null safety checks
        if (dc?.enemy?.enemyMovement == null)
            return;

        // Calculate distance to enemy
        float distanceToEnemy = GetDistanceToPlayer(dc.enemy.enemyMovement);

        // Apply damage bonus only if within melee range
        if (distanceToEnemy <= radius && itemAttackModifier != null)
        {
            itemAttackModifier._baseValue_k__BackingField += flatValue;
            itemAttackModifier.hasModifications = true;
        }
    }

    protected override void OnInitOrAmountChanged()
    {
        // Calculate radius based on current stack count
        radius = baseRadius + (amount * radiusAddPerAmount); // 8.0 + (stacks * 2.0)
        // Calculate flat damage bonus
        flatValue = amount * damagePerAmount; // stacks * 0.25
    }
}
```

## Technical Notes
- **IL2CPP Interop**: Uses IL2CPP runtime for Unity integration
- **Native Method Calls**: Direct calls to native IL2CPP methods for performance
- **Memory Management**: Uses unsafe pointers for direct field access
- **Performance**: Extremely lightweight - only distance calculation on each attack
- **Range Limitation**: Unlike many items, this has a hard range limitation making it specialized for melee builds

## Related Items
- **TacticalGlasses**: Similar conditional damage bonus (health-based instead of range-based)
- **Scarf**: Ground-state conditional damage bonus
- **GamerGoggles**: Health-percentage based damage bonus
- **Bonker**: Area damage item that also uses radius mechanics

## Synergies
- **Movement Speed Items**: Helps close distance to enemies for melee range
- **TurboSocks**: +15% movement speed per stack
- **PhantomShroud**: Evasion + speed boost on evade
- **CowardsCloak**: Speed boost when taking damage

## Build Considerations
- **Melee Focused**: Best suited for close-combat builds
- **High Stack Value**: Linear scaling makes multiple stacks very effective
- **Range Dependency**: Requires positioning management to be effective
- **No Cooldown**: Can apply on every single attack unlike proc-based items

---

*Data extracted from IL2CPP decompiled constructor at 0x18043C8F0 and C# interop wrapper analysis*