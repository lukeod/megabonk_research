# BloodyCleaver

## Overview
- **Item ID**: EItem.BloodyCleaver
- **Constructor Address**: 0x180401730
- **Category**: Status Effect / Bleeding
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| stacksPerAmount | int | 1 | Stacks gained per item |
| chanceToStackPerAmount | float | 0.5 | 50% base chance per stack |
| stacks | int | amount * stacksPerAmount | Calculated total stacks |
| totalChance | float | amount * chanceToStackPerAmount | Calculated total proc chance |
| damageSource | string | Static field | Damage source identifier |
| dc | DamageContainer | Initialized | Created with 0.0 damage and item's damage source |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | No direct stat modifications | N/A | N/A |

## Special Mechanics
- **Bleeding Application**: Applies bleeding debuff (ID 64) for 5.0 seconds on successful proc
- **Lifesteal Integration**: Subscribes to lifesteal proc events via PlayerHealth.A_LifestealProc
- **Proc-Based**: Triggers on hit using ProcOnHitEffects with proc coefficient scaling
- **Stacking Logic**: Uses floor calculation for guaranteed stacks plus probability for remainder

## Formulas
- **Stacks**: `amount * stacksPerAmount` (always = amount since stacksPerAmount = 1)
- **Total Chance**: `amount * chanceToStackPerAmount` (50% per stack)
- **Guaranteed Stacks**: `floor(totalChance)`
- **Remainder Chance**: `totalChance - floor(totalChance)`
- **Final Stacks Applied**: `guaranteedStacks + (remainder chance proc ? 1 : 0)`

## Implementation Details
- **Update Frequency**: On-demand (ProcOnHitEffects)
- **Event Subscriptions**:
  - PlayerHealth.A_LifestealProc (Init/Cleanup managed)
- **Stack Behavior**: Each successful proc applies bleeding debuff to target enemy
- **Proc Mechanism**: Uses ItemUtility.TryProc with damage container's proc coefficient

## C# Pseudocode
```csharp
// Constructor logic
public ItemBloodyCleaver(ItemInventory itemInventoryRef)
{
    this.stacksPerAmount = 1;
    this.chanceToStackPerAmount = 0.5f;
    this.dc = new DamageContainer(0.0f, damageSource, null);
    base(itemInventoryRef);
}

// Amount change handler
protected override void OnInitOrAmountChanged()
{
    this.stacks = this.amount * this.stacksPerAmount;
    this.totalChance = (float)this.amount * this.chanceToStackPerAmount;
}

// Hit processing
public override void ProcOnHitEffects(DamageContainer dc)
{
    if (dc == null) return;

    if (ItemUtility.TryProc(dc.procCoefficient, this.totalChance))
    {
        int stacksToApply = 0;

        if (this.chanceToStackPerAmount > 0.0f)
        {
            stacksToApply = (int)Math.Floor(this.totalChance);
        }

        if (ItemUtility.TryProc(dc.procCoefficient, this.totalChance - stacksToApply))
        {
            stacksToApply++;
        }

        for (int i = 0; i < stacksToApply; i++)
        {
            if (dc.enemy != null)
            {
                dc.enemy.AddDebuff(64, dc, 5.0f, 1); // Bleeding for 5 seconds
            }
        }
    }
}

// Event management
public override void Init()
{
    PlayerHealth.A_LifestealProc += OnLifestealProc;
}

public override void Cleanup()
{
    PlayerHealth.A_LifestealProc -= OnLifestealProc;
}
```

## Technical Notes
- **Memory Management**: Uses IL2CPP garbage collection barriers for DamageContainer reference
- **Event Safety**: Properly subscribes/unsubscribes from lifesteal events in Init/Cleanup
- **Null Safety**: Checks for null DamageContainer and enemy before applying effects
- **Floating Point Logic**: Uses Math.Floor for deterministic stack calculation with probabilistic remainder

## Related Items
- **UnstableTransfusion**: Also applies bleeding debuff (27% chance per stack, 5.0s duration)
- **GlovesBlood**: Creates bleeding explosion effect (5s duration)
- **Items with similar proc mechanics**: Bonker, Dragonfire, EagleClaw (all use chance-based proc systems)

---

*Generated from decompiled IL2CPP constructor at 0x180401730 and C# interop code*