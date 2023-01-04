---
layout: default
---

# Home

Hey ! I'm Obluda, a programmer that's mostly focused on making hacks for videogames. I'm familiar with C#, C/C++ and PowerPC Assembly (for the wii).

I discovered reverse engineering in 2020 and it became my main hobby! I used to focus on learning how games were made, now I also try to implement new things in them.

You can follow me on [Twitter](https://www.twitter.com/obluda3), the [Fediverse](https://wetdry.world/@obluda) or message me on Discord **@Obluda#1526**. 🤓 
 
# Projects

Here you'll find many of my different video games **projects** for those that I reversed in the past.

## Inazuma Eleven GO Strikers 2013 Documentation

Over the past two years I've spent a lot of time **reverse engineering** Strikers 2013's code, data structures and file formats. 
[Here](/strikers) you'll find a **documentation** of the things that I (and other people) have learnt so far.

## Inazuma Eleven GO Strikers 2013 Xtreme

![Screenshot of Xtreme13's titlescreen](/images/xtreme_titlescreen.png)

[Inazuma Eleven GO Strikers 2013 Xtreme](https://www.xtreme13.com/) is a mod aiming to complete the original game by restoring its unused content in addition to some extra features. 

I was the **main developer** behind this mod. We started working on it in **late 2020** (back then it didn't really have a name). It was released on *August 26 2022*.

These *extra features* were the results of the reverse engineering work I've done on this game. It includes a **music player/selector**, **random team generator**, **displaying hidden info** and stuff like that.

It uses [Kamek](https://github.com/Treeki/Kamek) by [Ninji](https://twitter.com/_ninji) to inject custom code into the game.

## Inazuma Eleven GO Strikers 2013 modding tools

I've made multiple tools for some reason, when I could've just made an *all-in-one*. Actually there's a reason but we'll save that story for another time...

### Strikers2013Editor

[Strikers2013Editor](https://github.com/obluda3/Strikers2013Editor) is used to modify **players**, **teams** and **moves** data structures in Strikers 2013. It also includes a save editor (although it's a bit uncomplete).

### Strikers2013-teambuilder

[Strikers2013-teambuilder](https://github.com/obluda3/strikers2013-teambuilder) is the first program I made back in *December 2020*. It was used to generate teams using gecko codes. It was the first accessible way to add unused players to one's team.

### Strikers2013-Tools

[Strikers2013-Tools](https://github.com/obluda3/strikers2013-tools) is a program I initialy made for the french fan-translation project of the game. I made it in order to *import* files into the games' archives, back when available tools like [Kuriimu2]() didn't support them. I also added support for the game's text format, and later re-implemented the game's [compression algorithm]().

## Street Football

Street Football is a video game based on the TV Show *Foot 2 Rue*. I worked a little bit on the game, for shitposting purposes...

[Here](/f2r.md)'s a documentation of the game's file formats. And [Here](/f2r_script.txt) you'll get a full dump of the game's script.

## Croaarc

[Croaarc](https://github.com/obluda3/Croaarc) is an archive extractor/packer that I made for **Webfoot** games. 
I quickly made it after figuring out the structure for the archive format used in them.

I plan to continue my work on these games but I don't really know when.

# Tutorials 

- Using a hex editor to figure out some structs
- Adding musics to Xtreme 2013
- Adding an emblem to Xtreme 2013