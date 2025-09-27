
## Core Stat System

### Stat Components Structure
The stat system uses a three-component calculation model with the following memory layout:
```cpp
struct StatComponents {
    float _baseValue;           // offset 0x14
    float _additiveValue;       // offset 0x18
    float _multiplicativeValue;  // offset 0x1C
};
```

### Master Stat Calculation Formula
**Location**: `StatComponents::GetFinalValue` at `0x1803f5a70`
```
Final Value = (Additive1 + Additive2) × (Base1 + Base2) × (Multiplicative1 × Multiplicative2)
```
Where components 1 and 2 typically represent base stats and item modifiers respectively.

### Stat Enum (EStat) - 60 Total Stats
Stats are NOT obfuscated and include combat, defensive, utility, and meta categories. The stat system uses a three-layer storage architecture:
- `Dictionary<EStat, float> stats` - Final calculated values
- `Dictionary<EStat, float> rawStats` - Raw stat values before modifiers
- `Dictionary<EStat, StatComponents> statValuesMap` - Component breakdown for each stat

Key stats include:
- Combat: MaxHealth, HealthRegen, Shield, Armor, Evasion, DamageMultiplier, CritChance, CritDamage
- Special: Overheal, Luck, Difficulty, Execute, Thorns, Lifesteal
- Movement: MoveSpeedMultiplier, JumpHeight, ExtraJumps, FallDamageReduction
- Economy: GoldIncreaseMultiplier, XpIncreaseMultiplier, Projectiles

### Stat Modification Types (EStatModifyType)
1. **Addition** - Adds to the additive component
2. **Multiplication** - Multiplies the multiplicative component
3. **Flat** - Directly sets or overrides values

### Stat-Specific Constraints
**Location**: `PlayerStatsNew::GetStat` at `0x1803eb5a0`
- **MaxHealth**: Minimum cap of 1.0 (prevents death from stat reduction)
- **Projectiles** (EStat 16): Forced to integer via `floor()`
- **ExtraJumps** (EStat 45): Forced to integer via `floor()`

---

## Combat Mechanics

### Damage Calculation Pipeline

#### 1. Base Damage Formula
**Location**: `DamageUtility::GetPlayerDamage` at `0x18043CAE0`
```
PostArmorDamage = (1.0 - ArmorStat) × BaseDamage
FinalDamage = PostArmorDamage × (1.0 - DamageReductionMultiplier)
```
- Damage clamped to range [1.0, 2,147,483,600.0]
- Minimum damage of 1.0 ensures all hits do at least 1 damage

#### 2. Armor Mechanics
- **Formula**: `DamageReduction = 1.0 - ArmorStat`
- **No hard cap**, but 100% armor (1.0) provides full immunity
- **Special Flags**:
  - Flag 8: Armor effectiveness reduced to 75%
  - Flag 2: Armor effectiveness reduced to 50%

#### 3. Evasion System
**Location**: `DamageUtility::CheckEvade` at `0x18043C8F0`
- **Formula**: `Evaded = (EvasionStat >= Random(0,1))`
- **No hard cap** - 100% evasion achieved at stat value 1.0
- Uses `System.Random.NextDouble()` for RNG

#### 4. Critical Hit System
**Location**: `DamageUtility::GetCritDamageMultiplier` at `0x18043C990`
- **Multi-Crit Support**: CritChance > 100% enables multiple critical hits
- **Damage Multipliers**:
  - 0 crits: 1.0× (normal damage)
  - 1 crit: 2.0×
  - n crits (n≥2): `(n×0.5)² + (n+1)`
  - Examples: 2 crits = 4.0×, 3 crits = 6.25×, 4 crits = 9.0×

### Weapon Damage Calculation
**Location**: `WeaponUtility::GetDamage` at `0x1803FE380`
```
BaseDamage = WeaponDamage × PlayerDamageMultiplier
If WeaponType == 7: Add PlayerBonusDamage before multiplication
```

**Damage Container Creation** (`WeaponUtility::GetDamageContainer` at `0x1803FDB70`):
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

### Projectile System
The projectile system implements several key features:
- **Object Pooling**: Reuses projectile instances for performance
- **Direction Vectors**: Stored in damage container for trajectory calculation
- **Element Type Transfer**: Weapon's element type is transferred to projectiles
- **Proc Coefficient**: Affects status effect trigger chances on projectile hits

