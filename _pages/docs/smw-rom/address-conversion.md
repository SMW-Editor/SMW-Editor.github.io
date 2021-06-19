---
permalink: /docs/smw-rom/address-conversion/
title: "Address Conversion"
toc: true
sidebar:
    nav: "smw_rom_docs"
---

The same SNES address will correspond to a different PC address depending on the <a href="https://en.wikibooks.org/wiki/Super_NES_Programming/SNES_memory_map" target="_blank">map mode</a> of the ROM.
This page lists the algorithms used to convert between SNES and PC addresses in different map modes.

All algorithms here are based on the source code of <a href="https://github.com/RPGHacker/asar/blob/69a9a241fde29ed6b63ebdf1ae5a2684ce5eac16/src/asar/libsmw.h" target="_blank">ASAR</a>, a popular SNES assembler.

# LoROM

PC to SNES:
```
in_WRAM := SNES & FE_0000₁₆ = 7E_0000₁₆
in_MISC := SNES & 70_8000₁₆ = 70_0000₁₆
in_SRAM := SNES & 40_8000₁₆ = 0

if not (in_WRAM or in_MISC or in_SRAM):
    PC := ((SNES & 7F_0000₁₆) >> 1) | (SNES & 7FFF₁₆)
else:
    invalid address
```

SNES to PC:
```
if PC < 40_0000₁₆:
    SNES := ((PC << 1) & 7F_0000₁₆) | (PC & 7FFF₁₆) | 80_8000₁₆
else:
    invalid address
```

# HiROM

PC to SNES:
```
if PC < 40_0000₁₆:
    SNES := PC | C0_0000₁₆
else:
    invalid address
```

SNES to PC:
```
in_WRAM := SNES & FE_0000₁₆ = 7E_0000₁₆
in_MISC := SNES & 40_8000₁₆ = 0

if not (in_WRAM or in_MISC):
    PC := SNES & 3F_FFFF₁₆
else:
    invalid address
```

# Ex LoROM

PC to SNES:
```
if PC < 80_0000₁₆:
    if PC & 40_0000₁₆ ≠ 0:
        A := PC - 40_0000₁₆
        M := 8000₁₆
    else:
        A := PC
        M := 808000₁₆
    SNES := ((A << 1) & 7F_0000₁₆) | (A & 7FFF₁₆) | M
else:
    invalid address
```

SNES to PC:
```
in_WRAM_SRAM  := SNES & F0_0000₁₆ = 70_0000₁₆
outside_LoROM := SNES & 40_8000₁₆ = 0

if not (in_WRAM_SRAM or outside_LoROM):
    PC := (((SNES & 7F_0000₁₆) >> 1) | (SNES & 7FFF₁₆))
    if SNES & 80_0000₁₆ ≠ 0:
        PC := PC + 40_0000₁₆
else:
    invalid address
```

# Ex HiROM

PC to SNES:
```
if PC < 80_0000₁₆:
    if SNES & 40_0000₁₆ ≠ 0:
        PC := SNES
    else:
        PC := SNES | C0_0000₁₆
else:
    invalid address
```

SNES to PC:
```
in_WRAM := SNES & FE_0000₁₆ ≠ 7E_0000₁₆
in_MISC := SNES & 40_8000₁₆ ≠ 0

if not (in_WRAM or in_MISC):
    if SNES & 80_0000₁₆ ≠ 0:
        PC := (SNES & 3F_FFFF₁₆)
    else:
        PC := (SNES & 3F_FFFF₁₆) | 40_0000₁₆
else:
    invalid address
```

# SFX ROM

PC to SNES:
```
if PC < 20_0000₁₆:
    SNES := ((PC << 1) & 7F_0000₁₆) | (PC & 7FFF₁₆) | 8000₁₆
else:
    invalid address
```

SNES to PC:
```
in_WRAM_SRAM_openbus := SNES & 60_0000₁₆ = 60_0000₁₆
in_MISC              := SNES & 40_8000₁₆ = 0
in_fastROM           := SNES & 80_0000₁₆ = 80_0000₁₆

if not (in_WRAM_SRAM_openbus or in_MISC or in_fastROM):
    if SNES & 40_0000₁₆ ≠ 0:
        PC = SNES & 3F_FFFF₁₆
    else:
        PC = ((SNES & 7F_0000₁₆) >> 1) | (SNES & 7FFF₁₆)
else:
    invalid address
```

# SA1 ROM

PC to SNES:
```
if SA1_BANKS.contains(PC & 70_0000₁₆):
    BANK := SA1_BANKS.index_of(PC & 70_0000₁₆) << 21
    SNES := BANK | ((SNES & 0F_8000₁₆) << 1) | (SNES & 7FFF₁₆) | 8000₁₆
else:
    invalid address
```

SNES to PC:
```
if SNES & 40_8000₁₆ = 8000₁₆:
    B  := (addr & E0_0000₁₆) >> 21
    PC := SA1_BANKS[B] | ((addr & 1F_0000₁₆) >> 1) | (addr & 00_7FFF₁₆)
else if SNES & C0_0000₁₆ = C0_0000₁₆:
    B  := ((addr & 10_0000₁₆) >> 20) | ((addr & 20_0000₁₆) >> 19)
    PC := SA1_BANKS[B] | (addr & 0F_FFFF₁₆)
else:
    invalid address
```

Where:
```
SA1_BANKS[0] := 0 << 20
SA1_BANKS[1] := 1 << 20
SA1_BANKS[4] := 2 << 20
SA1_BANKS[5] := 3 << 20
```

SA1 banks 2, 3, 6, and 7 are not used.

# Big SA1 ROM

PC to SNES:
```
if PC < 80_0000₁₆:
    if PC & 40_0000₁₆ = 40_0000₁₆:
        SNES := PC | C0_0000₁₆
    else:
        SNES := ((PC << 1) & 3F_0000₁₆) | (PC & 7FFF₁₆)
        if PC & 60_0000₁₆ = 20_0000₁₆:
            SNES := SNES | 8000₁₆
        else if PC & 60_0000₁₆ = 0:
            SNES := SNES | 80_8000₁₆
else:
    invalid address
```

SNES to PC:
```
in_HiROM := SNES & C0_0000₁₆ = C0_0000₁₆
in_LoROM := SNES & C0_0000₁₆ = either(0, 80_0000₁₆)

if in_HiROM:
    PC := (SNES & 3F_FFFF₁₆) | 40_0000₁₆
else if in_LoROM and SNES & 8000₁₆ ≠ 0:
    PC := ((SNES & 80_0000₁₆) >> 2) | 
          ((SNES & 3F_0000₁₆) >> 1) | (SNES & 7FFF₁₆)
else:
    invalid address
```
