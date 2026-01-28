# Megabonk Weapon System Reference

> **Last Validated**: 2026-01-28 (addresses re-verified + upgrade system documented)
> **Source**: `D:\dev\megabonk\decompiled\Assembly-CSharp\`
> **IDA Verification**: WeaponData, WeaponUtility, UpgradeData, Rarity functions decompiled via IDA MCP

---

## Weapon Enum (EWeapon)

| ID | Name | Projectile Type | Category |
|----|------|----------------|----------|
| -1 | None | - | - |
| 0 | FireStaff | ProjectileBasic | Ranged/Magic |
| 1 | Bone | ProjectileBasic | Ranged |
| 2 | Sword | ProjectileMelee | Melee |
| 3 | Revolver | ProjectileBasic | Ranged/Hitscan |
| 4 | Aura | CombatAura | Aura |
| 5 | Axe | ProjectileAxe | Thrown/Boomerang |
| 6 | Bow | ProjectileArrow | Ranged |
| 7 | Aegis | AegisAttack | Defensive/Shield |
| 8 | Test | - | Debug |
| 9 | LightningStaff | ProjectileLightningBolt | Ranged/Magic |
| 10 | Flamewalker | Firefield | Ground/DoT |
| 11 | Rockets | ProjectileRocket (Rocket) | Ranged/Explosive |
| 12 | Bananarang | ProjectileBanana | Thrown/Boomerang |
| 13 | Tornado | ProjectileWhirlwind | Area/DoT |
| 14 | Dexecutioner | ProjectileDexecutioner | Melee/Zone |
| 15 | Sniper | ProjectileSniper | Ranged/Hitscan |
| 16 | Frostwalker | IceAura | Aura/Freeze |
| 17 | SpaceNoodle | LaserBeamAttack | Beam/Channel |
| 18 | DragonsBreath | ProjectileDragonsBreath | Cone/Channel |
| 19 | Chunkers | ChunkersAttack | Orbiting |
| 20 | Mine | ProjectileMines | Trap |
| 21 | PoisonFlask | ProjectilePoisonFlask | Thrown/AoE |
| 22 | BlackHole | ProjectileBlackHole | Area/Pull |
| 23 | Katana | ProjectileKatana | Melee/Zone |
| 24 | BloodMagic | ProjectileBloodMagic | Targeted/Magic |
| 25 | BluetoothDagger | ProjectileBluetooth | Homing |
| 26 | Dice | ProjectileDice | Thrown/RNG |
| 27 | HeroSword | ProjectileHeroSword | Melee/Projectile |
| 28 | CorruptSword | ProjectileCringeSword | Melee/Projectile |
| 29 | Shotgun | ProjectileShotgun | Ranged/Spread |
| 30 | Scythe | ProjectileScythe | Melee/Empowered |

---

## WeaponData Structure

**Location**: `WeaponData.cs`

Core weapon configuration stored in ScriptableObject:

```csharp
public class WeaponData : UnlockableBase, IUpgradable {
    public EWeapon eWeapon;              // Weapon type identifier
    public Texture icon;                  // UI icon
    public bool onlySpawnWhenCloseEnemies; // Targeting requirement

    // Base Stats
    public Dictionary<EStat, float> baseStats;  // Per-stat base values
    public float damage;                  // Base damage
    public float knockback;               // Base knockback force
    public float critChance;              // Base crit chance (0.0-1.0+)
    public EElement element;              // Elemental type

    // Projectile Stats
    public int projectiles;               // Number of projectiles per attack
    public int projectileBounces;         // Bounce/pierce count
    public float projectileSpeed;         // Projectile velocity
    public bool canBounce;                // Enables bouncing behavior

    // Timing
    public float attackDuration;          // Attack animation duration
    public float maxDuration;             // Maximum projectile lifetime
    public float endCooldown;             // Cooldown after attack
    public float burstTime;               // Burst attack window
    public float minBurstInterval;        // Minimum time between bursts

