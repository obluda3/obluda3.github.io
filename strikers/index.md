---
layout: default
---

# Strikers 2013 Documentation

This page was made in order to centralize all the knowledge that has been acquired through reverse engineering of the game's files / code.

You can read it in whatever order but I'd recommend starting with **Filesystem**.

Special thanks to [Alpha](https://www.youtube.com/@Jhawk_029), [onepiecefreak](https://github.com/onepiecefreak3) and [Rosetta](https://twitter.com/fatalblooms92) for their help and work on the game.

Here's a useful document: [Filemap](https://docs.google.com/spreadsheets/d/1Ze-WjHwCtC_ie8IV7JgQagF9QfnqWJfbxfC_Dy8u2eQ/)

- [Filesystem](#filesystem)
	- [Xtreme's filesystem](#xtreme-filesystem)
	- [Editing the game's archives](#archive-editing)
- [Common file formats](#common-file-formats)
	- [BLN](#bln)
	- [Bin Archives](#bin-archives)
	- [Nintendo formats](#nintendo-formats)
	- [SHTX](#shtx)
	- [TexCut](#texcut)
	- [SSAD](#ssad)
	- [Data Cache](#data-cache)
- [ShadeLZ Compression](#shadelz-compression)
- [Players](#players)
	- [Keys](#keys)
	- [Player copies](#player-copies)
	- [Charge Data](#charge-data)
	- [PLAYER_DEF](#player_def)
	- Body models
- Special Moves
	- Definition
	- Special Info
	- File Info
	- Animations
- 3D Models
- [Miximaxes](#miximaxes)
- Teams
	- Emblems
	- Uniforms
	- File Info
	- Team Definition
- Armed
- Unlocks
- Savedata
- RNG
- Audio files
	- Music
	- Sound clips
- Font files
- Text files
- [Code](#code)
	- PWORK
	- Move Power
	- Loading files
	- UtilitySato
	- Button Checks
	- [cTASK](#ctask)
	- Settings menu
	- stNameWindow
	- CSprStudio
	- cPopup
	- Rendering text
	- VWF
	- Rendering textures

# Filesystem

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


**mcb0.bln** and **mcb1.bln** are the two parts of the [BLN](#BLN) format, the main archive used in the game. It stores the same files that are contained in the *bin* archives. The mcb0 is used to split the mcb1 in various "folders" **(BLN Sub)**, containing multiple files that could be used in the same context. That way the game is able to load multiple files at once into memory at the same location. It also means that the mcb contains many duplicates of the same files since each one could be used in different contexts. 

Each file stores a reference to its corresponding file in the *bin* archives through the *Archive Index* and *Archive Offset* parameters. 

The files within the game's archives don't have any name. They are usually described through their *index* within the archives. The following notation will be used in this document: `10 000 × Archive Index + File Index`. For example `40023` would be the file with index `23` within `dat.bin`. This is also how the game accesses them.


## Archive Editing

This has actually nothing to do in a documentation, but I will explain it regardless.

The first way to modify archives was using Kuriimu2. This tool allows one to edit files within the MCB and its subfiles. However it had two downsides: 
- First, it was only able to replace a single file in its BLN Sub, not its copies in the rest of the MCB (nor its original archive).
- Second, it wasn't able to replace a file with another one that's bigger in size than the original. This was a direct consequence of the first point.

This led to the creation of Strikers2013-Tools for the purposes of the various fantranslation projects.

Its usage is pretty simple, in the **Archive** section, select the archive you wish to extract and press the extract button.

If you wish to import files, make a folder with all of the files that you want to import into your desired archive. Name them accordingly, with their file index. If your files are decompressed, they need to have the `.dec` extension. 

## Xtreme Filesystem

The need for a way to distribute mods without having to provide an entire mcb+dat/ui/grp/scn/scn_sh build of multiple gigabytes has led me to develop a new way to load files. 

In Xtreme, it's possible to replace files without having to actually modify the archive files. The `Modified` folder in the game's root acts as a replacement for the filesystem of some files. It contains various subfolders : `grp`, `scn`, `ui`, `scn` and `scn_sh`. The files inside of those folders are expected to be numbered from 0 to 10000. 

The idea is that when the game tries to load a file, Xtreme checks if it's already present inside of one of those subfolders. If it is, it loads the modified version instead.

For example, if one wishes to replace the file "81.bin of dat.bin" (40081), they would have to copy the modified (and already compressed version of it) to `Modified\dat\81.bin`. 

All of that is handled inside of `fileloader.cpp` in Xtreme's source code. The source file is pretty long but the actual code is short (and quite easy to understand I would think). The only problem is that we need to patch a lot of different functions inside of the game's code, which are all handled differently. 

There's also a way to *redirect* files. It's a special feature we invented. It means that when the game tries to load a specific file (let's say 32567), we can force it to load another one instead (let's say 40356). This will be done regardless of if one of them is modified inside of the Modified folder.

# Common file formats

## BLN

As explained above, MCB0 is a table for the data stored in MCB1.

For some reason, the data is stored in little endian.

### MCB0

The following table describes the structure of a single entry within the file. There's no header to the archive, but the game stops reading it once it reaches a null entry.

| Offset | Name | DataType |
| --- | --- | --- |
| 0x0 | ID | ``u32`` |
| 0x4 | Offset | `u32` | 
| 0x8 | Size | `u32` |

### MCB1

The following table describes the structure of a single file entry. The files within the archive are aligned to the nearest `0x800` multiple. The end of a BLN Sub is marked by the bytes `FF 7F 00 00`.

| Offset | Name | DataType |
| --- | --- | --- |
| 0x0 | Archive index | `u32` |
| 0x4 | Archive offset | `u32` |
| 0x8 | Size | `u32` |
| 0xC | File Data | `u8[Size]` |

## Bin Archives

This format is used for the other main archives in the game. 

The header is as follows:

| Offset | Name | DataType |
| --- | --- | --- |
| 0x0 | File Count | `u32` |
| 0x4 | Pad Factor | `u32` |
| 0x8 | Mul Factor | `u32` |
| 0xC | Shift Factor | `u32` |
| 0x10 | Mask | `u32` |

Mul Factor, Pad Factor, Shift Factor and Mask are constants used in the calculation of each file's offset and size. The file table is stored right after that, it consists of a simple array of ``u32``s that we could call `OffSize`s. `OffSize` contains both the offset and the size of a single file. They can be extracted using the following formulas:

```python
Offset = (OffSize >> ShiftFactor) * PadFactor
Size = (OffSize & Mask) * Mul Factor
```

Each file is compressed using [ShadeLZ](), and its header-less variant for those in **dat.bin**.
## Nintendo formats

Some of the files in the game are stored in the regular formats used on the Wii. They include:

- [BRRES]() used for 3d models, animations and textures.
- [U8]() or `.arc` archives, used mostly for the 2D animated move names graphics

## SHTX

**SHTX** (short for **SH**ade **T**e**X**ture) is the main texture file formats used in the game. Each file could store a texture in one of 3 different formats:
- 32bpp, raw pixel data
- 8bpp, with a 256-color palette.
- 4bpp, with a 16-color palette.

A file's structure is as follows:

| Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x0 | Magic | `u32` | |
| 0x4 | Format | `u16` | Gives the bits per pixel, FF=>32bpp; FS; 8bpp; F4=>4bpp |
| 0x6 | Width | `u16` | |
| 0x8 | Height | `u16` | |
| 0xA | Unk A | `u16` | |
| 0xB | Unk B | `u16` | |
| 0xC | Palette | `u32[256]` | Doesn't appear for 32bpp images |
| 0xC+Palette | Texture Data | `u8[bpp × Width × Height]` | |

## TexCut

This format has no particular *file magic*. The name has been given after the game's symbols that call it an array of `SHD_TEXCUT`. It's used to define rectangular "sections" of a texture that needs to be split. For some reason, the data is stored in little endian

A `SHD_TEXCUT` has the following structure:

| Offset | Name | DataType |
| --- | --- | --- |
| 0x0 | X Offset | `u16` |
| 0x2 | Y Offset | `u16` |
| 0x4 | Width | `u16` |
| 0x6 | Height | `u16` |

## SSAD

**SSAD** (likely short for **S**hade **S**prite **A**nimation **D**ata) is used to define layouts and sprite animations. Its data is stored in little endian.

A SSAD is made of different *entries* (elements in the layout), that hold various *fields* (parameters) to describe the way it will look on screen. Some of the fields can even be keyframed, thus handling the *animation* part of the format. Most of the format hasn't been figured out yet.

The file has the following header (real length: 0x1C):

| Offset | Name | DataType |
| --- | --- | --- |
| 0x0 | Magic | `u32` |
| 0x14 | Entry count | `u32` |

Each field has the following header:

| Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x0 | Field Name | `u32` | |
| 0x4 | Length | `u32` | |
| 0x8 | Success Values | `u32[2]` | Only present if the field allows keyframes. The name of the fields comes from the symbols, their use is not known |
| 0x10 | Keyframe Count | `u32` | Only present if the field allows keyframes. |
| 0x14 | Keyframes | `Keyframe[Keyframe Count]` | Only present if the field allows keyframes. The Keyframe structure is 0x1C bytes long |

Here are the different fields:

| Field Name | Description | Has KeyFrame |
| --- | --- | --- |
| PART | | |
| NAME | Name of the entry | | 
| AREA | 4 `u32`s => rectangle (x1,y1,x2,y2) | |
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

## Data Cache

Again, its name comes from guessing through the name of the `DCLoad` function used to load them. It's used in *dat.bin* to serialize various data structures, in an effecient (similar to how [Flatbuffers](https://flatbuffers.dev/) work). The idea is that any parameter that is an offset within the file is replaced, when the file is loaded in memory, with a pointer to the actual address it targeted. This is done by simply adding the address of the beginning of the file in memory to the offset.

A file could contain multiple *sections*, each one being an array for a specific structure. Some sections can contain text, others contain more complicated structures.

Header:

| Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x0 | Section count | `u32` | |
| 0x4 | Pointer Fix List | `u32` | Offset |
| 0x8 | Beginning | `u32` | Points to the start of the data |

Then, for each section:

| Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x0 | Section Offset | `u32` | |
| 0x4 | Structure Count | `u32` | Size of the array |

The *pointer fix list* is a section containing the offsets of various `u32`s within the file that need to be treated as pointers. Basically it's a list of all the offsets of pointers within the file.

# ShadeLZ Compression

All the files in the game's archives are compressed using a mixed LZ-RLE algorithm that's used in multiple Shade games. A compressed file may include this header:

| Offset | Name | DataType | --- |
| --- | --- | --- | --- |
| 0x0 | Magic | `u32` | Always `FC AA 55 A7` |
| 0x4 | Compressed Size | `u32` | |
| 0x8 | Decompressed Size | `u32` | |

Here's a rough translation of the algorithm:

```python
b = data[i]
if b & 0x80:
	n = (b & 0x60) >> 5 + 4
	offset = data[i+1] + ((b & 0x1F) << 8)
	# copy n bytes starting from position i-offset to the output

else if b & 0x60:
	n = b & 0x1F
	# copy n bytes starting from position i-offset (same offset as previous case) to the output

else if b & 0x40:
	if b & 0x10:
		n = (b & 0xF) << 8 + data[i+1]
		d = data[i+2]
	else:
		n = b & 0xF
		d = data[i+1]
	# copy byte d n times to the output

else:
	if b & 0x20:
		n = data[i+1] + ((b & 0x1F) << 8)
	else:
		n = b & 0x1F
	# copy n raw bytes starting from position i+1 or i+2
```

# Players

In the next section we'll focus on the various data structures used to store data related to the game's various players.

First thing you should know is that there are `0x19B` players in that game. Some of them are not used at all, this includes people like Shinoyama, or Saru's miximaxed form. Others are present but not playable, like the managers or coaches.

## Keys

File: 40011, **7** sections but only the last two sections are known.

### Section 6

This section defines all of the different Key Profiles that exist in the game. First of all, each key can give a specific `StatBonus` to each stat.

Here's the `StatBonus` struct:

| Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x0 | Unknown | `u16` | Probably some padding |
| 0x2 | Level 1 | `u16` | Stat increase that you get with 50% kizuna |
| 0x4 | Level 2 | `u16` | Stat increase that you get with 75% kizuna |
| 0x6 | Level 3 | `u16` | Stat increase that you get with 100% kizuna |

Knowing that, it's pretty easy to deduce what the `KeyProfile` struct looks like.

| Offset | Name | DataType |
| --- | --- | --- |
| 0x0 | Kick | `StatBonus` |
| 0x8 | Catch | `StatBonus` |
| 0x10 | Body | `StatBonus` |
| 0x18 | Guard | `StatBonus` |
| 0x20 | Control | `StatBonus` |
| 0x28 | Speed | `StatBonus` |

### Section 7

This section is just a `u8[0x19C]` table. Each one of its values corresponds to the key profile of a specific player, the index of the table being the player id.

## Player Copies

File: 40017, 3 sections that are basically copies of eachother.

This file is used to link the different versions of a player (for example, Tachimukai 2 and Japan) to their common `ListID`. 

It's a table of the following structure:

| Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x0 | ListID | `u32` | An ID that's shared between all versions |
| 0x1 | Player IDs | `u32[5]` | |

Fun fact: Kidou has more than 5 different versions, his case is handled separately in the game's code.

## Charge Data

File: 40015, 1st section

This file is used to store the various gauge increments that happen when you perform specific actions. The gauge is one of the most important pieces of gameplay in that game, it's refilled automatically for all players, but can be refilled through specific action such as passes, tackling etc. They correspond to what the competitive community call **charge profiles**. The gauge can be filled until it reaches 200, will stay full for 5 in-game minutes (unless the player is holding the ball).

Each player has a specific *gauge profile*, and each gauge profile gives different bonuses. Here are the different gauge profiles with their community names.

| Index | Name |
| --- | --- |
| | Boy GK Medium |
| | Boy GK Fast |
| | Boy GK Slow |
| | Boy DF Medium |
| | Boy DF Fast |
| | Boy DF Slow |
| | Boy MF Medium |
| | Boy MF Fast |
| | Boy MF Slow |
| | Boy FW Medium |
| | Boy FW Fast |
| | Boy FW Slow |
| | Girl GK Medium (UNUSED) |
| | Girl GK Fast |
| | Girl GK Slow (UNUSED) |
| | Girl DF Medium |
| | Girl DF Fast |
| | Girl DF Slow (UNUSED) |
| | Girl MF Medium |
| | Girl MF Fast |
| | Girl MF Slow (UNUSED) |
| | Girl FW Medium |
| | Girl FW Fast |
| | Girl FW Slow (UNUSED) |
| | Keshin User GK Medium (UNUSED) |
| | Keshin User GK Fast (UNUSED) |
| | Keshin User GK Slow |
| | Keshin User DF Medium |
| | Keshin User DF Fast (UNUSED) |
| | Keshin User DF Slow |
| | Keshin User MF Medium |
| | Keshin User MF Fast (UNUSED) |
| | Keshin User MF Slow |
| | Keshin User FW Medium |
| | Keshin User FW Fast (UNUSED) |
| | Keshin User FW Slow |
| | Mixi-Max (UNUSED) |

This file just contains an array of the following structure:

| Offset | Name |
| --- | --- |
| 0x0 | Idle |
| 0x4 | Holding the ball |
| 0x8 | Pass |
| 0xC | Normal Shoot |
| 0x10 | Normal Catch |
| 0x14 | Goal |
| 0x18 | Received goal |
| 0x1C | Tackle |
| 0x24 | Tackle (on an opponent) |
| 0x28 | Tackle (on an opponent) |
| 0x40 | Tactical Action (on an opponent) |
| 0x44 | Tactical Action |
| 0x48 | Tactical Action (on an opponent?) |
| 0x4C | Through pass |
| 0x50 | Direct shot |
| 0x54 | Cross |
| 0x58 | Volley |

## PLAYER_DEF

File: 40015, 2nd section

Arguably the second most important structure. It's 0x148 bytes long.

| Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x0 | Hex ID | u32 | Player's HEX ID |
| 0x8 | Hidden name | u32 | Index into an entry of the text file |
| 0xC | Short Name ID | u32 | Index into an entry of the text file |
| 0x10 | Full Name ID | u32 | Index into an entry of the text file |
| 0x14 | Player Name | string |  |
| 0x2c | Gender | u32 | 0 = Male 1 = Female 2 = Other |
| 0x30 | Idle Animation | u32 | Used in caravan and minigame selection |
| 0x38 | Description | u32 | Index into an entry of the text file |
| 0x3c | Bodytype | u32 | 0 = Man 1 = Large 2 = Chibi 3 = Muscle 4 = Girl1 5 = Girl2 |
| 0x40 | Height | u32 | Player height specification |
| 0x44 | Shadow Size | u32 | |
| 0x48 | Tactical Action | u32 | 0x14 = Feint 0x15 = Roll 0x16 = Short 0x17 = Jump 0x18 = White Sprint 0x19 = Red Sprint 0x1A = Girl |
| 0x4C | Course Animation | u32 | | 
| 0x50 | Team | u32 | |
| 0x54 | Emblem | u32 | |
| 0x58 | Team Portrait ID | u32 | Portrait index in the team list |
| 0x5C | Position | u32 | GK = 0; DF = 0x23; MF = 0x24; FW = 0x25 |
| 0x60 | Face Model | u32 | Player's 3D Model (in match) |
| 0x64 | Face Model | u32 | Player's 3D Model |
| 0x68 | Face Model | u32 | Player's 3D Model |
| 0x6C | Body Model | u32 | Player's Body Model (reserved - replaced in game) |
| 0x70 | Body Model | u32 | Player's Body Model (reserved - replaced in game) |
| 0x78 | Portrait | u32 | Player's 2D Portrait |
| 0x80 | Left Match Portrait | u32 | 2D Portrait in Match, left side |
| 0x84 | Right Match Portrait | u32 | 2D Portrait in Match, right side |
| 0x88 | Neck and legs skin color | u32 | xRGB |
| 0x8C | Arms and knees color | u32 | xRGB |
| 0xF4 | Element | u32 | 0 = Wind 1 = Wood 2 = Fire 3 = Earth 4 = Void |
| 0xF8 | Charge profile | u32 | Player's charge profile |
| 0x104 | Voice | u32 |  |
| 0x108 | Original ID | u32 | ID of the "main" player for those that have multiple versions of themselves |
| 0x110 | Price | s16 | A value above 0 enables the player, a value of -1 makes the player unlocked by default |
| 0x112 | List position | u16 | |
| 0x114 | List position | u32 | |


# Miximaxes

One of the main additional mechanics in Strikers 2013. Funnily enough, it's only handled in the game's code, not in any files. It can only be performed if the player has a miximax unlocked in its stats.

Once a miximax is performed, multiple things happen to the player doing it: their face model, face portrait, moveset, idle animation, size and tactical action change to match those of their aura. Note that the body model isn't affected, meaning that in vanilla Strikers, a miximax cannot turn a normal-sized player to a large size player.

In miximax state, the game switch the regular gauge to one that cannot be refiled, but where you can use any move at any time. This causes one major bug, commonly called the **mixibug**. The details are not known as to why it happens, but basically it comes from an error with the way the game calculates the gauge reduction after performing a move. 

Usually, once the miximax gauge gets to 0, the game releases the player's miximax. Because of the contest mechanic, there are cases where a miximaxed player would try to perform a move, but they would be stopped by their opponent. In those cases, the game can expect the miximaxed player's gauge to reach 0, even if it won't. It would then try to release the miximax while it should still be present, which causes the mixibug.

As to how miximaxes are stored, it's fairly simple. The actual structure is stored at address `0x804c8b90`. It looks like this:

| Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x0 | Source Player | u16 | |
| 0x2 | Slot | u16 | Used for players which have multiple miximaxes |
| 0x4 | Aura | u16 | |
| 0x6 | Move 1 | u16 | This is the move that's performed when you transform |
| 0x8 | Move 2 | u16 | |
| 0xA | Move 3 | u16 | |

# Code

In the next section, you'll find a description of some parts of the game's code and data structures that I have reversed.

## cTask

The `cTask` is a class that's used everywhere in the game. It's used to manage menus, and various parts of the game that needs to be updated following a **task** system. The **cTask** contains a stack of tasks, and would always perform the task that's located at the top of its stack. 

A task can have different behavior depending on if it's on its first call, a random call or its last call. A task is almost always associated with a parent class, so that when it's called, the task can have access to the parent structure. It can also be called with an integer argument, usually to describe some kind of state.

Later in the documentation you'll find some examples of parts of the game that use this class.

The following methods are available:

| Name | Description |
| --- | --- | 
| Update | Executes the task that is at the top of the stack with 2 as its argument. If it has never been executed, it first executes it with 0 as the argument. Updates the call count, and elapsed time. |
| Push | Push the task that's passed as an argument to the top of the stack |
| Pop | Terminate the task that's at the top of the stack, and remove it from the stack |


Task structure:

| Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x0 | Function | u32 | Pointer to the function |
| 0x4 | Parent class | u32 | Pointer to the parent class mentioned earlier |
| 0x8 | unk | u32 | |

cTask structure:

 Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x0 | Tasks | Task* | Pointer to an array of tasks. |
| 0x4 | Elapsed time | u32 | not sure about this one |
| 0x8 | Call count | u32 | Counts how many times the current function has been executed. |
| 0x4 | Remaining tasks | u32 | Counts how many tasks are free |
| 0x8 | Task Count | u32 | |
## Charge Data

File: 40015, 1st section

This file is used to store the various gauge increments that happen when you perform specific actions. The gauge is one of the most important pieces of gameplay in that game, it's refilled automatically for all players, but can be refilled through specific action such as passes, tackling etc.

Each player has a specific *gauge profile*, and each gauge profile gives different bonuses. Here are the different gauge profiles with their community names.

| Index | Name |
| --- | --- |
| | Boy GK Medium |
| | Boy GK Fast |
| | Boy GK Slow |
| | Boy DF Medium |
| | Boy DF Fast |
| | Boy DF Slow |
| | Boy MF Medium |
| | Boy MF Fast |
| | Boy MF Slow |
| | Boy FW Medium |
| | Boy FW Fast |
| | Boy FW Slow |
| | Girl GK Medium (UNUSED) |
| | Girl GK Fast |
| | Girl GK Slow (UNUSED) |
| | Girl DF Medium |
| | Girl DF Fast |
| | Girl DF Slow (UNUSED) |
| | Girl MF Medium |
| | Girl MF Fast |
| | Girl MF Slow (UNUSED) |
| | Girl FW Medium |
| | Girl FW Fast |
| | Girl FW Slow (UNUSED) |
| | Keshin User GK Medium (UNUSED) |
| | Keshin User GK Fast (UNUSED) |
| | Keshin User GK Slow |
| | Keshin User DF Medium |
| | Keshin User DF Fast (UNUSED) |
| | Keshin User DF Slow |
| | Keshin User MF Medium |
| | Keshin User MF Fast (UNUSED) |
| | Keshin User MF Slow |
| | Keshin User FW Medium |
| | Keshin User FW Fast (UNUSED) |
| | Keshin User FW Slow |
| | Mixi-Max (UNUSED) |

This file just contains an array of the following structure:

| Offset | Name |
| --- | --- |
| 0x0 | Idle |
| 0x4 | Holding the ball |
| 0x8 | Pass |
| 0xC | Normal Shoot |
| 0x10 | Normal Catch |
| 0x14 | Goal |
| 0x18 | Received goal |
| 0x1C | Tackle |
| 0x24 | Tackle (on an opponent) |
| 0x28 | Tackle (on an opponent) |
| 0x40 | Tactical Action (on an opponent) |
| 0x44 | Tactical Action |
| 0x48 | Tactical Action (on an opponent?) |
| 0x4C | Through pass |
| 0x50 | Direct shot |
| 0x54 | Cross |
| 0x58 | Volley |

## PLAYER_DEF

File: 40015, 2nd section

Arguably the second most important structure. It's 0x148 bytes long.

| Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x0 | Hex ID | u32 | Player's HEX ID |
| 0x8 | Hidden name | u32 | Index into an entry of the text file |
| 0xC | Short Name ID | u32 | Index into an entry of the text file |
| 0x10 | Full Name ID | u32 | Index into an entry of the text file |
| 0x14 | Player Name | string |  |
| 0x2c | Gender | u32 | 0 = Male 1 = Female 2 = Other |
| 0x30 | Idle Animation | u32 | Used in caravan and minigame selection |
| 0x38 | Description | u32 | Index into an entry of the text file |
| 0x3c | Bodytype | u32 | 0 = Man 1 = Large 2 = Chibi 3 = Muscle 4 = Girl1 5 = Girl2 |
| 0x40 | Height | u32 | Player height specification |
| 0x44 | Shadow Size | u32 | |
| 0x48 | Tactical Action | u32 | 0x14 = Feint 0x15 = Roll 0x16 = Short 0x17 = Jump 0x18 = White Sprint 0x19 = Red Sprint 0x1A = Girl |
| 0x4C | Course Animation | u32 | 1 for males to have Kappa's animation | 
| 0x50 | Team | u32 | Player's team |
| 0x54 | Emblem | u32 | Player's emblem |
| 0x58 | Team Portrait ID | u32 | Portrait in the team list |
| 0x5C | Position | u32 | GK = 0 DF = 0x23 MF = 0x24 FW = 0x25 |
| 0x60 | Face Model | u32 | Player's 3D Model (in match) |
| 0x64 | Face Model | u32 | Player's 3D Model (index u32o grp.bin minus 800) |
| 0x68 | Face Model | u32 | Player's 3D Model (index u32o grp.bin minus 800) |
| 0x6C | Body Model | u32 | Player's Body Model (reserved - replaced in game) |
| 0x70 | Body Model | u32 | Player's Body Model (reserved - replaced in game) |
| 0x78 | Portrait | u32 | Player's 2D Portrait |
| 0x80 | Left Match Portrait | u32 | 2D Portrait in Match, left side |
| 0x84 | Right Match Portrait | u32 | 2D Portrait in Match, right side |
| 0x88 | Neck and legs skin color | u32 | xRGB |
| 0x8C | Arms and knees color | u32 | xRGB |
| 0xF4 | Element | u32 | 0 = Wind 1 = Wood 2 = Fire 3 = Earth 4 = Void |
| 0xF8 | Charge profile | u32 | Player's charge profile (see above) |
| 0x104 | Voice | u32 |  |
| 0x110 | Price | s16 | A value above 0 enables the player, a value of -1 makes the player unlocked by default |
| 0x112 | List position | u16 | |
| 0x114 | List position | u32 | |

In order to make a player appear in the list, give them a valid Team/Emblem, and a valid list position. If two players have the same list position and team, only the first one (based on their ID) will appear.

The emblem also controls the player's uniform. Some fields are replaced at runtime, such as the bodymodel. The voice profile only controls 