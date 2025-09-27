# MegaBonk Complete Item Reference
*Extracted from decompiled IL2CPP constructors via IDA Pro MCP*

## Item Overview
Total Items Documented: 69

### EStat Reference
| ID | Stat Name |
|----|-----------|
| 0 | Max Health |
| 1 | HP Regeneration |
| 2 | Shield |
| 3 | Thorns/Retaliation |
| 4 | Armor |
| 5 | Evasion |
| 9 | Area/Radius Multiplier |
| 12 | Power/Damage |
| 15 | Attack Speed |
| 16 | Projectiles |
| 17 | Lifesteal |
| 18 | Critical Chance |
| 25 | Movement Speed |
| 26 | Jump Height |
| 30 | Luck |
| 32 | Time-based Damage |
| 38 | Difficulty |
| 47 | Overheal |

### Debuff Types
| ID | Type |
|----|------|
| 1 | Poison |
| 2 | Freeze |
| 4 | Burn |
| 8 | Stun/Lightning |
| 32 | Charm |
| 64 | Bleeding |

---

## Backpack
- **Constructor Address**: 0x1803FFFA0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| projectilesPerAmount | 1 | Per stack |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 16 | amount | amount (cast to float) |

---

## Beacon
- **Constructor Address**: 0x180400920

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| extraShrinesPerAmount | 2 | Per stack |
| healingRadiusPerAmount | 2.0 | Per stack |
| healingFractionPerInterval | 0.025 | Base value |

---

## BeefyRing
- **Constructor Address**: 0x180400DC0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| maxHpPerStack | 10 | Per stack |
| powerPerHpPerAmount | 0.002 | Per HP per stack |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 0 | amount * 10 | Max HP |
| 12 | currentHP * 0.002 * amount | Dynamic power based on HP |

### Special Mechanics
- Updates power stat dynamically based on current HP every 1 second

---

## Beer
- **Constructor Address**: 0x180401080

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| damagePerStack | 0.2 | Per stack |
| maxHealthPerStack | -0.05 | Per stack (negative) |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 12 | amount * 0.2 | Power/Damage increase |
| 0 | amount * -0.05 | Max HP reduction |

---

## BloodyCleaver
- **Constructor Address**: 0x180401730

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| stacksPerAmount | 1 | Per stack |
| chanceToStackPerAmount | 0.5 | 50% base chance per stack |

### Special Mechanics
- Applies bleeding debuff (ID 64) for 5.0 seconds on proc
- Subscribes to lifesteal proc events

---

## BobDead
- **Constructor Address**: 0x180401E10

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| unitsPerProjectile | 14.0 | Distance per projectile spawn |
| minSpawnTime | 0.05 | Minimum spawn interval |

### Special Mechanics
- Spawns ghost projectiles when player moves
- Damage: baseDamage * 1.5
- Uses EStat 10 modifier * 10.0

---

## Bonker
- **Constructor Address**: 0x180402640

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| baseChance | 0.02 | 2% base proc chance |
| baseDamageMultiplier | 20.0 | Base damage multiplier |
| chancePerStack | 0.015 | 1.5% per stack |
| damageMultiplierPerStack | 10.0 | Per stack |
| radius | 3.5 | Base radius |
| maxProcsPerTick | 5 | Maximum procs per tick |

### Special Mechanics
- Area damage with 1.25x knockback
- Final radius: radius * amount + 7.0

---

## Borgor
- **Constructor Address**: 0x180402DF0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| baseChance | 0.02 | 2% base proc chance |
| chancePerAmount | 0.01 | 1% per stack |
| ratioHeal | 0.08 | 8% ratio heal |
| flatHealPerAmount | 2 | Per stack |
| flatHeal | 10 | Base flat heal |

### Special Mechanics
- Heals on enemy death with chance

---

## BrassKnuckles
- **Constructor Address**: 0x1804031C0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| damagePerAmount | 0.2 | 20% damage per stack |
| radius | 7.0 | Melee range check |

### Special Mechanics
- Only applies bonus if enemy is within 7.0 units

---

## Cactus
- **Constructor Address**: 0x180404130

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| damagePerAmount | 5.0 | Per stack |
| numProjectilesPerAmount | 2 | Per stack |