    // Size & Effects
    public float maxSizeMultiplier;       // Maximum scale multiplier
    public float effectDuration;          // Effect duration (DoT, debuff)
    public float procCoefficient;         // Item proc modifier (1.0 = full)
    public float spawnProjectileRange;    // Max spawn distance

    // Behavior Flags
    public EAmplificationMode amplificationMode;  // Bounce/Pierce/ShieldStack
    public bool useVision;                // Requires line of sight
    public bool canMultiHit;              // Can hit same enemy multiple times
    public bool hasCrosshair;             // Shows aiming crosshair
    public bool isAura;                   // Aura-type weapon

    // References
    public Vector3 spawnOffset;           // Spawn position offset
    public GameObject attack;             // Attack prefab
    public UpgradeData upgradeData;       // Upgrade tree data
    public MyAchievement AchievementRequirement;  // Unlock requirement
}
```

---

## WeaponData Base Stats Initialization

**Location**: `WeaponData$$Init` at `0x18039d830`

When a weapon is initialized, the following stats are added to the `baseStats` dictionary:

| EStat | Value | Description |
|-------|-------|-------------|
| 15 | 1.0 | Base multiplier (AttackSpeedMultiplier) |
| 12 | `this.damage` | Damage stat |
| 24 | `this.knockback` | Knockback force |
| 16 | `this.projectiles` | Projectile count (cast to float) |
| 45 | `this.projectileBounces` | Bounce/pierce count (cast to float) |
| 10 | `this.attackDuration` | Attack duration |
| 11 | `this.projectileSpeed` | Projectile velocity |
| 9 | 1.0 | Base multiplier (AttackDurationMultiplier) |
| 18 | `this.critChance` | Critical hit chance |
| 19 | 0.0 | CritDamage (initialized to 0) |

**GetBaseStat**: `WeaponData$$GetBaseStat` at `0x18039d610` retrieves values from this dictionary.

**GetUpgradeOffer**: `WeaponData$$GetUpgradeOffer` at `0x18039d800` delegates to `UpgradeData$$GetUpgradeOffer` at `0x180450070`.

---

## Upgrade System

**Location**: `UpgradeData$$GetUpgradeOffer` at `0x180450070`

### Rarity Multipliers
| ERarity | Value | Multiplier |
|---------|-------|------------|
| New | 0 | 1.0× |
| Common | 1 | 1.0× |
| Uncommon | 2 | 1.2× |
| Rare | 3 | 1.4× |
| Epic | 4 | 1.6× |
| Legendary | 5 | 2.0× |

### Upgrade Offer Generation

1. **Number of Stat Modifiers**:
   - For Common (rarity 1): 50% chance for 1 or 2 modifiers
   - For higher rarities: `(upgradeModifiers.Count > 1) + 1` (1-2 modifiers base)
   - **Campfire item** (EItem 41) adds extra modifier slots per stack

2. **Modifier Value Calculation**:
   ```
   upgradeValue = round(baseModifierValue × rarityMultiplier, 2)
   ```
   - Values rounded to 2 decimal places
   - Modifiers randomly selected from weapon's `upgradeModifiers` pool
   - Each modifier can only be selected once per upgrade

3. **Campfire Bonus Loop**:
   - For each Campfire stack owned:
     - If current modifier count < available pool size
     - Add +1 to modifier count (up to pool limit)

---

## Amplification Modes (EAmplificationMode)

```csharp
public enum EAmplificationMode {
    Bounce,       // Projectile bounces off walls/enemies
    Pierce,       // Projectile passes through enemies
    ShieldStack   // Shield stacking (Aegis only)
}
```

---

## Projectile Target Modes (EProjectileTargetMode)

```csharp
public enum EProjectileTargetMode {
    RandomDirection,        // Fire in random direction
    RandomTarget,          // Target random enemy
    Index0,                // Target closest/first enemy
    ClosestTargetInVision, // Target closest enemy in line of sight
    MoveDirection          // Fire in movement direction
}
```

---

## Weapon Categories

### Melee Weapons
Zone-based damage using `CheckZone()` method.

| Weapon | Projectile Class | Special Mechanic |
|--------|-----------------|------------------|
| Sword | ProjectileMelee | Standard melee swing |
| Katana | ProjectileKatana | Zone attack, testMultiplier |
| Dexecutioner | ProjectileDexecutioner | Large zone, effect spawning |
| HeroSword | ProjectileHeroSword | Moving projectile attack |
| CorruptSword | ProjectileCringeSword | Moving projectile (same as HeroSword) |
| Scythe | ProjectileScythe | **Big Hit System**: Every Nth hit is empowered (bigHitMultiplierSize, bigHitMultiplierDamage) |

**Scythe Big Hit Mechanic**:
- `nextHitIsBig` static flag controls empowered hits
- `bigHitCooldown` controls frequency
- Visual: Changes particle color (defaultColor vs bigHitColor)

### Ranged Projectiles
Standard projectile-based weapons.

| Weapon | Projectile Class | Special Mechanic |
|--------|-----------------|------------------|
| FireStaff | ProjectileBasic | Standard fireball |
| Bone | ProjectileBasic | Basic projectile |
| Revolver | ProjectileBasic | Hitscan-style |
| LightningStaff | ProjectileLightningBolt | Lightning strike |
| Bow | ProjectileArrow | Speed reduction over distance |
| Sniper | ProjectileSniper | Hitscan, maxDistance |
| Shotgun | ProjectileShotgun | Spread pattern (GetRawQuantity) |

**Arrow Speed Reduction**:
- `speedReduction` field controls deceleration
- Uses `ProjectileUtility::GetArrowSpeedReduction` at `0x1803F3B90`
- `ProjectileUtility::GetArrowSpeed` at `0x1803F3D60`

### Boomerang/Returning Weapons
Projectiles that return to player.

| Weapon | Projectile Class | Special Mechanic |
|--------|-----------------|------------------|
| Axe | ProjectileAxe | Moves to position then returns |
| Bananarang | ProjectileBanana | Multi-hit while returning, `returnTime`, `sqrCollectDistance` |

**Bananarang Mechanic**:
- `enemyHitCooldowns`: Per-enemy hit cooldown dictionary
- `numTimesEnemiesHitThisTick`: Tracks multi-hit in single tick
- `readyToCollectTime`: When projectile can be collected
- `dirToPlayer`: Return direction tracking

### Homing Weapons
Target-seeking projectiles.

| Weapon | Projectile Class | Special Mechanic |
|--------|-----------------|------------------|
| BluetoothDagger | ProjectileBluetooth | Direct homing to target |
| Rockets | Rocket | Homing missiles, `targetEnemy` tracking |
| BloodMagic | ProjectileBloodMagic | Targeted enemy attack |

**Rocket System**:
- Separate `Rocket` class (not ProjectileBase)
- `upTime`: Initial upward flight time
- `FindTarget()`: Acquires nearest enemy
- `procCoefficient`: Passed to damage container

### Aura Weapons
Continuous area effect weapons using `ConstantAttack` base class.

| Weapon | Attack Class | Special Mechanic |
|--------|--------------|------------------|
| Aura | CombatAura | Standard damage aura |
| Frostwalker | IceAura | Freeze aura, `GetFreezeTime()` |
| Chunkers | ChunkersAttack | Orbiting projectiles |

**ConstantAttack Base**:
```csharp
public abstract class ConstantAttack : MonoBehaviour {
    public WeaponBase weaponBase;
    protected abstract void Init();
    protected abstract void OnWeaponStatUpdate(EStat stat, EWeapon weapon);
    protected abstract void OnStatUpdate(EStat stat);
    public abstract float GetAuraRotationSpeed();
    public virtual bool IsManualRotation();
}
```

**Chunkers Mechanic**:
- Uses `RotatingProjectiles` component
- `amount`: Number of orbiting chunks
- `rotationSpeed`: Orbital velocity
- Attack/stop timing: `startTime`, `stopTime`, `nextStartTime`

### Channel Weapons
Continuous beam/cone attacks.

| Weapon | Attack Class | Special Mechanic |
|--------|--------------|------------------|
| DragonsBreath | ProjectileDragonsBreath | Cone attack, `rotationSpeed`, particle system |
| SpaceNoodle | LaserBeamAttack | Line beam, `whipSegments` for wavy effect |

**SpaceNoodle/LaserBeam Mechanic**:
- `linerenderer`: Visual beam
- `whipSegments`, `whipAmplitude`, `whipFrequency`: Wavy animation
- `laserRadius`: Collision width
- `isShooting` state tracking
- `enemyHitCooldowns`: Per-enemy hit tracking

**DragonsBreath Mechanic**:
- Uses particle system (`ParticleSystem ps`)
- `rotationSpeed`: Cone rotation
- `lingerTime`: Attack linger after stop
- `scaleOverTime`: Dynamic size changes
- `enemyHitCooldown`: DoT interval

### Ground/DoT Weapons
Leave persistent damaging zones.

| Weapon | Attack Class | Special Mechanic |
|--------|--------------|------------------|
| Flamewalker | Firefield | Ground fire trail |

**Firefield Mechanic** (`Assets.Scripts.Inventory__Items__Pickups.Weapons.Firefield`):
- `GetEffectiveRadius()`: Zone size
- `GetHitboxRadius()`: Damage collision size
- `IsWeaponAttack()`: Identifies as weapon damage
- Fixed intervals for damage check

### Area/Pull Weapons
Large area effects.

| Weapon | Projectile Class | Special Mechanic |
|--------|-----------------|------------------|
| BlackHole | ProjectileBlackHole | Enemy pull effect |
| Tornado | ProjectileWhirlwind | Moving area damage |

**BlackHole Mechanic**:
- `suckedEnemies`: HashSet of pulled enemies
- `maxSize`: Maximum growth
- `startFadeTime`: Fade out timing
- `moveTime`, `moveTimer`: Movement interpolation

**Tornado/Whirlwind Mechanic**:
- `enemyHitCooldowns`: Per-enemy cooldown
- `CheckRadiusDamage()`: Area hit detection
- Movement-based targeting

### Thrown/Explosive Weapons

| Weapon | Projectile Class | Special Mechanic |
|--------|-----------------|------------------|
| PoisonFlask | ProjectilePoisonFlask | Explodes on hit, applies poison |
| Mine | ProjectileMines | Trap, proximity trigger |
| Dice | ProjectileDice | RNG-based damage, roll 6 special |

**PoisonFlask Mechanic**:
- `explosionRadius`: AoE size
- `GetPoisonDuration()`: Poison debuff duration
- `GetNumPoisonStacks()`: Stack application count
- `ExplodeFlask()`: AoE damage + poison apply

**Dice Mechanic**:
- `diceRoll`: Current roll (1-6)
- `A_RollSix` event: Special trigger on 6
- `diceFx6`: Special effect for 6
- `explosionRadius`: Hitscan radius

### Special Weapons

| Weapon | Attack Class | Special Mechanic |
|--------|--------------|------------------|
| Aegis | AegisAttack | Shield system |

**Aegis Mechanic**:
- Shield-based defense weapon
- `currentAmount`: Active shield count
- `GetMaxShields()`: Maximum shields
- `UseShield()`: Consumes shield on hit
- `RegenShield()`: Shield regeneration
- `AmplifyShield()`: Shield stacking
- `A_Used`, `A_Regen` events

---

## Weapon Passives System

Special passive abilities for specific weapons.

**Base Class**: `WeaponPassive`
```csharp
public abstract class WeaponPassive {
    public Dictionary<EStat, StatModifiersContainer> statModifiers;
    protected WeaponBase weaponBase;
    public abstract void Init();
    public abstract void Cleanup();
}
```

**Factory**: `WeaponPassiveFactory::GetWeaponPassive(WeaponBase)`

### BloodMagic Passive
**Class**: `WeaponPassiveBloodMagic`
- `stacks`: Stack counter
- `stackChance`: Proc chance per kill
- `OnEnemyDied()`: Triggers on enemy death
- `rollCooldown`, `nextReadyTime`: Rate limiting
- `maxRollsUpgradesPerMinute`: Cap on stacks/minute

### Dice Passive
**Class**: `WeaponPassiveDice`
- `stacks`: Stack counter
- `critPer6`: Crit chance bonus per 6 rolled
- `accumulatedCritChance`: Total bonus crit
- `OnStackAdded()`: Called on roll 6
- `rollCooldown`, `nextRollTime`: Rate limiting

---

## WeaponBase Runtime Class

**Location**: `Assets.Scripts.Inventory__Items__Pickups.Weapons\WeaponBase.cs`

Runtime weapon instance managing state:

```csharp
public class WeaponBase {
    public WeaponData weaponData;
    public int level;
    public bool enabled;

