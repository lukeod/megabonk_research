# Megabonk Game Mechanics

> **Last Validated**: 2026-01-28 (IDA verification session - upgrade system, damage formulas)
> **Game Version**: Current (post-IL2CPP)
> **Validation Notes**: All enums, structures, and addresses verified via IDA MCP decompilation

---

## Core Stat System

### Stat Components Structure
The stat system uses a three-component calculation model with the following memory layout:
```cpp
struct StatComponents {
    bool hasModifications;          // offset 0x10
    float _baseValue;               // offset 0x14
    float _additiveValue;           // offset 0x18
    float _multiplicativeValue;     // offset 0x1C
};
```

### Master Stat Calculation Formula
**Location**: `StatComponents::GetFinalValue` at `0x18044CEB0` ✅ Verified 2026-01-28
```
Final Value = (Additive1 + Additive2) × (Base1 + Base2) × (Multiplicative1 × Multiplicative2)
```
Where components 1 and 2 typically represent base stats and item modifiers respectively.

**Modifier Functions** (verified):
- `AddAdditive(value)`: `additiveValue += value` (stacks additively)
- `AddMultiplier(value)`: `multiplicativeValue *= value` (stacks multiplicatively)
- `AddFlat(value)`: Directly adds to base value

### Stat Enum (EStat) - 57 Total Stats
Stats are NOT obfuscated and include combat, defensive, utility, and meta categories. The stat system uses a three-layer storage architecture:
- `Dictionary<EStat, float> stats` - Final calculated values
- `Dictionary<EStat, float> rawStats` - Raw stat values before modifiers
- `Dictionary<EStat, StatComponents> statValuesMap` - Component breakdown for each stat

**Complete EStat Index** (see [enums.md](enums.md) for full list):
| Index | Stat | Index | Stat |
|-------|------|-------|------|
| 0 | MaxHealth | 16 | Projectiles |
| 4 | Armor | 18 | CritChance |
| 5 | Evasion | 19 | CritDamage |
| 12 | DamageMultiplier | 25 | MoveSpeedMultiplier |
| 17 | Lifesteal | 30 | Luck |
| 45 | ProjectileBounces | **46** | **ExtraJumps** |

### Stat Modification Types (EStatModifyType)
```csharp
public enum EStatModifyType {
    Addition = 0,      // Adds to the additive component
    Multiplication = 1, // Multiplies the multiplicative component
    Flat = 2           // Directly sets or overrides values
}
```

### Stat-Specific Constraints
**Location**: `PlayerStatsNew$$GetStat` at `0x180446780` ✅ Verified 2026-01-28
- **MaxHealth (EStat 0)**: Minimum cap of 1.0 (prevents death from stat reduction)
- **Projectiles (EStat 16)**: Forced to integer via `floor()`
- **ProjectileBounces (EStat 45)**: Forced to integer via `floor()`

---

## Upgrade System

### Upgrade Offer Generation
**Location**: `UpgradeData$$GetUpgradeOffer` at `0x180450070` ✅ Verified 2026-01-28

When generating upgrade offers for weapons, the system:

1. **Gets Rarity Multiplier**: From `Rarity$$GetMultiplier(rarity)`
2. **Determines Stat Count**:
   - Base count = rarity value
   - If rarity == 1: 50% chance to stay at 1, 50% chance for 2
   - Increases based on available modifiers in pool
3. **Campfire Bonus** (EItem 41): Each Campfire owned adds +1 to maximum rarity/stat count
4. **Selects Random Modifiers**: Picks `statCount` random stats from available pool
5. **Scales Values**: `finalValue = baseValue × rarityMultiplier`, rounded to 2 decimals

### Rarity Stat Multipliers
**Location**: `Rarity$$GetMultiplier` at `0x18042DFC0` ✅ Verified 2026-01-28

| ERarity | Name | Multiplier |
|---------|------|------------|
| 0 | New | 1.0× |
| 1 | Common | 1.0× |
| 2 | Uncommon | 1.2× |
| 3 | Rare | 1.4× |
| 4 | Epic | 1.6× |
| 5 | Legendary | 2.0× |

### WeaponData Base Stats Initialization
**Location**: `WeaponData$$Init` at `0x18039D830` ✅ Verified 2026-01-28

