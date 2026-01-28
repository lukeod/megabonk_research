# Megabonk Enum Reference

> **Last Validated**: 2026-01-28 against decompiled Assembly-CSharp
> **Source**: `D:\dev\megabonk\decompiled\Assembly-CSharp\`

This document contains complete enum definitions extracted from the game's decompiled code.

---

## Stats & Modifiers

### EStat - Player Statistics (57 Values)
**Location**: `Assets.Scripts.Menu.Shop\EStat.cs`

| Index | Name | Category | Notes |
|-------|------|----------|-------|
| 0 | MaxHealth | Defensive | Min cap 1.0 |
| 1 | HealthRegen | Defensive | |
| 2 | Shield | Defensive | |
| 3 | Thorns | Defensive | |
| 4 | Armor | Defensive | |
| 5 | Evasion | Defensive | |
| 6 | Evolve | Utility | |
| 7 | DamageReductionMultiplier | Defensive | |
| 8 | DamageCooldownMultiplier | Utility | |
| 9 | SizeMultiplier | Utility | |
| 10 | DurationMultiplier | Utility | |
| 11 | ProjectileSpeedMultiplier | Offensive | |
| 12 | DamageMultiplier | Offensive | Used in DoT formulas |
| 13 | Unused0 | - | Reserved |
| 14 | EffectDurationMultiplier | Utility | |
| 15 | AttackSpeed | Offensive | |
| 16 | Projectiles | Offensive | Forced to integer |
| 17 | Lifesteal | Offensive | |
| 18 | CritChance | Offensive | Supports >100% |
| 19 | CritDamage | Offensive | |
| 20 | FireDamage | Offensive | |
| 21 | IceDamage | Offensive | |
| 22 | LightningDamage | Offensive | |
| 23 | EliteDamageMultiplier | Offensive | |
| 24 | KnockbackMultiplier | Offensive | |
| 25 | MoveSpeedMultiplier | Movement | |
| 26 | JumpHeight | Movement | |
| 27 | FallDamageReduction | Defensive | |
| 28 | Slam | Movement | |
| 29 | PickupRange | Utility | |
| 30 | Luck | Economy | Logarithmic scaling |
| 31 | GoldIncreaseMultiplier | Economy | |
| 32 | XpIncreaseMultiplier | Economy | |
| 33 | ChestIncreaseMultiplier | Economy | |
| 34 | ChestPriceMultiplier | Economy | |
| 35 | ShopPriceReduction | Economy | |
| 36 | Holiness | Special | |
| 37 | Wickedness | Special | |
| 38 | Difficulty | Difficulty | |
| 39 | EliteSpawnIncrease | Difficulty | |
| 40 | PowerupBoostMultiplier | Utility | |
| 41 | PowerupChance | Utility | |
| 42 | BurnChance | Offensive | |
| 43 | FreezeChance | Offensive | |
| 44 | WeaponBurstCooldown | Offensive | |
| 45 | ProjectileBounces | Offensive | |
| 46 | ExtraJumps | Movement | Forced to integer |
| 47 | Overheal | Defensive | |
| 48 | HealingMultiplier | Defensive | |
| 49 | SilverIncreaseMultiplier | Economy | |
| 50 | EnemyAmountMultiplier | Difficulty | |
| 51 | EnemySizeMultiplier | Difficulty | |
| 52 | EnemySpeedMultiplier | Difficulty | |
| 53 | EnemyHpMultiplier | Difficulty | |
| 54 | EnemyDamageMultiplier | Difficulty | |
| 55 | EnemyScalingMultiplier | Difficulty | |
| 56 | PoisonDamageMultiplier | Offensive | |

### EStatModifyType
**Location**: `Assets.Scripts.Inventory__Items__Pickups.Stats\EStatModifyType.cs`

```csharp
public enum EStatModifyType {
    Addition = 0,       // Adds to additive component
    Multiplication = 1, // Multiplies multiplicative component
    Flat = 2            // Direct set/override
}
```

### EStatCategory
**Location**: `Assets.Scripts.Inventory__Items__Pickups.Stats\EStatCategory.cs`

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

## Combat

### EDebuff - Enemy Debuffs (Bit Flags)
**Location**: `Assets.Scripts.Game.Combat.EnemyDebuffs\EDebuff.cs`

> **Important**: These are BIT FLAGS for combining multiple debuffs!

```csharp
public enum EDebuff {
    Poison = 1,         // 0x01 - DoT damage
    Freeze = 2,         // 0x02 - Movement stop (3.0s)
    Burn = 4,           // 0x04 - Fire DoT (3.0s)
    Stun = 8,           // 0x08 - Prevents actions (3.0s)
    Echo = 16,          // 0x10 - Damage reflection
    Charm = 32,         // 0x20 - Enemy control reversal
    Bloodmark = 64,     // 0x40 - Damage amplification (4.0s)
    DebuffsWithCap = 42 // 0x2A - Composite: Charm|Stun|Freeze
}
```

### EStatusEffect - Player Status Effects
**Location**: `Assets.Scripts.Inventory__Items__Pickups.Stats\EStatusEffect.cs`

> **Note**: These are PLAYER-side effects, separate from enemy EDebuff.

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

### EDamageEffect
**Location**: `Assets.Scripts.Game.Combat\EDamageEffect.cs`

```csharp
public enum EDamageEffect {
    None,       // Standard damage
    Bonk,       // Knockback effect
    Megacrit,   // Multi-hit critical
    Poison,     // Poison DoT visual
    Lightning,  // Lightning chain visual
    Fire,       // Fire/burn visual
    Execute,    // Instant kill threshold
    Cursed,     // Curse debuff visual
    Echo,       // Echo/reflection visual
    Bloodmark   // Bloodmark debuff visual
}
```

### EElement
**Location**: `Assets.Scripts.Game.Combat\EElement.cs`

```csharp
public enum EElement {
    Neutral,    // No elemental damage
    Lightning,  // Lightning element
    Ice,        // Ice/freeze element
    Fire,       // Fire/burn element
    Bleed       // Bleed status element
}
```

### DcFlags - Damage Container Flags
**Location**: `Assets.Scripts.Actors\DcFlags.cs`

```csharp
public enum DcFlags {
    None = 0,
    BypassEvade = 1,      // Ignore evasion check
    BossDamage = 2,       // 50% armor effectiveness
    BypassAegis = 4,      // Ignore Aegis interactions
    FinalBossDamage = 8,  // 75% armor effectiveness
    IgnoreArmor = 16,     // Complete armor bypass
    BypassAll = 5         // BypassEvade | BypassAegis
}
```

---

## Movement

### EMovementState
**Location**: `Assets.Scripts.Movement\EMovementState.cs`

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

### EFrictionSurface
**Location**: `FrictionModifier.cs` (nested enum)

```csharp
public enum EFrictionSurface {
    Normal,  // 1.0× modifier
    Ice      // 0.4× modifier (slippery)
}
```

---

## Items & Rarity

### ERarity - Upgrades/Encounters
**Location**: `Assets.Scripts.Inventory__Items__Pickups\ERarity.cs`

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

### EItemRarity - Item Drops
**Location**: `Assets.Scripts.Inventory__Items__Pickups.Items\EItemRarity.cs`

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

> **Note**: ERarity and EItemRarity have DIFFERENT value mappings!

---

## Weapons

### EWeapon
**Location**: `EWeapon.cs`

```csharp
public enum EWeapon {
    None = -1,
    FireStaff = 0,
    Bone = 1,
    Sword = 2,
    Revolver = 3,
    Aura = 4,
    Axe = 5,
    Bow = 6,
    Aegis = 7,          // Special: adds Thorns (EStat 3) to damage before multiplication
    Test = 8,
    LightningStaff = 9,
    Flamewalker = 10,
    Rockets = 11,
    Bananarang = 12,
    Tornado = 13,
    Dexecutioner = 14,
    Sniper = 15,
    Frostwalker = 16,
    SpaceNoodle = 17,
    DragonsBreath = 18,
    Chunkers = 19,
    Mine = 20,
    PoisonFlask = 21,
    BlackHole = 22,
    Katana = 23,
    BloodMagic = 24,
    BluetoothDagger = 25,
    Dice = 26,
    HeroSword = 27,
    CorruptSword = 28,
    Shotgun = 29,
    Scythe = 30
}
```

---

## Enemies

### EEnemyFlag - Enemy Type Flags (Bit Flags)
**Location**: `Assets.Scripts.Actors.Enemies\EEnemyFlag.cs`

> **Important**: These are BIT FLAGS for combining enemy types!

```csharp
public enum EEnemyFlag {
    None = 0,
    Elite = 1,              // 0x01 - Elite enemy
    Boss = 2,               // 0x02 - Boss enemy
    StageBoss = 4,          // 0x04 - Stage boss
    Challenge = 8,          // 0x08 - Challenge enemy
    SummonerMiniboss = 16,  // 0x10 - Summoner miniboss
    FinalBoss = 32,         // 0x20 - Final boss
    AnyBoss = 54            // 0x36 - Composite: Boss|StageBoss|SummonerMiniboss|FinalBoss
}
```

> **Note**: `AnyBoss = 54` is used for execute immunity checks (2+4+16+32 = 54)

---

## Maps & Challenges

### EMap
**Location**: `EMap.cs`

```csharp
// Map identifiers - exact values TBD via IDA
public enum EMap {
    // Values not fully documented
}
```

---

## Key Relationships

### Debuff Type → Duration
| EDebuff Value | Duration | Stacking |
|---------------|----------|----------|
| Poison (1) | Variable | Full stacking |
| Freeze (2) | 3.0s | Non-stacking |
| Burn (4) | 3.0s | Stacks damage |
| Stun (8) | 3.0s | Non-stacking |
| Bloodmark (64) | 4.0s | Stacks |

### EStat → Constraints
| EStat Index | Constraint |
|-------------|------------|
| 0 (MaxHealth) | Minimum 1.0 |
| 16 (Projectiles) | floor() to integer |
| 46 (ExtraJumps) | floor() to integer |

### ERarity vs EItemRarity Mapping
| ERarity | Value | EItemRarity | Value |
|---------|-------|-------------|-------|
| New | 0 | - | - |
| Common | 1 | Common | 0 |
| Uncommon | 2 | - | - |
| Rare | 3 | Rare | 1 |
| Epic | 4 | Epic | 2 |
| Legendary | 5 | Legendary | 3 |
| - | - | Corrupted | 4 |
| - | - | Quest | 5 |

---

## Function Address Reference

Key enum-related functions in native code:

| Function | Address | Purpose |
|----------|---------|---------|
| PlayerStatsNew::GetStat | 0x180446780 | Applies stat constraints |
| Enemy::AddDebuff | 0x1804AAF70 | Debuff stacking logic |
| Rarity::CalculateRarityWeights | 0x1803F3E70 | Luck-based weight calc |
| Rarity::GetMultiplier | 0x18042DFC0 | Rarity stat multipliers |
| PlayerMovementValues::GetMoveSpeed | 0x18044ECA0 | Speed with surface modifier |
