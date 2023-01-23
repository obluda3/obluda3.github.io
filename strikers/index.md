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
	- [Bin Archives](#bin-archives)
	- [Nintendo formats](#nintendo-formats)
	- [SHTX](#shtx)
	- [TexCut](#texcut)
	- [SSAD](#ssad)
	- [Data Cache](#data-cache)
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
- 3D Models
- Miximaxes
- Teams
	- Emblems
	- Uniforms
	- File Info
	- Team Definition
- Armed
- Unlocks
- Savedata
- Code bits
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

For some reason, the data is stored in little endian.

#### MCB0

The following table describes the structure of a single entry within the file. There's no header to the archive, but the game stops reading it once it reaches a null entry.

| Offset | Name | DataType |
| --- | --- | --- |
| 0x0 | ID | u32 |
| 0x4 | Offset | u32 | 
| 0x8 | Size | u32 |

#### MCB1

The following table describes the structure of a single file entry. The files within the archive are aligned to the nearest `0x800` multiple. The end of a BLN Sub is marked by the bytes `FF 7F 00 00`.

| Offset | Name | DataType |
| --- | --- | --- | --- |
| 0x0 | Archive index | u32 |
| 0x4 | Archive offset | u32 |
| 0x8 | Size | u32 |
| 0xC | File Data | u8[Size] |

### Bin Archives

This format is used for the other main archives in the game. 

The header is as follows:

| Offset | Name | DataType |
| --- | --- | --- |
| 0x0 | File Count | u32 |
| 0x4 | Pad Factor | u32 |
| 0x8 | Mul Factor | u32 |
| 0xC | Shift Factor | u32 |
| 0x10 | Mask | u32 |

Mul Factor, Pad Factor, Shift Factor and Mask are constants used in the calculation of each file's offset and size. The file table is stored right after that, it consists of a simple array of `u32`s that we could call `OffSize`s. `OffSize` contains both the offset and the size of a single file. They can be extracted using the following formulas:

```python
Offset = (OffSize >> ShiftFactor) * PadFactor
Size = (OffSize & Mask) * Mul Factor
```

Each file is compressed using [ShadeLZ](), and its header-less variant for those in **dat.bin**.
### Nintendo formats

Some of the files in the game are stored in the regular formats used on the Wii. They include:

- [BRRES]() used for 3d models, animations and textures.
- [U8]() or `.arc` archives, used mostly for the 2D animated move names graphics

### SHTX

**SHTX** (short for **SH**ade **T**e**X**ture) is the main texture file formats used in the game. Each file could store a texture in one of 3 different formats:
- 32bpp, raw pixel data
- 8bpp, with a 256-color palette.
- 4bpp, with a 16-color palette.

A file's structure is as follows:

| Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x0 | Magic | u32 | |
| 0x4 | Format | u16 | Gives the bits per pixel, FF=>32bpp; FS; 8bpp; F4=>4bpp |
| 0x6 | Width | u16 | |
| 0x8 | Height | u16 | |
| 0xA | Unk A | u16 | |
| 0xB | Unk B | u16 | |
| 0xC | Palette | u32[256] | Doesn't appear for 32bpp images |
| 0xC+Palette | Texture Data | u8[bpp × Width × Height] | |

### TexCut

This format has no particular *file magic*. The name has been given after the game's symbols that call it an array of `SHD_TEXCUT`. It's used to define rectangular "sections" of a texture that needs to be split. For some reason, the data is stored in little endian

A `SHD_TEXCUT` has the following structure:

| Offset | Name | DataType |
| --- | --- | --- |
| 0x0 | X Offset | u16 |
| 0x2 | Y Offset | u16 |
| 0x4 | Width | u16 |
| 0x6 | Height | u16 |

### SSAD

**SSAD** (likely short for **S**hade **S**prite **A**nimation **D**ata) is used to define layouts and sprite animations. Its data is stored in little endian.

A SSAD is made of different *entries* (elements in the layout), that hold various *fields* (parameters) to describe the way it will look on screen. Some of the fields can even be keyframed, thus handling the *animation* part of the format. Most of the format hasn't been figured out yet.

The file has the following header (real length: 0x1C):

| Offset | Name | DataType |
| --- | --- | --- |
| 0x0 | Magic | u32 |
| 0x14 | Entry count | u32 |

Each field has the following header:

| Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x0 | Field Name | u32 | |
| 0x4 | Length | u32 | |
| 0x8 | Success Values | u32[2] | Only present if the field allows keyframes. The name of the fields comes from the symbols, their use is not known |
| 0x10 | Keyframe Count | u32 | Only present if the field allows keyframes. |
| 0x14 | Keyframes | Keyframe[Keyframe Count] | Only present if the field allows keyframes. The Keyframe structure is 0x1C bytes long |

Here are the different fields:

| Field Name | Description | Has KeyFrame |
| --- | --- | --- |
| PART | | |
| NAME | Name of the entry | | 
| AREA | 4 u32s => rectangle (x1,y1,x2,y2) | |
| ORGX/ORGY | origin x/y  | |
| TBDT |   | |
| MYID |   | |
| PAID |   | |
| CHID |   | |
| PCID |   | |
| SUCD |   | |
| PRIO |   | |
| POSX | X Position | X |
| POSY | Y Position | X |
| ANGL | Rotation angle | X |
| SCAX | X Scale | X |
| SCAY | Y Scale | X |
| TRAN |   | X | 
| HIDE |   | X | 
| FLPH | Flip H | X |
| FLPV | Flip V | X |
| UDAT |   | X |
| PCOL |   | X | 
| PALT |   | X |
| VERT |   | X |
| IMGX | Image X | X |
| IMGY | Image Y | X |
| IMGW | Image Width | X |
| IMGH | Image Height | X |
| ORFX/ORFY |   | X |

### Data Cache

Again, its name comes from guessing through the name of the `DCLoad` function used to load them. It's used in *dat.bin* to serialize various data structures, in an effecient (similar to how [Flatbuffers]() work). The idea is that any parameter that is an offset within the file is replaced, when the file is loaded in memory, with a pointer to the actual address it targeted. This is done by simply adding the address of the beginning of the file in memory to the offset.

A file could contain multiple *sections*, each one being an array for a specific structure. Some sections can contain text, others contain more complicated structures.

Header:

| Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x0 | Section count | u32 | |
| 0x4 | Pointer Fix List | u32 | Offset |
| 0x8 | Beginning | u32 | Points to the start of the data |

Then, for each section:

| Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x0 | Section Offset | u32 | |
| 0x4 | Structure Count | u32 | Size of the array |

The *pointer fix list* is a section containing the offsets of various u32s within the file that need to be treated as pointers.