When a weapon is initialized, these base stats are set:
| EStat ID | Field Source | Description |
|----------|--------------|-------------|
| 9 | 1.0 (constant) | AttackDurationMultiplier |
| 10 | `attackDuration` | AttackDuration |
| 11 | `projectileSpeed` | ProjectileSpeed |
| 12 | `damage` | Damage |
| 15 | 1.0 (constant) | AttackSpeedMultiplier |
| 16 | `projectiles` | Projectiles (cast to float) |
| 18 | `critChance` | CritChance |
| 19 | 0.0 (constant) | CritDamage (starts at 0) |
| 24 | `knockback` | Knockback |
| 45 | `projectileBounces` | ProjectileBounces (cast to float) |

### Stat Categories
```csharp
public enum EStatCategory {
    Offensive = 0,
    Defensive = 1,
    Movement = 2,
    Utility = 3,
    Difficulty = 4,
    Coolness = 5,
    Null = 6
}
```

---

## Combat Mechanics

### Damage Calculation Pipeline

#### 1. Base Damage Formula
**Location**: `DamageUtility$$GetPlayerDamage` at `0x180470230`
```
PostArmorDamage = (1.0 - ArmorStat) × BaseDamage
FinalDamage = PostArmorDamage × (1.0 - DamageReductionMultiplier)
```
- Damage clamped to range [1.0, 2,147,483,600.0]
- Minimum damage of 1.0 ensures all hits do at least 1 damage

#### 2. Armor Mechanics
- **Formula**: `DamageReduction = 1.0 - ArmorStat`
- **No hard cap**, but 100% armor (1.0) provides full immunity

**DcFlags (Damage Container Flags)**:
```csharp
public enum DcFlags {
    None = 0,
    BypassEvade = 1,      // Ignore evasion stat check
    BossDamage = 2,       // Armor effectiveness reduced to 50%
    BypassAegis = 4,      // Ignore Aegis shield interactions
    FinalBossDamage = 8,  // Armor effectiveness reduced to 75%
    IgnoreArmor = 16,     // Bypass armor completely
    BypassAll = 5         // BypassEvade | BypassAegis (composite)
}
```

#### 3. Evasion System
**Location**: `DamageUtility::CheckEvade` at `0x180470040` ✅ Verified 2026-01-28
- **Formula**: `Evaded = (max(0.0, EvasionStat) >= Random.NextDouble())`
- **Stat**: EStat 5 (Evasion)
- **No hard cap** - 100% evasion achieved at stat value 1.0
- Uses `System.Random.NextDouble()` for RNG (0.0 to 1.0)

#### 4. Critical Hit System
**Location**: `DamageUtility$$GetCritDamageMultiplier` at `0x1804700E0`

**Multi-Crit Mechanics** (Verified via IDA decompilation):
1. Calculate base crits: `baseCrits = floor(critChance)`
2. Roll for bonus crit: `if (critChance - baseCrits) > Random(0,1): bonusCrit = 1`
3. `totalCrits = baseCrits + bonusCrit`

**Damage Multipliers**:
  - 0 crits: No critical hit (returns false)
  - 1 crit: 2.0×
  - n crits (n≥2): Special scaling formula
  - Examples: 2 crits = 4.0×, 3 crits = 6.25×, 4 crits = 9.0×

### DamageContainer Structure
```csharp
public class DamageContainer {
    public Vector3 direction;           // Damage/knockback direction
    public float damage;                // Final damage value
    public bool crit;                   // Critical hit flag
    public bool isExecute;              // Execute flag
    public float knockback;             // Knockback force
    public Enemy enemy;                 // Target reference
    public EDamageEffect damageEffect;  // Effect type (Bonk, Megacrit, etc.)
    public EElement element;            // Element type
    public float procCoefficient;       // Status proc multiplier (default 1.0)
    public string damageSource;         // Source identifier
    public int damageBlockedByArmor;    // Armor interaction
    public DcFlags flags;               // Special flags
    public bool canProcJoe;             // Joe's Dagger eligibility
}
```

### Weapon Damage Calculation
**Location**: `WeaponUtility$$GetDamage` at `0x180435A70`
```
BaseDamage = WeaponDamage × PlayerDamageMultiplier
If WeaponType == Aegis (7): Add PlayerStats[Thorns] (EStat 3) before multiplication
```