---

## Health & Shield Systems

### Health Regeneration
**Location**: `PlayerHealth::Tick` at `0x1803E9EB0`
- **Interval-based**: Heals at defined `healInterval`
- **Accumulation**: Fractional healing stored in `healingValue` field
- **Application**: When `healingValue >= 1.0`, applies `floor(healingValue)` healing
- **DoT Support**: Negative values apply damage over time using same system
- **Field Structure**: Uses `hp`, `maxHp`, `overheal`, `maxOverheal`, `shield`, `maxShield` fields

### Shield Mechanics
- **Regeneration**: `shield += shieldHealingPerTick × deltaTime`
- **Activation**: Begins at `shieldRechargeAtTime`
- **Cap**: Limited by `maxShield` stat
- **Range**: Clamped to [0, maxShield]

### Overheal System
- **Decay Formula**: `Overheal -= (maxOverheal × overhealRemovalFraction × deltaTime)`
- **Important**: Decay based on MAX overheal, not current
- **Cap**: Clamped to [0, maxOverheal]

---

## Special Combat Mechanics

### Execute Mechanic
**Location**: `DamageUtility::ApplyExecute` at `0x18043C840`
- **Effect**: Sets damage to 2% of enemy max HP
- **Immunity**: Bosses (flag 54) are immune
- **Override**: Replaces normal damage calculation when triggered

### Thorns (Retaliation)
**Location**: `PlayerHealth::Retaliate` at `0x1803E9C20`
- **Damage**: 1:1 ratio with Thorns stat value
- **Direction**: Reflects in opposite direction of incoming damage
- **Activation**: Only when Thorns stat > 0

### Lifesteal
- Accumulates healing in `lifestealHeal` field
- Applied per tick with fractional support

---

## Movement & Physics

### Movement Speed Calculation
**Location**: `PlayerMovementValues::GetMoveSpeed` at `0x18044ECA0`
```
SpeedMultiplier = ((StatValue - 1.5) × 0.25) + 1.0  // Clamped [1.0, 4.0]
FinalSpeed = BaseSpeed × SpeedMultiplier × SurfaceModifier
```

### Surface Modifiers
- Normal surface: 1.0×
- Surface type 1 (water/mud): 0.4×

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

## Elemental Damage & Status Effects

### Elemental Types

#### Fire (Burn)
**Location**: `ItemDragonfire::TryProcBurn` at `0x180408700`
- **Proc Chance**: Based on item's `burnChance` field
- **Duration**: 3.0 seconds (hardcoded)
- **Debuff Type**: 4
- **Stack Count**: 1 per application
- **Proc Coefficient**: Uses damage container's proc coefficient unless overridden
- **Damage Formula** (`DebuffFire::GetDamage` at `0x18043DB10`):
  - Damage per tick = `(PlayerStat[12] × CharacterLevel × 0.5) + 1.0`
  - Scales with character level
  - Minimum damage of 1.0 guaranteed

#### Ice (Freeze)
**Location**: `ItemIceCube::TryProcFreeze` at `0x180412540`
- **Proc Chance**: Based on item's `freezeChance` field
- **Duration**: 3.0 seconds (hardcoded)
- **Debuff Type**: 2
- **Stack Count**: 1 per application
- **Special**: Triggers `A_FreezeEnemy` event when successful
- **Enemy Effect**: Calls `Enemy::AddDebuff` with freeze parameters
- **Boss Immunity**: Bosses may have immunity (flag-based)
- **Note**: Pure crowd control effect, no DoT damage component found

#### Lightning
**Location**: `WeaponUtility::LightningStrike` at `0x1803FF6F0`
- **Visual Effect**: Spawns lightning strike from object pool
- **Position**: Enemy center + random offset (0.25 unit radius)
- **Chain Lightning**: Triggers if bounces > 0
- **Damage Application**: Direct damage via `Enemy::DamageFromPlayerWeapon`

**Chain Lightning** (`WeaponUtility::ChainLightning` at `0x1803FCC90`):
- **Bounce System**: Chains to nearby enemies within `bounceRange`
- **Bounce Count**: Decrements with each chain
- **Proc Coefficient**: Separate coefficient for bounced hits
- **Target Selection**: Finds enemies within range for chaining

