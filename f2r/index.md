---
layout: default
---

# Street Football

Alright, so this one may feel like a joke. It's some dumb shovelware game on the Wii, why did I even do that? 

Short story is that Street Football, or **Foot 2 Rue** as it's called in France is one of my favorite shows from my childhood. And I wanted to look into this game's files to see if there was a chance to get the TV show's original soundtrack.

Sadly enough, this game (which I had never played) has its own soundtrack (which is pretty bad). But I still tried to look into its files to see if I could find anything. The "game" basically consists of a series of cutscenes, which are played back to back with some minigames in between and some dialogues. 

Most of the game's files use Nintendo's proprietary formats, they can be edited using Brawlcrate. The different files are copied with a `_en`, `_fr`, `_it` suffix depending on the language of the console. The way the dialogues and the sound effects appear on screen as you play is managed by some `.csv` files.

Now we'll get to the part I actually re'd. The dialogue files.

`struct` is the main one, it contains information about each **scene** in the game, which lines are supposed to be played within them (and which character is supposed to say each line).

The way the actual text is stored however is pretty interesting. It's contained in `dic`, encoded in `CP1250`. This file contains a list of the different words which are used in the game. That's right, **each word is stored separately**.

So now, you may ask how does the game display sentences from that? The answer is the `text` file. This file contains a list of indices. These indices are supposed to match the words that appear in `dic`.

That means that in order to write a sentence, one would have to store each word separately in `dic` and then write each word's index within `dic` into `text`.

I think that it's a pretty convoluted way of storing text but hey it was fun figuring that out.

Here's the technical part now. By the way, a lot of formats aren't really aligned. And integers are stored in little endian.

`struct` header:

| Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x1 | File length | `u32` | |
| 0x5 | Scene count | `u8`| |
| 0x6 | File start | `u32` | |
| 0xA | Scenes | `Scene[]` | |

Now the structure for each scene:

| Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x0 | Scene Name | String | Fixed length, 20 bytes |
| 0x14 | Line count | `u8` | |

And the structure for each line within a scene:

| Offset | Name | DataType | Notes |
| --- | --- | --- | --- |
| 0x4 | Line Index | `u16` | Points to the `text` file |
| 0x6 | Character Name | String | Fixed length, 26 bytes |

`dic` doesn't contain a structure that's too hard to understand, it's just an array of 35 bytes long strings.

Same thing for `text`. Each line of text is 64 bytes long. They're just `u16` arrays of 32 word indices.

Overall I had fun working on that game, the formats were quite simple to understand. This is work I've done one random afternoon in 2021.

I made a shitpost using what I found, but I can't seem to find it anymore sadly...
 


