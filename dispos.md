## Dispos

"Dispos" are, as far as I understand, the internal name for the data structure used to specify how to place units on the map. It's likely an abbreviation for "disposition". There are code references in the FE8 prototype that mention the term "dispos", and they're more commonly referenced by this name in hacking when we start seeing more strings in the code (such as FE9-10, and the 3DSFE games).

Each map has several "dispo groups" associated with it, which are loaded at different times (for example, for cutscenes or reinforcements). The layout of a single "dispo" entry is like this:

```
Dispo Entry (Size: 0x50):
00 | short | pid (character ID)
02 | short | jid (class ID)
04 | byte  | x position on load
05 | byte  | y position on load
06 | ?
09 | byte  | ?
0A | byte  | starting level
0B | byte  | ?
0C | word  | ?
10 | short[] | items; length 5?
1A | ?
1C | ?
24 | short | flags
26 | short | ?
28 | word[] | array of pointers to AI "types" (strings); length 4
38 | word[] | array of ?; length 4?
48 | byte | ?
49 | byte | ?
4A | short | ?
4C | byte | ?
4D | byte | ?
4E | ?
```

For more details on the dispo format, you can refer to the [FE11 Unit Disposition format documentation by Blazer]( https://github.com/laqieer/Fire-Emblem-Nightmare-Modules/blob/05958fa74be236b7b78aacffc96dd8e7a20edfe2/FE11%20Nightmare%20Modules/Unit%20Disposition%20Editors/Format%20Documentation.txt).
