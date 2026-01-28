# BloodyCleaver

## Overview
- **Item ID**: EItem.BloodyCleaver
- **Constructor Address**: 0x18043A790
- **Category**: Status Effect / Bleeding
- **Rarity**: Unknown

## Base Properties
| Property | Type | Value | Notes |
|----------|------|-------|-------|
| bloodmarkStacksPerLifestealPerAmount | int | 1 | Stacks gained per lifesteal per item |
| bloodmarkChancePerAmount | float | 0.5 | 50% base chance per stack |
| bloodmarkStacksPerLifesteal | int | amount * bloodmarkStacksPerLifestealPerAmount | Calculated total stacks per lifesteal |
| totalBloodmarkChance | float | amount * bloodmarkChancePerAmount | Calculated total proc chance |
| damageSource | string | Static field | Damage source identifier |
| dc | DamageContainer | Initialized | Created with 0.0 damage and item's damage source |
| lifestealProcTracker | Dictionary<Enemy, int> | Initialized | Tracks lifesteal procs per enemy |

## Stat Modifiers
| EStat ID | Stat Name | Value/Formula | Scaling Type |
|----------|-----------|---------------|--------------|
| None | No direct stat modifications | N/A | N/A |

## Special Mechanics
- **Bleeding Application (On Hit)**: Applies bleeding debuff (ID 64) for 5.0 seconds on successful proc
- **Bleeding Application (Lifesteal)**: Applies bleeding debuff (ID 64) for 4.0 seconds via lifesteal tracking
- **Lifesteal Integration**: Subscribes to lifesteal proc events via PlayerHealth.A_LifestealProc
- **Lifesteal Tracking**: Uses Dictionary<Enemy, int> to track lifesteal procs per enemy, applied during Tick
- **Proc-Based**: Triggers on hit using ProcOnHitEffects with proc coefficient scaling
- **Stacking Logic**: Uses floor calculation for guaranteed stacks plus probability for remainder

## Formulas
- **bloodmarkStacksPerLifesteal**: `amount * bloodmarkStacksPerLifestealPerAmount` (always = amount since bloodmarkStacksPerLifestealPerAmount = 1)
- **totalBloodmarkChance**: `amount * bloodmarkChancePerAmount` (50% per stack)
- **Guaranteed Stacks**: `floor(totalBloodmarkChance)`
- **Remainder Chance**: `totalBloodmarkChance - floor(totalBloodmarkChance)`
- **Final Stacks Applied**: `guaranteedStacks + (remainder chance proc ? 1 : 0)`

## Implementation Details
- **Update Frequency**: On-demand (ProcOnHitEffects) + Tick for lifesteal procs
- **Event Subscriptions**:
  - PlayerHealth.A_LifestealProc (Init/Cleanup managed)
- **Stack Behavior**: Each successful proc applies bleeding debuff to target enemy
- **Proc Mechanism**: Uses ItemUtility.TryProc with damage container's proc coefficient
- **Lifesteal Debuff Stacks**: `bloodmarkStacksPerLifesteal * lifestealProcCount` per enemy

## C# Pseudocode
```csharp
// Constructor logic
public ItemBloodyCleaver(ItemInventory itemInventoryRef)
{
    this.bloodmarkStacksPerLifestealPerAmount = 1;
    this.bloodmarkChancePerAmount = 0.5f;
    this.dc = new DamageContainer(0.0f, damageSource, null);
    this.lifestealProcTracker = new Dictionary<Enemy, int>();
    base(itemInventoryRef);
}

// Amount change handler
protected override void OnInitOrAmountChanged()
{
    this.bloodmarkStacksPerLifesteal = this.amount * this.bloodmarkStacksPerLifestealPerAmount;
    this.totalBloodmarkChance = (float)this.amount * this.bloodmarkChancePerAmount;
}

// Hit processing
public override void ProcOnHitEffects(DamageContainer dc)
{
    if (dc == null) return;

    if (ItemUtility.TryProc(dc.procCoefficient, this.totalBloodmarkChance))
    {
        int stacksToApply = 0;

        if (this.bloodmarkChancePerAmount > 0.0f)
        {
            stacksToApply = (int)Math.Floor(this.totalBloodmarkChance);
        }

        if (ItemUtility.TryProc(dc.procCoefficient, this.totalBloodmarkChance - stacksToApply))
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

// Tick - applies bloodmark debuffs from lifesteal procs
public override void Tick()
{
    foreach (var kvp in lifestealProcTracker)
    {
        if (kvp.Key != null)
        {
            kvp.Key.AddDebuff(64, dc, 4.0f, bloodmarkStacksPerLifesteal * kvp.Value);
        }
    }
    lifestealProcTracker.Clear();
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

*Generated from decompiled IL2CPP constructor at 0x18043A790 and C# interop code*