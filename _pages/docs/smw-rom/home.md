---
permalink: /docs/smw-rom/home/
title: "Documentation of the SMW ROM"
toc: true
sidebar:
    nav: "smw_rom_docs"
---

This site is intended for thorough documentation of data within the Super Mario World ROM, listing exact locations of everything and explaining the data formats.

Content posted here will be added incrementally as we learn about the ROM during development of the SMW Editor.

# About the ROM
Super Mario World ROM is exactly 512 kB large.
Some ROMs have 512 B of empty data at the beginning of the file: this is called the SMC header, it's simply ignored.

The address mapping of Super Mario World is LoROM.
<a href="https://en.wikibooks.org/wiki/Super_NES_Programming/SNES_memory_map" target="_blank">Here</a> you can learn more about the SNES memory map.

# Notation and nomenclature

In order to avoid confusion, the following sections explain how all kinds data are notated in this documentation, and what certain terms used here mean.

## Number literals

All number values that are not addresses may have a base prefix:
- Binary numbers (base 2) are suffixed with subscript 2, for example <code>11010100<sub>2</sub></code>
- Hexadecimal numbers (base 16) are suffixed with subscript 16, for example <code>DEADC0DE<sub>16</sub></code>
- Decimal numbers (base 10) have no suffix, for example `42`

## Arithmetics and comparisons

| Symbol                     | Function         |
| -------------------------- | ---------------- |
| `+`                        | addition         |
| `-`                        | subtraction      |
| `×`                        | multiplication   |
| `/`                        | division         |
| `mod`                      | modulo           |
| <code>n<sup>x</sup></code> | power            |
| `∨`                        | bitwise OR       |
| `∧`                        | bitwise AND      |
| <code>&oplus;</code>       | bitwise XOR      |
| `¬`                        | bitwise NOT      |
| `=`                        | equal            |
| `≠`                        | not equal        |
| `<`                        | less than        |
| `≤`                        | less or equal    |
| `>`                        | greater than     |
| `≥`                        | greater or equal |

## Templates

Sometimes, it is easier to describe a certain data format by representing a sequence of bytes/nibbles/bits as a template.

Take this template of a binary number as an example: <code>001F0HLh<sub>2</sub></code>.
It contains both the digits that can appear in a binary representation, as well as some letters.

The digits are set values which never vary.
On the other hand, letters are just placeholders and can take any possible value.
What each placeholder means is usually explained right below the template.

## Addresses

Addresses are base 16 numbers prefixed with currency symbols:
- PC addresses: pound sign `£`, for example `£007FC0`
- SNES addresses: dollar sign `$`, for example `$08D9F9`

PC addresses are just indices of bytes contained inside of a ROM file.
You will use them to access a specific byte of the ROM e.g. using a hex editor.

An SNES address is what the SNES uses for accessing data in different parts of its memory.
Its corresponding PC address depends on the [map mode](/docs/smw-rom/internal-rom-header/#map-mode) used by a ROM.

## Values at given addresses

To represent them, we use the hypothetical functions `read(addr)` (reads a byte) and `readw(addr)` (reads a word), where `addr` can be either a PC or SNES address.

## Text strings

ASCII strings are enclosed between a pair of quotation marks, for example `"Hello World"`.

## Units of data

- Bit = the fundamental unit of data.<br>Can be either 0 (unset) or 1 (set).
- Nibble = 4 bits.
- Byte = 2 nibbles = 8 bits.
- Word = 2 bytes = 4 nibbles = 16 bits.
