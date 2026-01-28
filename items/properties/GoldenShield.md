# GoldenShield

## Overview
- **Item ID**: EItem.GoldenShield
- **Constructor Address**: 0x18045B770
- **Category**: Defensive/Economy
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| chancePerAmount | float | 1.0 | Chance multiplier per stack |
| goldPerAmount | int | 6 | Base gold per stack |
| cooldown | float | 0.1 | Cooldown between gold generation (seconds) |
| selfDamageCooldown | float | 0.2 | Cooldown for self-damage triggers (seconds) |
| chance | float | Dynamic | Calculated as amount * chancePerAmount |
| goldOnHit | int | Dynamic | Calculated gold amount on damage taken |

## Stat Modifiers
No direct EStat modifications - this item operates through event-driven mechanics.

## Special Mechanics
- **Gold Generation on Damage**: Generates gold when the player takes damage
- **Character Level Scaling**: Gold amount scales with character level (capped at level 20)
- **Position-based Spawning**: Gold spawns at player's transform position
- **Event-driven**: Subscribes to PlayerHealth.A_TakeDamage event

## Formulas

### Chance Calculation
```
chance = amount * chancePerAmount
chance = amount * 1.0 = amount (100% per stack)
```

### Gold Amount Calculation
```
levelBonus = min(characterLevel * 0.2, 20.0)
goldOnHit = amount * (levelBonus + goldPerAmount)
goldOnHit = amount * (min(level * 0.2, 20) + 6)
```

### Examples:
- **Level 1, 1 stack**: 1 * (0.2 + 6) = 6.2 → 6 gold
- **Level 10, 1 stack**: 1 * (2.0 + 6) = 8 gold
- **Level 50, 1 stack**: 1 * (10.0 + 6) = 16 gold
- **Level 100+, 1 stack**: 1 * (20.0 + 6) = 26 gold (capped)
- **Level 100+, 5 stacks**: 5 * (20.0 + 6) = 130 gold

## Implementation Details
- **Update Frequency**: On damage taken (event-driven)
- **Event Subscriptions**: PlayerHealth.A_TakeDamage
- **Stack Behavior**: Linear scaling - each stack increases both chance and gold amount

## C# Pseudocode
```csharp
public class ItemGoldenShield : ItemBase
{
    public float chancePerAmount = 1.0f;
    public int goldPerAmount = 6;
    public float cooldown = 0.1f;
    public float selfDamageCooldown = 0.2f;
    public float chance;
    public int goldOnHit;

    protected override void OnInitOrAmountChanged()
    {
        // Calculate chance (100% per stack)
        chance = amount * chancePerAmount;

        // Get character level and apply cap
        int characterLevel = MyPlayer.Instance.inventory.GetCharacterLevel();
        int levelBonus = Mathf.Min((int)(characterLevel * 0.2f), 20);

        // Calculate final gold amount
        goldOnHit = amount * (levelBonus + goldPerAmount);
    }

    public override void Init()
    {
        // Subscribe to damage events
        PlayerHealth.A_TakeDamage += OnPlayerTakeDamage;
    }

    public override void Cleanup()
    {
        // Unsubscribe from damage events
        PlayerHealth.A_TakeDamage -= OnPlayerTakeDamage;
    }

    private void OnPlayerTakeDamage(PlayerHealth playerHealth, DamageContainer dc, bool b)
    {
        // Get player position and spawn gold
        Vector3 playerPos = MyPlayer.Instance.transform.position;
        MoneyUtility.SpawnMoney(goldOnHit, playerPos);
    }
}
```

## Technical Notes
- **Performance**: Event-driven design means no continuous processing overhead
- **Gold Spawning**: Uses MoneyUtility.SpawnMoney() for consistent gold drop behavior
- **Level Cap**: Character level bonus is hard-capped at 20 (equivalent to level 100)
- **Stack Synergy**: Each additional stack provides both increased proc chance and gold amount
- **Position Dependency**: Requires valid player transform for gold spawning

## Related Items
- **GoldenSneakers**: Also generates gold, but based on movement distance
- **CreditCardGreen**: Luck-based gold generation from chests
- **CreditCardRed**: Damage-based bonuses from chest interactions

## Assembly Analysis
The native implementation shows:
- Constructor sets `chancePerAmount = 1.0`, `goldPerAmount = 6`, `cooldown = 0.1`, `selfDamageCooldown = 0.2`
- OnInitOrAmountChanged calculates dynamic values using character level
- Event subscription/unsubscription in Init/Cleanup methods
- OnPlayerTakeDamage spawns gold at player position using MoneyUtility

---

**Data Sources:**
- `megabonk_research/items.md` - Item overview and base properties
- `extracted_constructors/items/GoldenShield.c` - Constructor implementation
- `decompiled/Assembly-CSharp/.../ItemGoldenShield.cs` - C# class structure
- IDA Pro disassembly at 0x18045B770 - Constructor implementation