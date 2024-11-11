## Overlays

The Nintendo DS employs a system called "overlays" to manage memory efficiently. Overlays allow the console to dynamically load code or data into memory only when needed and unload it when it's no longer required. This helps optimize the limited memory resources of the DS.

When you unpack a DSFE ROM using a tool like DSLazy, you will find a folder called "overlay", which contains the binary files for each overlay. Additionally, the file "y9.bin" contains metadata about the overlays:

```
Overlay (Size: 0x20):
00 | word | Overlay ID
04 | word | Load address (where the overlay is placed in memory at runtime)
08 | word | Overlay size
0C | word | BSS (uninitialized data) size
10 | word | Static initializer start address
14 | word | Static initializer end address
18 | word | File ID
1C | word | 0 (delimiter between overlays)
```

For a detailed explanation of overlays, I recommend checking out the [overlays section](https://www.starcubelabs.com/reverse-engineering-ds/#overlays) in the Starcube Laboratories post on Reverse Engineering a DS game.

FE11 features 12 overlays, which is relatively low compared to other DS games I've seen such as Pokemon HeartGold or Pokemon Mystery Dungeon: Explorers of Sky.

While the game is running, the table tracking the currently-loaded overlays is found at `FE11:0x020E4DF4`. It is represented by a series of bytes in memory. If an overlay is active, the byte value at that overlay's index will be 1. If it's inactive, the value will be 0.

The functions for managing overlays are as follows:
* Loading an overlay: `FE11:0x0200F20C`
* Unloading an overlay: `FE11:0x0200F24C`

There are additionally proc commands that involve loading/unloading overlays, which we will see later when I talk about the DSFE proc system.

From my initial research:
* Overlay 0 and 2 are typically always active.
* Overlay 4 seems related to core gameplay (e.g. battle maps).
* Overlay 6 might handle the title and save screen.

There is still lots more to be uncovered about the overlays/how they are used.