> **Note**: Aegis uses the Thorns stat (EStat 3) as bonus damage, thematically linking shield defense to offense.

**Damage Container Creation** (`WeaponUtility$$GetDamageContainer` at `0x180435010`):
1. Roll for critical hit: `WeaponCrit + PlayerCrit`
2. Calculate knockback: `WeaponKnockback × PlayerKnockback`
3. Apply final damage:
   - Non-crit: `BaseDamage × ItemModifiers`
   - Critical: `BaseDamage × ItemModifiers × CritMultiplier × (WeaponCritDamage + PlayerCritDamage)`

### Enemy Damage Scaling
**Location**: `RunConfig::GetEnemyDamage` at `0x180451940`

**Map Tier Damage Multipliers**:
| Map Tier | Multiplier | Description |
|----------|------------|-------------|
| 0 | 0.75× | Easiest tier |
| 1 | 0.95× | Easy tier |
| 2 | 1.1× | Hard tier |
| Other | 1.0× | Normal damage |

Formula: `FinalEnemyDamage = BaseDamage × TierMultiplier`

### Elemental Types
```csharp
public enum EElement {
    Neutral,    // No elemental damage
    Lightning,  // Lightning element
    Ice,        // Ice/freeze element
    Fire,       // Fire/burn element
    Bleed       // Bleed status element
}
```

### Damage Effects
```csharp
public enum EDamageEffect {
    None,       // Standard damage
    Bonk,       // Knockback effect
    Megacrit,   // Multi-hit critical
    Poison,     // Poison DoT
    Lightning,  // Lightning chain
    Fire,       // Fire/burn
    Execute,    // Instant kill threshold
    Cursed,     // Curse debuff
    Echo,       // Echo/reflection
    Bloodmark   // Bloodmark debuff
}
```

---

## Health & Shield Systems

### Health Regeneration
**Location**: `PlayerHealth::Tick` at `0x1803E9EB0`
- **Interval-based**: Heals at defined `healInterval`
- **Accumulation**: Fractional healing stored in `healingValue` field
- **Application**: When `healingValue >= 1.0`, applies `floor(healingValue)` healing
- **DoT Support**: Negative values apply damage over time using same system

### PlayerHealth Fields
```csharp
public int hp;                    // Current health
public int maxHp;                 // Maximum health
public float overheal;            // Temporary HP above max
public float maxOverheal;         // Overheal capacity
public float shield;              // Current shield
public float maxShield;           // Shield capacity

public const float damageCooldownTime = 0.15f;  // 150ms invulnerability
```

### Shield Mechanics
- **Regeneration**: `shield += shieldHealingPerTick × deltaTime`
- **Activation**: Begins at `shieldRechargeAtTime` after taking damage
- **Cap**: Limited by `maxShield` stat
- **Range**: Clamped to [0, maxShield]

### Overheal System
- **Decay Formula**: `Overheal -= (maxOverheal × overhealRemovalFraction × deltaTime)`
- **Important**: Decay based on MAX overheal, not current
- **Cap**: Clamped to [0, maxOverheal]

---

## Special Combat Mechanics

### Execute Mechanic
**Location**: `DamageUtility::ApplyExecute` at `0x18046FFC0` ✅ Verified 2026-01-28
- **Flags Set**: `damageEffect = 6` (Execute), `isExecute = true`
- **Non-Boss Behavior**: If enemy has "can be executed" flag → damage = enemy HP (instant kill)
- **Boss Behavior**: damage = 2% of enemy max HP
- **Immunity**: Bosses (`EEnemyFlag.AnyBoss` = 54) cannot be instantly killed
- **Override**: Replaces normal damage calculation when triggered

### Thorns (Retaliation)
**Location**: `PlayerHealth::Retaliate` at `0x1803E9C20`
- **Damage**: 1:1 ratio with Thorns stat value
- **Direction**: Reflects in opposite direction of incoming damage
- **Activation**: Only when Thorns stat > 0

### Lifesteal
- Accumulates healing in `lifestealHeal` field (int)
- Applied per tick with fractional support
- Events: `A_LifestealProc`, `A_LifestealHealing`

---

## Movement & Physics