### Special Mechanics
- Counter-attacks when player takes damage
- Projectiles: amount * 2 + 1

---

## Campfire
- **Constructor Address**: 0x1804048B0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| healthRegenPerMinutePerAmount | 1100.0 | Per stack |
| setupTime | 0.6 | Setup time in seconds |
| distThreshold | 1.75 | Activation distance |

### Special Mechanics
- Activates when player stays within 1.75 units for 0.6 seconds

---

## Chonkplate
- **Constructor Address**: 0x180404BB0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| overhealPerAmount | 0.75 | Per stack |
| lifestealPerAmount | 0.2 | Per stack |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 47 | amount * 0.75 | Overheal |
| 17 | amount * 0.2 | Lifesteal |

---

## Clover
- **Constructor Address**: 0x180404E10

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| luckPerAmount | 0.075 | Per stack |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 30 | amount * 0.075 | Luck |

---

## CowardsCloak
- **Constructor Address**: 0x180405580

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| speedPerAmount | 0.05 | Base speed per stack |
| speedPerStack | 0.3 | Speed per coward stack |
| maxStacks | 2 | Maximum coward stacks |
| stacksPerAmount | 2 | Stacks gained per item |

### Special Mechanics
- Gains speed stacks when taking damage
- Stacks decay over time

---

## CreditCardGreen
- **Constructor Address**: 0x180405AE0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| luckPerChestPerAmount | 0.02 | Luck per chest per stack |

### Special Mechanics
- Responds to chest opening events

---

## CreditCardRed
- **Constructor Address**: 0x180406010

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| damagePerChestAmount | 0.025 | Damage per chest |

### Special Mechanics
- Responds to chest opening events

---

## CursedDoll
- **Constructor Address**: 0x180406CA0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| maxNumCursedEnemies | 1 | Initial maximum |
| damageMaxHpPercentage | 0.3 | 30% of enemy max HP |
| amountPerDoll | 2 | Per stack |
| attackCooldown | 1.0 | Between attacks |

### Special Mechanics
- Curses enemies up to amount * 2 limit
- Damages cursed enemies for 30% max HP (70% base damage for bosses)

---

## DemonBlade
- **Constructor Address**: 0x180407420

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| critChance | 0.01 | 1% crit chance |
| healChancePerStack | 0.25 | 25% heal chance per stack |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 18 | 0.01 | Critical chance |

---

## DemonicBlood
- **Constructor Address**: 0x180407AE0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| maxStacksPerAmount | Function result | From sub_180338070 |

### Special Mechanics
- Gains stacks on enemy death
- Applies HP stat bonus based on stacks

---

## DemonicSoul
- **Constructor Address**: 0x180408150

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| maxStacksPerAmount | Function result | From sub_180338070 |

### Special Mechanics
- Gains stacks on enemy death
- Applies attack damage bonus based on stacks

---

## Dragonfire
- **Constructor Address**: 0x180408780

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| procChancePerAmount | 0.15 | Per stack |
| burnChancePerAmount | 0.15 | Per stack |

### Special Mechanics
- Creates firefields on proc
- Uses hyperbolic scaling for chances
- Applies burn debuff for 3.0 seconds

---

## EagleClaw
- **Constructor Address**: 0x180408BC0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| procChancePerAmount | 0.08 | Per stack |
| damageAdditionPerAmount | 0.66 | Per stack |
| knockupForce | 3.5 | Base knockup |

### Special Mechanics
- Only adds damage when enemy is airborne
- Applies knockup on proc

---

## ElectricPlug
- **Constructor Address**: 0x1804097A0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| radius | 13.0 | Base radius |
| radiusPerAmount | 4.0 | Per stack |
| targets | 15 | Base targets |
| targetsPerAmount | 5 | Per stack |

### Special Mechanics
- Chain lightning when player takes damage
- Radius: EStat9 * radiusPerAmount + 12.0

---

## EnergyCore
- **Constructor Address**: 0x180409D50

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| orbsPerAmount | 2 | Per stack |
| numOrbs | 4 | Base orbs |
| cooldown | 4.0 | Between volleys |