**Lightning Stun** (`ItemLightningOrb::TryProcStun` at `0x180415880`):
- **Duration**: 3.0 seconds (hardcoded)
- **Debuff Type**: 8
- **Stack Count**: 1 per application
- **Effect**: Stun (crowd control)

#### Poison
**Location**: `DebuffPoison::MyTick` at `0x18043DF10`
- **Damage Formula** (`DebuffPoison::GetDamageForHpBar` at `0x18043DEA0`):
  - Damage per tick = `max(stacks, PlayerStat[12] × stacks)`
  - Total damage = `ticksLeft × DamagePerTick`
- **Debuff Type**: 3
- **Stacking**: Damage scales with stack count
- **Damage Source**: Static "poisonDamageSource" field
- **Note**: PlayerStat[12] appears to be a damage multiplier stat

### Debuff Stacking System
**Location**: `Enemy::AddDebuff` at `0x180441C70`
```
NewStacks = ExistingStacks + AppliedStacks
NewDuration = max(ExistingDuration, AppliedDuration)
```
- Uses dictionary storage: `Dictionary<EDebuff, AddDebuffContainer>`
- Invulnerable enemies ignore debuffs

#### AddDebuffContainer Structure
Contains the following fields for tracking debuff state:
- `eDebuff`: Debuff type ID
- `duration`: Time remaining on the debuff
- `dc`: DamageContainer reference for damage calculations
- `stacks`: Current stack count of the debuff

---

## Loot & Economy

### Luck System
**Location**: `Rarity::CalculateRarityWeights` at `0x1803F3E70`
```cpp
luckBonus = log(luck + 1.0) × 1.5
weight[i] = baseWeight[i] × pow(1.5, -(maxRarity - i - 1) × luckBonus)
```

### Base Rarity Weights
- Common: 70.0
- Uncommon: 15.0
- Rare: 6.0
- Legendary: 1.5

### Effects
- Higher luck exponentially increases rare item chances
- Natural logarithm provides diminishing returns
- Each tier is 1.5× less likely than previous (modified by luck)

---

## Stat Behaviors

### Combat Stats
| Stat | Mechanics | Caps/Constraints |
|------|-----------|------------------|
| MaxHealth | Base HP pool | Minimum 1.0 |
| HealthRegen | Interval-based healing | No cap |
| Shield | Regenerates after delay | Capped at maxShield |
| Armor | Damage reduction: `(1-Armor)×Damage` | No cap, 100% = immunity |
| Evasion | Chance to avoid damage | No cap, 100% at 1.0 |
| DamageMultiplier | Multiplicative damage boost | No cap |
| CritChance | Critical hit probability | No cap, supports multi-crit |
| CritDamage | Critical damage multiplier | Uses multi-crit formula |
| Lifesteal | HP recovery on damage | Accumulates fractionally |
| Thorns | Reflects damage 1:1 | No cap |

### Special Stats
| Stat | Mechanics | Notes |
|------|-----------|-------|
| Overheal | Temporary HP above max | Decays based on MAX overheal |
| Luck | Affects drop rarity | Logarithmic scaling |
| Execute | Instant kill at threshold | 2% enemy max HP, bosses immune |
| Projectiles | Number of projectiles | Integer only |
| ExtraJumps | Additional jump count | Integer only |

---

## Key Implementation Details

### Memory Management
- Object pooling for projectiles and effects
- Dictionary-based status effect tracking
- Component-based stat storage with backing fields

### Status Effect Management
**PlayerStatusEffects::Tick**:
- Iterates through active status effects dictionary
- Removes expired effects based on `MyTime::time`
- Calls `TickEffects` to apply ongoing damage/effects
- Stats are queued for updates via `HashSet<EStat> queuedUpdateStats`
- Updates process during `Tick()` method
- Event `A_StatUpdate` fires when stats change

### Performance Optimizations
- Fractional accumulation for healing/damage over time
- Interval-based updates for regeneration
- Cached stat calculations with queued updates

### Special Cases & Flags
- Flag 54: Boss immunity to execute
- Flag 8: 75% armor effectiveness
- Flag 2: 50% armor effectiveness
- Weapon Type 7: Adds bonus damage before multiplication

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
