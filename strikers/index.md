---
layout: default
---

# Strikers 2013 Documentation

This page was made in order to centralize all the knowledge that has been acquired through reverse engineering of the game's files / code.

You can read it in whatever order but I'd recommend starting with **Filesystem**.

Special thanks to [Alpha](), [onepiecefreak]() and [Rosetta]() for their help and work on the game.

- [Filesystem](#filesystem)
- [Common file formats](#common-file-formats)
	- [BLN](#bln)
	- [Bin Archives] (#bin-archives)
- ShadeLZ Compression
- Editing the game's archives
- Players
	- Keys
	- Player copies
	- Charge Data
	- PLAYER_DEF
	- Body models
- Special Moves
	- Definition
	- Special Info
	- File Info
	- Animations
- Miximaxes
- Teams
	- Emblems
	- Uniforms
	- File Info
	- Team Definition
- Armed
- Unlocks
- Savedata
- Code
	- Move Power
	- Loading files
	- UtilitySato
	- Button Checks
	- cTASK
	- stNameWindow
	- CSprStudio
	- cPopup

## Filesystem

At the root of the game, you'd run across various type of files.

**InazumaWii.dlf**, **InazumaWii.elf** are compiler-generated files that were left on the disc. The elf isn't stripped and contains all symbols of the game's functions.

The **movies** folder contains `MOC5` files. This is a video codec that was developped by **Mobiclip**.

**InazmaWii.brsar**, and the **stream** folder are responsible for managing the game's audio files. They can be edited with tools such as [Brawlcrate]()

**ui.bin**, **scn.bin**, **scn_sh.bin**, **grp.bin**, **dat.bin**, respectively contain UI elements, special moves and 3D scenes, special shoots scenes, various 3d models, other data. They can also be named with their *archive index*, all in the following table: 

| Archive Name | Archive Index |
| --- | --- |
| grp | 0 |
| scn | 1 |
| scn_sh | 2 |
| ui | 3 |
| dat | 4 |


**mcb0.bln** and **mcb1.bln** are the two parts of the [BLN](#BLN) format, the main archive used in the game. 

The files within the game's archives don't have any name. They are usually described through their *index* within the archives. The following notation will be used in this document: `10 000 × Archive Index + File Index`. For example `40023` would be the file with index `23` within `dat.bin`. This is also how the game accesses them.

## Common file formats

### BLN

It stores the same files that are contained in the *bin* archives. The mcb0 is used to split the mcb1 in various "folders" **(BLN Sub)**, containing multiple files that could be used in the same context. That way the game is able to load multiple files at once into memory at the same location. It also means that the mcb contains many duplicates of the same files since they could be used in different situations. 

Each file stores a reference to its corresponding file in the *bin* archives through the *Archive Index* and *Archive Offset* parameters. 

#### MCB0

The following table describes the structure of a single entry within the file. There's no header to the archive, but the game stops reading it once it reaches a null entry.

| Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x0 | ID | u32 | |
| 0x4 | Offset | u32 |  |
| 0x8 | Size | u32 | |

#### MCB1

The following table describes the structure of a single file entry. The files within the archive are aligned to the nearest `0x800` multiple. The end of a BLN Sub is marked by the bytes `FF 7F 00 00`.

| Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x0 | Archive index | u32 | |
| 0x4 | Archive offset | u32 |  |
| 0x8 | Size | u32 | |
| 0xC | File Data | u8[Size] | |

### Bin Archives

This format is used for the other main archives in the game. 

The header is as follows:

| Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x0 | File Count | u32 | |
| 0x4 | Pad Factor | u32 | |
| 0x8 | Mul Factor | u32 | |
| 0xC | Shift Factor | u32 | |
| 0x10 | Mask | u32 | |

Mul Factor, Pad Factor, Shift Factor and Mask are constants used in the calculation of each file's offset and size. The file table is stored right after that, it consists of a simple array of `u32`s that we could call `OffSize`s. `OffSize` contains both the offset and the size of a single file. They can be extracted using the following formulas:

```python
Offset = (OffSize >> ShiftFactor) * PadFactor
Size = (OffSize & Mask) * Mul Factor
```

Each file is compressed using [ShadeLZ], and its header-less variant for those in **dat.bin**