### Special Mechanics
- Fires orb volleys periodically
- Orbs: amount * 2 + 5

---

## FlappyFeathers
- **Constructor Address**: 0x18040BDF0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| speedBoostPerAmount | 1.8 | Per stack |
| jumpHeightAdditionPerAmount | 0.15 | Per stack |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 26 | amount * 0.15 | Jump height |

### Special Mechanics
- Speed boost on jump

---

## GamerGoggles
- **Constructor Address**: 0x18040C330

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| maxDamagePerAmount | 1.0 | Per stack |
| updateCooldown | 1.0 | Update interval |

### Special Mechanics
- Damage boost when health < 50%
- Damage = (0.5 - healthPercent) * 2 * maxDamage

---

## Gasmask
- **Constructor Address**: 0x18040CAE0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| armorPerStack | 0.005 | Per poisoned enemy |
| overhealPerStack | 0.005 | Per poisoned enemy |
| maxArmorPerAmount | 0.4 | Cap per stack |
| maxOverhealPerAmount | 0.25 | Cap per stack |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 47 | Variable | Based on poisoned enemies |
| 4 | Variable | Based on poisoned enemies |

---

## Ghost
- **Constructor Address**: 0x18040D050

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| numGhostsPerAmount | 6 | Per stack |

### Special Mechanics
- Spawns ghosts on interaction

---

## GiantFork
- **Constructor Address**: 0x18040D1D0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| critChancePerAmount | 0.15 | Per stack |
| megaCritChancePerAmount | 0.14 | Per stack |
| megaCritDamageMultiplier | 4.0 | Mega crit damage |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 18 | amount * 0.15 | Critical chance |

---

## GlovesBlood
- **Constructor Address**: 0x18040DA40

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| cooldown | 9.0 | Seconds |
| baseDamageMultiplier | 3.15 | Damage multiplier |
| baseRadius | 10.0 | Effect radius |
| healPercentage | 0.075 | 7.5% heal |

### Special Mechanics
- Explosion on hit with healing
- Applies bleeding debuff for 5 seconds

---

## GlovesCursed
- **Constructor Address**: 0x18040E3F0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| procChancePerAmount | 0.05 | Per stack |
| difficultyPerAmount | 0.1 | Per stack |
| maxHpMultiplierPerAmount | 0.8 | Per stack |
| baseDamageMultiplier | 0.85 | Damage scaling |
| baseRadius | 4.0 | Area of effect |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 0 | pow(0.8, amount) | Max HP multiplier |
| 38 | amount * 0.1 | Difficulty |

---

## GlovesLightning
- **Constructor Address**: 0x18040EC20

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| cooldown | 10.0 | Seconds |
| baseDamageMultiplier | 3.0 | Damage scaling |
| baseRadius | 8.0 | Area of effect |

### Special Mechanics
- Lightning damage to all enemies in radius
- Applies lightning debuff for 3 seconds

---

## GlovesPoison
- **Constructor Address**: 0x18040F360

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| cooldown | 9.5 | Seconds |
| baseDamageMultiplier | 1.5 | Damage scaling |
| baseRadius | 15.0 | Area of effect |
| poisonStacksPerAmount | 10 | Per item |

### Special Mechanics
- Poison damage to all enemies in radius
- Applies poison debuff for 5 seconds

---

## GlovesPower
- **Constructor Address**: 0x18040FD30

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| knockbackForce | 9999.0 | Knockback strength |
| procChancePerAmount | 0.08 | Per stack |
| radiusPerAmount | 5.0 | Per stack |
| radius | 8.0 | Base radius |

### Special Mechanics
- Massive knockback on proc
- Cooldown: clamp(3.2 - amount*0.2, 0.2, 2.0)
- Radius: amount*5.0 + 10.0

---

## GoldenShield
- **Constructor Address**: 0x1804104E0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| chancePerAmount | 1.0 | Per stack |
| goldPerAmount | 3 | Base gold per stack |

### Special Mechanics
- Gold on taking damage
- Scales with character level (capped at 20 bonus)

---

## GoldenSneakers
- **Constructor Address**: 0x180410890

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| goldPerMeterBase | 0.05 | Base gold per meter |
| checkInterval | 0.5 | Check interval |