### Movement Speed Calculation
**Location**: `PlayerMovementValues::GetMoveSpeed` at `0x18044ECA0`
```
SpeedMultiplier = ((StatValue - 1.5) × 0.25) + 1.0  // Clamped [1.0, 4.0]
FinalSpeed = BaseSpeed × SpeedMultiplier × SurfaceModifier
```

### Movement Constants
```csharp
private const float defaultMoveSpeed = 2700f;
private const float defaultSwimSpeed = 10f;
public const float defaultMaxSpeed = 10f;
private const float defaultSlideForce = 200f;
private const float defaultAirDeceleration = 0.003f;
private const float defaultExtraGravity = 11f;
```

### Surface Modifiers (EFrictionSurface)
```csharp
public enum EFrictionSurface {
    Normal,  // 1.0× modifier
    Ice      // 0.4× modifier (slippery surfaces)
}
```

### Movement States (EMovementState)
```csharp
public enum EMovementState {
    Idle = 1,
    Walking = 2,
    Crouching = 4,
    Sliding = 8,
    Airborne = 16,
    Wallrunning = 32,
    CategoryCrouched = 12,      // Crouching | Sliding
    CategoryFootstepNoise = 34  // Walking | Wallrunning
}
```

### Movement State Multipliers
**Location**: `MovementTick` at `0x180347410`
- **Grounded + Sliding**:
  - Horizontal: 0.15×
  - Vertical: 0.25× (against movement) or 0.0×
- **Airborne**: Both axes: 0.45×
- **Underwater**: Gravity: 2.0× (jumping) or 1.0×

### Push/Knockback System
**Location**: `PushPlayer` at `0x180349040`
- Adds push force directly to rigidbody velocity
- Sets `pushed` flag and resets push counter
- Temporarily disables gravity during push
- Triggers camera shake effect

---

## Enemy Debuff System

### Debuff Types (EDebuff) - Bit Flags
```csharp
public enum EDebuff {
    Poison = 1,       // DoT damage
    Freeze = 2,       // Movement slow/stop
    Burn = 4,         // Fire DoT
    Stun = 8,         // Prevents actions
    Echo = 16,        // Damage reflection
    Charm = 32,       // Enemy control reversal
    Bloodmark = 64,   // Damage amplification
    DebuffsWithCap = 42  // Charm | Stun | Freeze (composite for stack caps)
}
```

> **Important**: These are BIT FLAGS, not sequential values!

### Debuff Durations
| Debuff | Duration | Stacking |
|--------|----------|----------|
| Burn (Fire) | 3.0s | Stacks damage |
| Freeze (Ice) | 3.0s | Non-stacking (CC only) |
| Stun (Lightning) | 3.0s | Non-stacking (CC only) |
| Poison | Variable | Full stacking |
| Bloodmark | **4.0s** | Stacks |

### Debuff Stacking System
**Location**: `Enemy$$AddDebuff` at `0x1804AAF70`
```
NewStacks = ExistingStacks + AppliedStacks
NewDuration = max(ExistingDuration, AppliedDuration)
```
- Uses dictionary storage: `Dictionary<EDebuff, EnemyDebuff>`
- Invulnerable enemies ignore debuffs

### AddDebuffContainer Structure
```csharp
public struct AddDebuffContainer {
    public EDebuff eDebuff;     // Debuff type (bit flag)
    public DamageContainer dc;  // Damage reference
    public float duration;      // Duration in seconds
    public int stacks;          // Stack count (default 1)
}
```

### Debuff Damage Formulas

**Fire (DebuffFire::GetDamage)** at `0x18043DB10`:
```
DamagePerTick = (PlayerStat[DamageMultiplier] × CharacterLevel × 0.5) + 1.0
```
- Scales with character level
- Minimum damage 1.0 guaranteed

**Poison (DebuffPoison::GetDamageForHpBar)** at `0x18043DEA0`:
```
DamagePerTick = max(stacks, PlayerStat[DamageMultiplier] × stacks)
TotalDamage = ticksLeft × DamagePerTick
```
- Scales multiplicatively with stack count

---

## Player Status Effects

> **Note**: Player status effects (EStatusEffect) are SEPARATE from enemy debuffs (EDebuff).

