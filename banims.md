## Battle Animations

Each binary file in the `/data/b/` directory represents an "archive" for that particular class's animations. I use the term "archive" because the binary file appears to consist multiple files.

Each class has one "archive" for its unarmed animation, and one "archive" per weapon that class can wield. Example: `LORD` for the unarmed Lord class, and `LORD_SW` for the Lord class wielding a sword.

The first 0x100 bytes are the archive's file table. This is an array of the following 8-byte structure:

```
00 | word | start of file
04 | word | file size
```

In practice, there are really two tables. The first half of the 0x100 bytes are the animation files, and the second half are the palette files. Effectively, each animation can have a maximum of 16 animations and 16 palettes.

0x000-0x080 - animation files
0x080-0x100 - palette files

The animation files are indexed by the type of animation. So far, these are the ones I have come across:

```
0: Dodge
1: Ranged Attack
2: Ranged Critical
3: Retreat (i.e. after attacking)
4: Idle
5: Magic Attack
6: Magic Critical
7: Melee Attack
8: Melee Double
9: Melee Critical
10: Melee Double Critical
```

---

### File header:

Each animation file has a "header" (size: 0x20 bytes) that describes a few attributes about the file, such as its size and the location of the "pointer table".

The pointer table's purpose appears to be to give the game the exact location of a pointer in the file, so that it can loop over the pointers and update them to their real value after being loaded into memory at runtime.

```
00    | word | size (excludes header)
04    | word | offset to pointer table (excludes header)
08    | word | number of pointers in the file
0C-20 | ?    | blank
```

### General animation metadata:

```
00-10 | ?     | blank? maybe populated at runtime
10    | short | frame count
12    | short | unknown
14    | word  | pointer to frame image / OAM data
18    | word  | pointer to sequence data
```

### Frame data (image / OAM):

Each frame of animation has information about the sprite that will be displayed. Some frames of animation have a "primary" and a "secondary" image, depending on their size. For example, the Cavalier class (SOCIALKNIGHT) has two images that need to be loaded.

I'm unsure if this is to divide the palette of the hair from the palette of the base sprite, but it seems to be that way at a glance.

```
00 | short | unknown
02 | short | OAM entry count
04 | word  | pointer to main image data ("primary image"); compressed
08 | word  | pointer to additional image data ("secondary image"); compressed
0C | word  | pointer to OAM data
```

### OAM Data:

Object Attribute Memory (OAM for short) relates to how the Nintendo DS draws sprites to the screen. Each sprite has 3 "attributes" that describe how the sprite gets constructed and drawn from the image data loaded into VRAM.

References for more information:

* https://feuniverse.us/t/what-is-oam/6193
* https://www.coranac.com/tonc/text/objbg.htm

```
00 | short | attribute 0 (y coord, mode, shape, etc.)
02 | short | attribute 1 (x coord, rotation/scaling, size)
04 | short | attribute 2 (char name, priority, palette)
```

### Sequence data:

I'm calling this the "sequence" data since it represents the order and duration of how each frame of animation is drawn. It seems somewhat similar to the standard DS "NANR" format, but dissimilar enough that it seems like Intelligent Systems may have their own custom adaptation.

```
00 | short | unknown
02 | short | unknown
04 | short | sequence entry count
06 | short | unknown
08 | word  | pointer to sequence entry list
```

### Sequence Entry:

Each "entry" in the sequence refers to a frame (i.e. the sprite/image details).

Most of the vanilla animations seem to have every frame of animation have a duration of 3 frames. In theory, this could be modified so that the frames have more "punch" to them.

The shorts 04 and 06 seem to adjust the position of the drawn frame.

```
00 | byte  | frame
01 | byte  | unknown
02 | short | duration (in frames)
04 | short | maybe x offset?
06 | short | maybe y offset?
```