### Special Mechanics
- Generates gold based on distance traveled
- goldPerMeter = (amount-1)*0.025 + 0.05

---

## GrandmasSecretTonic
- **Constructor Address**: 0x180410F70

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| critChanceTotal | 0.02 | Total crit chance |
| baseRadius | 3.0 | Base radius |
| radiusPerAmount | 1.0 | Per stack |
| maxRadius | 8.0 | Maximum radius |
| damageSpreadMultiplier | 0.5 | Spread damage |
| procChance | 0.5 | On crit |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 18 | 0.02 | Critical chance |

---

## GymSauce
- **Constructor Address**: 0x18040C050

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| damagePerAmount | 0.1 | Per stack |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 12 | amount * 0.1 | Damage |

---

## HolyBook
- **Constructor Address**: 0x180411D80

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| maxHpPerAmount | 100.0 | Per stack |
| hpRegenPerAmount | 50.0 | Per stack |
| overhealPerAmount | 0.25 | Per stack |
| radiusPerAmount | 1.0 | Per stack |
| cooldown | 1.5 | Seconds |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 0 | amount * 100.0 | Max HP |
| 1 | amount * 50.0 | HP Regen |
| 47 | amount * 0.25 | Overheal |

---

## IceCrystal
- **Constructor Address**: 0x180412060

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| procChancePerAmount | 0.075 | Per stack |

### Special Mechanics
- Freeze debuff on proc
- Duration: clamp(amount*0.1+2.5, 0, 6)
- Chance: hyperbolic scaling with 0.75 factor

---

## IceCube
- **Constructor Address**: 0x180412610

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| procChancePerAmount | 0.2 | Per stack |
| freezeChancePerAmount | 0.4 | Per stack |
| damageRatio | 0.8 | Base damage |
| damageRatioPerAmount | 0.4 | Per stack |

### Special Mechanics
- Ice projectiles on proc
- Damage: 80% + 40% per stack
- Freeze for 3.0 seconds

---

## IdleJuice
- **Constructor Address**: 0x180412E70

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| damagePerAmount | 1.0 | Per stack |
| damagePerSecond | 0.04 | Damage gain/sec |
| setupTime | 0.6 | Setup time |
| distThreshold | 1.75 | Distance threshold |

### Special Mechanics
- Campfire system like Campfire item
- Accumulates damage while idle

---

## JoesDagger
- **Constructor Address**: 0x1804144D0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| attackDamagePerProc | 0.01 | Per proc |
| executionChancePerAmount | 0.01 | Per stack |
| maxStacks | 999999 | Maximum stacks |

### Special Mechanics
- 1% execution chance
- Accumulates damage on enemy hits

---

## Kevin
- **Constructor Address**: 0x180414E20

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| damageChancePerAmount | 0.25 | 25% per stack |

### Special Mechanics
- Self-damage system
- Responds to enemy damage events

---

## Key
- **Constructor Address**: 0x180414FE0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| chancePerStack | 0.1 | Per stack |

### Special Mechanics
- Diminishing returns: chance = amount*0.1/(amount*0.1+1)

---

## LeechingCrystal
- **Constructor Address**: 0x1804152A0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| regenAdditivePerAmount | -0.5 | Per stack (negative) |
| maxHpPerAmount | 50.0 | Per stack |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 0 | amount * 50.0 | Max HP |
| 1 | amount * -0.5 | HP Regen (negative) |

---

## LightningOrb
- **Constructor Address**: 0x180415900

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| procChancePerAmount | 0.25 | Per stack |
| stunChancePerAmount | 0.25 | Per stack |
| baseRadius | 40.0 | Constant |
| damageRatioPerAmount | 0.4 | Per stack |

### Special Mechanics
- Lightning strikes random enemy in radius
- Stun for 3.0 seconds on proc
- Hyperbolic scaling for chances

---

## Medkit
- **Constructor Address**: 0x180415C40

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| hpRegenPerAmount | 45.0 | Per stack |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 1 | amount * 45.0 | HP Regen |

---