    private Dictionary<EStat, float> weaponStats;
    private List<List<StatModifier>> upgrades;
    private WeaponPassive passive;

    public static Action<EStat, EWeapon> A_WeaponStatUpdate;

    // Core methods
    public void Enable();
    public void Disable();
    public void Use();
    public void Upgrade(List<StatModifier> upgradeOffer);
    public float GetValue(EStat stat);
}
```

---

## Weapon Inventory

**Location**: `Assets.Scripts.Inventory__Items__Pickups.Weapons\WeaponInventory.cs`

```csharp
public class WeaponInventory {
    public Dictionary<EWeapon, WeaponBase> weapons;

    public static Action<WeaponBase> A_WeaponAdded;
    public static Action<WeaponBase> A_WeaponRemoved;
    public static Action<WeaponBase> A_WeaponToggled;

    public void AddWeapon(WeaponData weaponData, List<StatModifier> upgradeOffer);
    public void ToggleWeapon(EWeapon eWeapon, bool enable);
    public int GetNumWeapons();
    public int GetWeaponLevel(EWeapon eWeapon);
    public bool IsMaxed();
    public bool HasAimableWeapon();
}
```

---

## Damage Calculation

**Location**: `WeaponUtility$$GetDamageContainer` at `0x180435010`

### Damage Pipeline (Verified via IDA decompilation)

1. **Knockback Calculation**:
   ```
   knockback = weaponStats[24] × PlayerStats.GetStat(24)
   ```

2. **Critical Hit Check**:
   ```
   critChance = PlayerStats.GetStat(18) + weaponStats[18]
   isCrit = DamageUtility.GetCritDamageMultiplier(critChance, &multiplier)
   ```

3. **Base Damage**:
   ```
   if (forceDamage == -1):
       baseDamage = weaponStats[12]  // EStat.Damage
       if (eWeapon == 7):  // Aegis special
           baseDamage = PlayerStats.GetStat(3) + baseDamage  // Add Thorns stat BEFORE mult
   else:
       baseDamage = forceDamage
   ```
   > **Note**: Aegis uses EStat 3 (Thorns) as bonus damage - shield retaliation boosts attack.

4. **Item Modifier Application**:
   ```
   damage = GetNewDamage(baseDamage, itemModifier)
   // GetNewDamage: StatComponents.GetFinalValue(playerDamageStats, itemModifierStats) × baseDamage
   ```

5. **Critical Damage**:
   ```
   if (isCrit):
       playerCritDmg = PlayerStats.GetStat(19)  // EStat.CritDamage
       weaponCritDmg = weaponStats[19]
       finalDamage = ((playerCritDmg + weaponCritDmg) × critMultiplier) × damage
   else:
       finalDamage = damage
   ```

**Key Functions** (Addresses verified 2026-01-28):
| Function | Address | Purpose |
|----------|---------|---------|
| WeaponUtility$$GetDamageContainer | 0x180435010 | Create damage container |
| WeaponUtility$$GetDamage | 0x180435A70 | Get weapon damage |
| WeaponUtility$$GetNewDamage | 0x180435D30 | Apply item modifiers to damage |
| WeaponUtility$$GetAttackQuantity | 0x180434CB0 | Get projectile count |
| WeaponUtility$$GetAttackSizeMultiplier | 0x180434D60 | Get size multiplier |
| WeaponUtility$$GetBurstInterval | 0x180434E00 | Get burst timing |
| WeaponUtility$$GetCritChance | 0x180434F10 | Get crit chance |
| WeaponUtility$$GetCritDamageMultiplier | 0x180434F90 | Get crit damage |
| WeaponUtility$$GetKnockback | 0x180435C90 | Get knockback force |
| WeaponUtility$$GetProjectileBounces | 0x180435EF0 | Get bounce count |
| WeaponUtility$$GetProjectileSpeed | 0x180435F70 | Get projectile speed |
| WeaponUtility$$GetDuration | 0x180435BF0 | Get attack duration |
| WeaponUtility$$GetWeaponCooldown | 0x180435FE0 | Get weapon cooldown |
| WeaponUtility$$GetWeaponRange | 0x1804362F0 | Get weapon range |
| WeaponUtility$$LightningStrike | 0x180436360 | Spawn lightning |
| WeaponUtility$$ChainLightning | 0x1804346B0 | Chain lightning logic |
| Rarity$$GetMultiplier | 0x18042DFC0 | Rarity stat multipliers |

---

## Weapon Attack System

**Location**: `Assets.Scripts.Inventory__Items__Pickups.Weapons.Attacks\WeaponAttack.cs`

```csharp
public class WeaponAttack {
    // Key methods at native addresses:
    // WeaponAttack::SpawnProjectile at 0x1803F8F60
    // WeaponAttack::StartAttack at 0x1803F9510
    // WeaponAttack::ProjectileHit at 0x1803F8D80
    // WeaponAttack::ProjectileDone at 0x1803F8CC0
}
```

---

## ProjectileBase Structure

**Location**: `Assets.Scripts.Inventory__Items__Pickups.Weapons.Projectiles\ProjectileBase.cs`

```csharp
public abstract class ProjectileBase : MonoBehaviour {
    public WeaponBase weaponBase;
    protected WeaponAttack weaponAttack;