### Player Status Types (EStatusEffect)
```csharp
public enum EStatusEffect {
    Haste = 0,
    Rage = 1,
    Shield = 2,
    Stonks = 3,
    TimeFreeze = 4,
    Invulnerability = 5,
    Slow = 6,
    Freeze = 7,
    Bleed = 8,
    Poison = 9,
    BossPoison = 10
}
```

### PlayerStatusEffects System
**Location**: `PlayerStatusEffects::Tick`
- `Dictionary<EStatusEffect, StatusEffect> statusEffects` - Active effects
- Removes expired effects based on `MyTime::time`
- Events: `A_StatusEffectAdded`, `A_StatusEffectRemoved`, `A_StatusModifiedStat`

> **Note**: Stat updates (`queuedUpdateStats`, `A_StatUpdate`) are managed by `PlayerStatsNew`, not `PlayerStatusEffects`.

---

## Item Proc System

### Fire Proc (ItemDragonfire)
**Location**: `ItemDragonfire::TryProcBurn` at `0x180408700`
- **Proc Chance**: Based on `burnChance` field (hyperbolic scaling)
- **Duration**: 3.0 seconds
- **Debuff Type**: EDebuff.Burn (4)

### Ice Proc (ItemIceCube)
**Location**: `ItemIceCube::TryProcFreeze` at `0x180412540`
- **Proc Chance**: Based on `freezeChance` field
- **Duration**: 3.0 seconds
- **Debuff Type**: EDebuff.Freeze (2)
- **Event**: Triggers `A_FreezeEnemy`
- **Effect**: Pure crowd control (no damage)

### Lightning Proc (ItemLightningOrb)
**Location**: `ItemLightningOrb::TryProcStun` at `0x180415880`
- **Duration**: 3.0 seconds
- **Debuff Type**: EDebuff.Stun (8)
- **Effect**: Stun (crowd control)

### Lightning Strike System
**Location**: `WeaponUtility::LightningStrike` at `0x180436360`
- Spawns lightning strike from object pool
- Position: Enemy center + random offset (0.25 unit radius)
- Chain Lightning: Triggers if `bounces > 0`

**Chain Lightning** (`WeaponUtility::ChainLightning` at `0x1804346B0`):
- Chains to nearby enemies within `bounceRange`
- Bounce count decrements with each chain
- Separate proc coefficient for bounced hits