## Mirror
- **Constructor Address**: 0x180416A80

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| cooldown | 8.0 | Base cooldown |
| minCooldown | 4.0 | Minimum |
| damagePerAmount | 0.25 | Per stack |

### Special Mechanics
- Reflects damage back to attackers
- Cooldown: max(8-amount, 4)
- Damage multiplier: amount*0.25 + 1.0

---

## MoldyCheese
- **Constructor Address**: 0x180421870

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| chanceToStackPerAmount | 0.4 | Per stack |

### Special Mechanics
- Applies poison debuff for 5.0 seconds
- Tracks poison application statistics

---

## Oats
- **Constructor Address**: 0x180421AC0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| hpPerAmount | 25.0 | Per stack |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 0 | amount * 25.0 | Max HP |

---

## PhantomShroud
- **Constructor Address**: 0x180422330

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| evasionPerAmount | 0.05 | 5% per stack |
| damageMultiplierPerAmount | 0.5 | Per stack |
| speedAdditionPerAmount | 0.15 | Per stack |
| timeout | 2.0 | Initial timeout |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 5 | amount * 0.05 | Evasion |

### Special Mechanics
- Speed/damage boost on evade
- Timeout: (amount-1)*0.5 + 3.0, max 6.0

---

## QuinsMask
- **Constructor Address**: 0x180422B10

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| thornsPerAmount | 20.0 | Per stack |
| baseRadius | 5.0 | Base radius |
| radiusPerAmount | 1.0 | Per stack |
| maxRadius | 10.0 | Maximum |
| damageSpreadMultiplier | 0.5 | Spread damage |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 3 | amount * 20.0 | Thorns |

---

## Rollerblades
- **Constructor Address**: 0x180422F10

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| maxAttackSpeedPerAmount | 0.4 | 40% per stack |
| updateStatsInterval | 0.25 | Update frequency |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 15 | Dynamic | Based on movement speed |

### Special Mechanics
- Attack speed scales with movement speed

---

## Scarf
- **Constructor Address**: 0x180423470

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| damageAddPerAmount | 0.33 | 33% per stack |

### Special Mechanics
- Damage bonus based on grounded state

---

## Skuleg
- **Constructor Address**: 0x180423810

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| difficultyPerAmount | 0.07 | Per stack |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 38 | amount * 0.07 | Difficulty |

---

## SluttyCannon
- **Constructor Address**: 0x180423E20

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| procChancePerAmount | 0.2 | Per stack |
| damageRatioPerAmount | 0.4 | Per stack |
| maxProcsPerTick | 4 | Maximum procs |

### Special Mechanics
- Spawns rockets on proc
- Hyperbolic scaling capped at 60%
- Damage: amount*0.4 + 1.0

---

## SoulHarvester
- **Constructor Address**: 0x180424800

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| numProjectiles | 2 | Base projectiles |

### Special Mechanics
- Responds to enemy deaths
- Damage multiplier = amount

---

## SpeedBoi
- **Constructor Address**: 0x180425570

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| damageMultiplierDuringFreeze | 2.0 | During slow |
| durationPerAmount | 2.0 | Per stack |
| normalCooldown | 10.0 | Base cooldown |
| slowdownHpRatio | 0.5 | HP threshold |

### Special Mechanics
- Duration: amount*2 + 8, max 15
- Slows time when HP < 50%

---

## SpicyMeatball
- **Constructor Address**: 0x180425B80

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| baseRadius | 3.0 | Base radius |
| radiusPerAmount | 1.0 | Per stack |
| maxRadius | 8.0 | Maximum |
| damageSpreadMultiplier | 0.65 | Spread damage |
| procChance | 0.25 | 25% chance |
| maxProcsPerTick | 50 | Maximum procs |

### Special Mechanics
- Chain explosions on hit
- Radius scales with EStat 9

---

## SpikyShield
- **Constructor Address**: 0x1804261A0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| armorPerAmount | 0.1 | Per stack |
| retaliationPerArmorPerAmount | 200.0 | Per armor per stack |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 4 | 0.1 | Armor |
| 3 | Dynamic | amount * 200 * current armor |

### Special Mechanics
- Updates retaliation damage based on current armor

---

## TacticalGlasses
- **Constructor Address**: 0x1804263F0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| damagePerAmount | 0.2 | Per stack |