    public float projectileRadius;
    public Vector3 direction;
    public int bounces;
    public int maxBounces;

    protected float expirationTime;
    protected float projectileSpeed;
    protected HashSet<Collider> hitEnemies;

    // Abstract methods for subclass implementation
    protected abstract bool TryInit(int projectileIndex);
    protected abstract Vector3 GetMovementDirection();
    protected abstract void MyFixedUpdate();
    protected abstract void MyUpdate();
    protected abstract void FindMovementDirection();
}
```

**Key Functions**:
| Function | Address | Purpose |
|----------|---------|---------|
| ProjectileBase::Set | 0x1803F1480 | Initialize projectile |
| ProjectileBase::HitEnemy | 0x1803F0A10 | Enemy collision |
| ProjectileBase::HitOther | 0x1803F0E10 | Non-enemy collision |
| ProjectileBase::StepMovement | 0x1803F17C0 | Movement update |
| ProjectileBase::CheckCollision | 0x1803F04C0 | Collision detection |

---

## Elemental Types (EElement)

```csharp
public enum EElement {
    Neutral,    // No elemental damage
    Lightning,  // Lightning element - stun
    Ice,        // Ice element - freeze
    Fire,       // Fire element - burn
    Bleed       // Bleed element - DoT
}
```

---

## Notes

### IL2CPP Limitations
All method implementations in decompiled C# are stubs. Actual weapon stats are:
1. Defined in Unity ScriptableObjects (WeaponData assets)
2. Calculated in native IL2CPP code at documented addresses

### Pool Sizes
`WeaponUtility::GetMaxProjectilesPoolSize` at `0x1803FE770` returns object pool sizes per weapon type for performance optimization.

### Multi-Hit System
- `canMultiHit` flag enables hitting same enemy multiple times
- `enemyHitCooldowns` dictionary tracks per-enemy cooldowns
- `hitEnemies` HashSet prevents duplicate hits in single frame

### Proc Coefficient
- Default: 1.0 (100% item proc rate)
- Override: -1.0 (use default)
- Applied to all item procs from weapon damage
