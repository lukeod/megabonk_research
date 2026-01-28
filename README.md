# Megabonk Research

Research notes and documentation for Megabonk reverse engineering and modding.

## Documentation

| File | Description |
|------|-------------|
| [mechanics.md](mechanics.md) | Core game mechanics, formulas, and systems |
| [enums.md](enums.md) | Complete enum reference with all values |
| [weapons.md](weapons.md) | Weapon system, projectiles, and attack mechanics |
| [items.md](items.md) | Item system overview |
| [items/properties/](items/properties/) | Individual item property documentation |

## Last Validated

- **Date**: 2026-01-28
- **Source**: Decompiled Assembly-CSharp (IL2CPP)
- **Method**: Automated deep-dive analysis of C# structure + IDA function list verification

## Key Findings Summary

### Corrected Values
- **EStat count**: 57 stats (indices 0-56), not 60
- **ExtraJumps**: EStat index 46, not 45 (ProjectileBounces is 45)
- **EDebuff**: Uses BIT FLAGS (Poison=1, Freeze=2, Burn=4, Stun=8), not sequential values
- **Rarity system**: Two separate enums (ERarity for upgrades, EItemRarity for drops)

### IL2CPP Limitations
All C# method implementations are stubs - actual logic is in native code at documented addresses. Formula verification requires IDA disassembly of GameAssembly.dll.

## Directory Structure

```
megabonk_research/
  mechanics.md      - Core mechanics documentation
  enums.md          - Complete enum definitions
  items.md          - Item system overview
  items/
    properties/     - Per-item documentation
```