### Special Mechanics
- Bonus damage against enemies with >90% health

---

## TimeBracelet
- **Constructor Address**: 0x180400680

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| damagePerAmount | 0.08 | Per stack |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 32 | amount * 0.08 | Time damage |

---

## ToxicBarrel
- **Constructor Address**: 0x180426C30

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| radiusPerAmount | 1.0 | Per stack |
| poisonStacksPerAmount | 5 | Per stack |
| cooldown | 0.25 | Seconds |
| poisonDuration | 5.0 | Seconds |

### Special Mechanics
- Poison area on player damage
- Radius: amount*1.0 + 7.0

---

## TurboSocks
- **Constructor Address**: 0x180403080

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| moveSpeedPerAmount | 0.15 | Per stack |

### Stat Modifiers
| EStat | Value | Calculation |
|-------|-------|-------------|
| 25 | amount * 0.15 | Movement speed |

---

## UnstableTransfusion
- **Constructor Address**: 0x180427280

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| chanceToStackPerAmount | 0.27 | Per stack |

### Special Mechanics
- Applies bleeding debuff for 5.0 seconds
- Stacking calculation with floor and remainder

---

## WeebHeadset
- **Constructor Address**: 0x1804273A0

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| charmChancePerAmount | 0.02 | Per stack |
| durationPerAmount | 1.5 | Per stack |
| maxProcsPerTick | 100 | Maximum |

### Special Mechanics
- Charms enemies (debuff 32)
- Duration: amount*1.5 + 5.0
- Hyperbolic scaling, min 0.02

---

## Wrench
- **Constructor Address**: 0x180427620

### Base Properties
| Property | Value | Notes |
|----------|-------|-------|
| chargeSpeedIncrease | 0.04 | Per stack |
| chargeRewardIncrease | 0.075 | Per stack |

### Special Mechanics
- Affects equipment charge mechanics

---

## ZaWarudo
- **Constructor Address**: 0x1803FFD80

### Base Properties
No properties set in constructor

### Special Mechanics
- Implementation not found in constructor code
- Likely placeholder or special implementation elsewhere

---

## Analysis Summary

### Items by Category

**Health/Healing (12 items)**:
BeefyRing, Beer, Borgor, Chonkplate, HolyBook, LeechingCrystal, Medkit, Oats, Gasmask, DemonicBlood, DemonBlade, GlovesBlood

**Damage/Power (17 items)**:
Beer, Bonker, BrassKnuckles, DemonBlade, DemonicSoul, EagleClaw, GiantFork, GymSauce, TacticalGlasses, TimeBracelet, GamerGoggles, Scarf, SpeedBoi

**Elemental/Status (15 items)**:
BloodyCleaver, Dragonfire, IceCrystal, IceCube, LightningOrb, MoldyCheese, ToxicBarrel, UnstableTransfusion, GlovesPoison, GlovesLightning, ElectricPlug, WeebHeadset

**Movement/Speed (6 items)**:
CowardsCloak, FlappyFeathers, GoldenSneakers, Rollerblades, TurboSocks, PhantomShroud

**Defensive (6 items)**:
Chonkplate, GoldenShield, Mirror, PhantomShroud, QuinsMask, SpikyShield

**Economy (3 items)**:
CreditCardGreen, CreditCardRed, GoldenShield

**Utility/Special (10 items)**:
Backpack, Beacon, Campfire, Clover, CursedDoll, Ghost, IdleJuice, Key, Wrench, ZaWarudo

### Scaling Mechanics

**Hyperbolic Scaling**: Dragonfire, EagleClaw, IceCrystal, IceCube, LightningOrb, SluttyCannon, WeebHeadset, Key

**Dynamic Updates**: BeefyRing, GamerGoggles, Gasmask, Rollerblades, SpikyShield

**Event-Driven**: Many items use Unity event system for reactive behavior

### Common Patterns
- Most items scale linearly with amount (stack count)
- Proc chances often use hyperbolic scaling to prevent 100% rates
- Cooldown-based items typically range from 0.25s to 10s
- Debuff durations typically 3-5 seconds
- Area effects commonly scale with EStat 9