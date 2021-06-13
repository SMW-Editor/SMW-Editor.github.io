---
permalink: /docs/smw-rom/internal-rom-header/
title: "Internal ROM Header"
toc: true
sidebar:
    nav: "smw_rom_docs"
---

Internal ROM Header (or IRH) is a string of 64 bytes containing the game's metadata.
Because vanilla Super Mario World is LoROM, its IRH is located at byte `£007FC0` and ends at `£007FE0`.
This game uses IRH Ver. 1.

# Contents of IRH

| Location  | Data                    | Length \[bytes] | Value in vanilla SMW (US)                      | 
| --------- | ----------------------- | --------------- | ---------------------------------------------- |
| `£007FC0` | Internal ROM Name       | 21              | <code>"SUPER MARIOWORLD &nbsp; &nbsp; "</code> |
| `£007FD5` | Map Mode                | 1               | `0xF0` (LoROM)                                 |
| `£007FD6` | Cartridge Type          | 1               | `0x02` (ROM + SRAM + Battery)                  |
| `£007FD7` | ROM Size                | 1               | `0x09` (512 kB)                                |
| `£007FD8` | SRAM Size               | 1               | `0x01` (2 kB)                                  |
| `£007FD9` | Region/Destination Code | 1               | `0x01` (North America)                         |
| `£007FDA` | Developer ID            | 1               | `0x01` (Nintendo)                              |
| `£007FDB` | Version Number          | 1               | `0x00` (1.0)                                   |
| `£007FDC` | Checksum Complement     | 2               | `0x5F25`                                       |
| `£007FDE` | Checksum                | 2               | `0xA0DA`                                       |

The remaining 32 bytes contain the interrupt vectors which are outside the scope of this page.

# Data format

This section explains what each piece of the IRH is and how it is interpreted.

## Internal ROM name

Fixed size ASCII string: each of the 21 bytes is interpreted as a single ASCII character.
Only values between `0x20` and `0x7F` are allowed in this string, meaning that the special ASCII characters are not supported.

## Map mode

This byte can be represented as bits: `001F0HLh`

Where each letter corresponds to a flag:

| Flag | Value  | Description                   |
| ---- | -----  | ----------------------------- |
| F    | 0<br>1 | Regular speed<br>High speed   |
| H    | 0<br>1 | Non-ExHiROM<br>ExHiROM        |
| L    | 0<br>1 | Non-ExLoROM<br>ExLoROM        |
| h    | 0<br>1 | LoROM<br>HiROM                |

Note: only one or none of the last 3 bits (`H`, `L`, and `h`) may be set at the same time.
The `F` bit may be set or unset regardless of the values of other bits.
This yields the following range of values:

| Value        | Map Mode      |
| ------------ | ------------- |
| `0b00100000` | LoROM         |
| `0b00100001` | HiROM         |
| `0b00100010` | ExLoROM       |
| `0b00100100` | ExHiROM       |
| `0b00110000` | Fast LoROM    |
| `0b00110001` | Fast HiROM    |
| `0b00110010` | Fast ExLoROM  |
| `0b00110100` | Fast ExHiROM  |

## Cartridge type

This byte can be represented as a pair nibbles: `XP`.

The `P` nibble states what memory chips are used by the game cartridge:

| Values of `P` | Memory chips         |
| ------------- | -------------------- |
| `0` or `3`    | ROM                  |
| `1` or `4`    | ROM + SRAM           |
| `2` or `5`    | ROM + SRAM + Battery |
| `6`           | ROM + Battery        |

If `P` has a value greater than `2`, it means that the cartridge uses the expansion chip specified by nibble `X`:

| Values of `X` | Expansion chip |
| ------------- | -------------- |
| 0             | DSP            |
| 1             | SuperFX        |
| 2             | OBC-1          |
| 3             | SA1            |
| 4             | S-DD1          |
| 5             | S-RTC          |
| E             | Other          |
| F             | Custom         |

## ROM size

The actual size of the ROM equals <code>2<sup>n</sup></code>KiB, where `n` is the value of this byte.

## SRAM size

If this byte equals `0x00`, the size of SRAM is also 0.
Otherwise, the actual size of SRAM equals <code>2<sup>n</sup></code>KiB, where `n` is the value of this byte.

## Region/destination code

Just an enumeration of numbers corresponding to certain regions.

| Value  | Region                  |
| ------ | ----------------------- |
| `0x00` | Japan                   |
| `0x01` | Most of North America   |
| `0x02` | Most of Europe          |
| `0x03` | Sweden                  |
| `0x04` | Finland                 |
| `0x05` | Denmark                 |
| `0x06` | France                  |
| `0x07` | The Netherlands         |
| `0x08` | Spain                   |
| `0x09` | Germany                 |
| `0x0A` | Italy                   |
| `0x0B` | China                   |
| `0x0C` | Indonesia               |
| `0x0D` | South Korea             |
| `0x0E` | Global                  |
| `0x0F` | Canada                  |
| `0x10` | Brazil                  |
| `0x11` | Australia               |
| `0x12` | Other (1)               |
| `0x13` | Other (2)               |
| `0x14` | Other (3)               |

## Developer ID

A number assigned to each developer.

Nintendo's ID is `0x01`.

## Version number

This byte is the minor version number: the major version number is always assumed to be `1`.
For example, if this byte is `0x00`, the full version number is `1.0`.

## Checksum & complement check

The checksum is calculated by adding the values of every single byte in the ROM and keeping the 2 least significant bytes of the result.
Because the checksum itself is also used in this calculation, it comes together with a complement check equal to `¬checksum`, ensuring that they always add up to the same value:

```
checksum + complement = 0x1FE
```

Checksum and complement check are used by SNES for validating the ROM.
Bitwise XOR of the checksum and its complement check is always equal to `0xFFFF`.

This can be used to find the location of the IRH and deduce the mapping mode from that:
- If <code>readw(£007FDC) &oplus; readw(£007FDE) = 0xFFFF</code>, it means that the IRH is at `£007FC0` and we're in LoROM.
- If <code>readw(£00FFDC) &oplus; readw(£00FFDE) = 0xFFFF</code>, it means that the IRH is at `£00FFC0` and we're in HiROM.

# Acknowledgments

Information in the [Data format](/docs/smw-rom/internal-rom-header/#data-format) section is based on this video by Retro Game Mechanics Explained.
Thank you!

<a href   = "http://www.youtube.com/watch?feature=player_embedded&v=RAa561BoWwA"
   target = "_blank">
   <img src    = "http://img.youtube.com/vi/RAa561BoWwA/0.jpg"
        alt    = "Internal ROM Header - Super Nintendo Entertainment System Features Pt. 09c"
        border = "10" />
</a>