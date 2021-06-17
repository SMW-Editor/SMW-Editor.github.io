---
permalink: /docs/smw-rom/compression-methods/
title: "Compression Methods"
toc: true
sidebar:
    nav: "smw_rom_docs"
---

Some data within the Super Mario World ROM is compressed using different algorithms.
This page explains how to decompress that data depending on the algorithm.

# LC_RLE1

The first byte is a command in the format <code>FLLLLLLL<sub>2</sub></code>, where `F` stands for the RLE flag, and `LLLLLLL` stands for `length`.
Depending on the value of the RLE flag, the bytes that follow the command are interpreted either way:

| RLE flag | Behaviour                                        |
| -------- | ------------------------------------------------ |
| `0`      | The following `length` bytes are read directly.  |
| `1`      | The following 1 byte is repeated `length` times. |

After that we get another command, another string, rinse and repeat until we read command <code>FF<sub>16</sub></code> followed by another <code>FF<sub>16</sub></code>, meaning that the data ends there.

Our implementation of the LC_RLE1 decompressor can be found <a href="https://github.com/SMW-Editor/smw-editor/blob/parse-level/smwe_rom/src/compression/lc_rle1.rs" target="_blank">here</a>.

# LC_LZ2

The first byte is a header in the format <code>CCCLLLLL<sub>2</sub></code>, where `CCC` stands for the Command, and `LLLLL` stands for `length`.
Depending on the command, the bytes that follow the header are interpreted in a variety of ways:

| Command                                            | Behaviour                                                                                                      |
| -------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| <code>000<sub>2</sub></code> (Direct&nbsp;Copy)    | The following `length + 1` bytes are read directly.                                                            |
| <code>001<sub>2</sub></code> (Byte&nbsp;Fill)      | The following 1 byte is repeated `length + 1` times.                                                           |
| <code>010<sub>2</sub></code> (Word&nbsp;Fill)      | The following 2 bytes are alternated until `length + 1` bytes are written.                                     |
| <code>011<sub>2</sub></code> (Incresing&nbsp;Fill) | The following 1 byte is repeated `length + 1` times, but after each write the value is incremented by 1.       |
| <code>100<sub>2</sub></code> (Repeat)              | The following 1 word is interpreted as an index to the output buffer from which `length + 1` bytes are copied. |
| <code>111<sub>2</sub></code> (Long&nbsp;Length)    | This extends the header to 2 bytes and changes its format to<br><code>111CCCLL LLLLLLLL<sub>2</sub></code>.    |

After that we get another command, another string, rinse and repeat until we read header <code>FF<sub>16</sub></code> which means that the data ends there.

Our implementation of the LC_LZ2 decompressor can be found <a href="https://github.com/SMW-Editor/smw-editor/blob/parse-level/smwe_rom/src/compression/lc_lz2.rs" target="_blank">here</a>.
