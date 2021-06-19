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

| Location  | Data                    | Length \[bytes] | Value in vanilla SMW (US)                           |
| --------- | ----------------------- | --------------- | --------------------------------------------------- |
| `£007FC0` | Internal ROM Name       | 21              | <code>"SUPER MARIOWORLD &nbsp; &nbsp; "</code>      |
| `£007FD5` | Map Mode                | 1               | <code>F0<sub>16</sub></code> (LoROM)                |
| `£007FD6` | Cartridge Type          | 1               | <code>02<sub>16</sub></code> (ROM + SRAM + Battery) |
| `£007FD7` | ROM Size                | 1               | <code>09<sub>16</sub></code> (512 kB)               |
| `£007FD8` | SRAM Size               | 1               | <code>01<sub>16</sub></code> (2 kB)                 |
| `£007FD9` | Region/Destination Code | 1               | <code>01<sub>16</sub></code> (North America)        |
| `£007FDA` | Developer ID            | 1               | <code>01<sub>16</sub></code> (Nintendo)             |
| `£007FDB` | Version Number          | 1               | <code>00<sub>16</sub></code> (1.0)                  |
| `£007FDC` | Checksum Complement     | 2               | <code>5F25<sub>16</sub></code>                      |
| `£007FDE` | Checksum                | 2               | <code>A0DA<sub>16</sub></code>                      |

The remaining 32 bytes contain the interrupt vectors which are outside the scope of this page.

# Data format

This section explains what each piece of the IRH is and how it is interpreted.

## Internal ROM name

Fixed size ASCII string: each of the 21 bytes is interpreted as a single ASCII character.
Only values between <code>20<sub>16</sub></code> and <code>7F<sub>16</sub></code> are allowed in this string, meaning that the special ASCII characters are not supported.

## Map mode

This byte can be represented as bits: <code>001F0HLh<sub>2</sub></code>

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

| Value                             | Map Mode      |
| --------------------------------- | ------------- |
| <code>00100000<sub>2</sub></code> | LoROM         |
| <code>00100001<sub>2</sub></code> | HiROM         |
| <code>00100010<sub>2</sub></code> | ExLoROM       |
| <code>00100100<sub>2</sub></code> | ExHiROM       |
| <code>00110000<sub>2</sub></code> | Fast LoROM    |
| <code>00110001<sub>2</sub></code> | Fast HiROM    |
| <code>00110010<sub>2</sub></code> | Fast ExLoROM  |
| <code>00110100<sub>2</sub></code> | Fast ExHiROM  |

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

| Values of `X`               | Expansion chip |
| --------------------------- | -------------- |
| <code>0<sub>16</sub></code> | DSP            |
| <code>1<sub>16</sub></code> | SuperFX        |
| <code>2<sub>16</sub></code> | OBC-1          |
| <code>3<sub>16</sub></code> | SA1            |
| <code>4<sub>16</sub></code> | S-DD1          |
| <code>5<sub>16</sub></code> | S-RTC          |
| <code>E<sub>16</sub></code> | Other          |
| <code>F<sub>16</sub></code> | Custom         |

## ROM size

The actual size of the ROM equals <code>2<sup>n</sup></code>KiB, where `n` is the value of this byte.

## SRAM size

If this byte equals `0`, the size of SRAM is also 0.
Otherwise, the actual size of SRAM equals <code>2<sup>n</sup></code>KiB, where `n` is the value of this byte.

## Region/destination code

Just an enumeration of numbers corresponding to certain regions.

| Value                        | Region                  |
| ---------------------------- | ----------------------- |
| <code>00<sub>16</sub></code> | Japan                   |
| <code>01<sub>16</sub></code> | Most of North America   |
| <code>02<sub>16</sub></code> | Most of Europe          |
| <code>03<sub>16</sub></code> | Sweden                  |
| <code>04<sub>16</sub></code> | Finland                 |
| <code>05<sub>16</sub></code> | Denmark                 |
| <code>06<sub>16</sub></code> | France                  |
| <code>07<sub>16</sub></code> | The Netherlands         |
| <code>08<sub>16</sub></code> | Spain                   |
| <code>09<sub>16</sub></code> | Germany                 |
| <code>0A<sub>16</sub></code> | Italy                   |
| <code>0B<sub>16</sub></code> | China                   |
| <code>0C<sub>16</sub></code> | Indonesia               |
| <code>0D<sub>16</sub></code> | South Korea             |
| <code>0E<sub>16</sub></code> | Global                  |
| <code>0F<sub>16</sub></code> | Canada                  |
| <code>10<sub>16</sub></code> | Brazil                  |
| <code>11<sub>16</sub></code> | Australia               |
| <code>12<sub>16</sub></code> | Other (1)               |
| <code>13<sub>16</sub></code> | Other (2)               |
| <code>14<sub>16</sub></code> | Other (3)               |

## Developer ID

A number assigned to each developer.

Nintendo's ID is `1`.

## Version number

This byte is the minor version number: the major version number is always assumed to be `1`.
For example, if this byte is `0`, the full version number is `1.0`.

## Checksum & complement check

The checksum is calculated by adding the values of every single byte in the ROM and keeping the 2 least significant bytes of the result.
Because the checksum itself is also used in this calculation, it comes together with a complement check equal to `¬checksum`, ensuring that they always add up to the same value:

```
checksum + complement = 1FE₁₆
```

Checksum and complement check are used by SNES for validating the ROM.
Bitwise XOR of the checksum and its complement check is always equal to <code>FFFF<sub>16</sub></code>.

This can be used to find the location of the IRH and deduce the mapping mode from that:
- If <code>readw(£007FDC) ^ readw(£007FDE) = FFFF<sub>16</sub></code>, it means that the IRH is at `£007FC0` and we're in LoROM.
- If <code>readw(£00FFDC) ^ readw(£00FFDE) = FFFF<sub>16</sub></code>, it means that the IRH is at `£00FFC0` and we're in HiROM.

# Acknowledgments

Information in the [Data format](/docs/smw-rom/internal-rom-header/#data-format) section is based on this video by Retro Game Mechanics Explained.
Thank you!

{% include video id="RAa561BoWwA" provider="youtube" %}