### Proc Coefficient
The `procCoefficient` field in DamageContainer affects all item procs:
- Default: 1.0 (full proc chance)
- Override value: -1.0 (use container's default)
- Applied multiplicatively to proc calculations

---

## Loot & Economy

### Rarity Systems

> **Important**: The game has TWO separate rarity enums!

**ERarity** (Upgrades/Encounters):
```csharp
public enum ERarity {
    New = 0,
    Common = 1,
    Uncommon = 2,
    Rare = 3,
    Epic = 4,
    Legendary = 5
}
```

**EItemRarity** (Item Drops):
```csharp
public enum EItemRarity {
    Common = 0,
    Rare = 1,
    Epic = 2,
    Legendary = 3,
    Corrupted = 4,
    Quest = 5
}
```

### Rarity Stat Multipliers
**Location**: `Rarity$$GetMultiplier` at `0x18042DFC0`

Used for weapon upgrade scaling and other rarity-based calculations:

| ERarity | Value | Multiplier |
|---------|-------|------------|
| New | 0 | 1.0× |
| Common | 1 | 1.0× |
| Uncommon | 2 | 1.2× |
| Rare | 3 | 1.4× |
| Epic | 4 | 1.6× |
| Legendary | 5 | 2.0× |

### Luck System
**Location**: `Rarity::CalculateRarityWeights` at `0x1803F3E70`
```cpp
luckBonus = log(luck + 1.0) × 1.5
weight[i] = baseWeight[i] × pow(1.5, -(maxRarity - i - 1) × luckBonus)
```

### Effects
- Higher luck exponentially increases rare item chances
- Natural logarithm provides diminishing returns
- Each tier is 1.5× less likely than previous (modified by luck)
- Many items use hyperbolic scaling to prevent 100% proc chances

---

## Weapons

### Weapon Types (EWeapon)
```csharp
public enum EWeapon {
    None = -1,
    FireStaff = 0,      Bone = 1,           Sword = 2,
    Revolver = 3,       Aura = 4,           Axe = 5,
    Bow = 6,            Aegis = 7,          Test = 8,
    LightningStaff = 9, Flamewalker = 10,   Rockets = 11,
    Bananarang = 12,    Tornado = 13,       Dexecutioner = 14,
    Sniper = 15,        Frostwalker = 16,   SpaceNoodle = 17,
    DragonsBreath = 18, Chunkers = 19,      Mine = 20,
    PoisonFlask = 21,   BlackHole = 22,     Katana = 23,
    BloodMagic = 24,    BluetoothDagger = 25, Dice = 26,
    HeroSword = 27,     CorruptSword = 28,  Shotgun = 29,
    Scythe = 30
}
```

### Arrow/Bow Projectile Mechanics
**Location**: `ProjectileUtility::GetArrowSpeedReduction` at `0x18044CD00`

**Initial Arrow Speed**:
```
ArrowSpeed = ProjectileSpeed + (AttackSizeMultiplier × 0.25)
```

**Speed Reduction Per Tick** (FixedUpdate at 50 Hz):
```
SpeedReduction = ArrowSpeed × 0.02 / Duration
```

**Per-Frame Deceleration**:
```
newSpeed = max(0.0, min(99.0, currentSpeed - speedReduction))
```

**Targeting Range** (kinematic formula):
```
targetingRange = projectileSpeed² / (2 × speedReduction)
```

- Higher `Duration` = Lower deceleration = Arrows travel farther
- Speed clamped to [0.0, 99.0]
- Arrows can fully stop (reach 0 speed)

---

## Summary of Core Formulas

1. **Stat Calculation**: `(Add1+Add2) × (Base1+Base2) × (Mult1×Mult2)`
2. **Damage**: `(1-Armor) × BaseDamage × (1-DamageReduction)`, min 1.0
3. **Movement Speed**: `BaseSpeed × ((Stat-1.5)×0.25+1.0) × SurfaceModifier`
4. **Crit Damage (n≥2)**: `(n×0.5)² + (n+1)`
5. **Luck Rarity**: `weight = base × pow(1.5, -log(luck+1)×1.5×(maxRarity-i-1))`
6. **Shield Regen**: `shield += regenRate × deltaTime`
7. **Overheal Decay**: `overheal -= maxOverheal × decayRate × deltaTime`
8. **Debuff Stacking**: `stacks = old + new, duration = max(old, new)`
9. **Fire DoT**: `(DamageMultiplier × Level × 0.5) + 1.0`
10. **Poison DoT**: `max(stacks, DamageMultiplier × stacks)`
11. **Arrow Deceleration**: `reduction = (speed + size×0.25) × 0.02 / duration`
12. **Crit Damage (n=1)**: `2.0×` (hardcoded special case)

---

## Key Implementation Notes

### IL2CPP Limitations
All method implementations in decompiled C# are stubs returning default values. Actual game logic is in native IL2CPP code at documented addresses. Formula verification requires IDA disassembly.

### Performance Optimizations
- Object pooling for projectiles, damage containers, and effects
- Dictionary-based status effect tracking
- Fractional accumulation for healing/damage over time
- Interval-based updates for regeneration
- Cached stat calculations with queued updates

### Special Flags Reference

**EEnemyFlag** (Bit Flags):
| Flag | Value | Effect |
|------|-------|--------|
| None | 0 | No special flags |
| Elite | 1 | Elite enemy |
| Boss | 2 | Boss enemy |
| StageBoss | 4 | Stage boss |
| Challenge | 8 | Challenge enemy |
| SummonerMiniboss | 16 | Summoner miniboss |
| FinalBoss | 32 | Final boss |
| AnyBoss | 54 | Composite: Boss\|StageBoss\|SummonerMiniboss\|FinalBoss (execute immunity) |

**DcFlags** (Damage Container):
| Flag | Value | Effect |
|------|-------|--------|
| BossDamage | 2 | 50% armor effectiveness |
| FinalBossDamage | 8 | 75% armor effectiveness |
| IgnoreArmor | 16 | Complete armor bypass |

**Other**:
| Flag | Value | Effect |
|------|-------|--------|
| Aegis Weapon Type | 7 | Adds bonus damage before multiplication |
