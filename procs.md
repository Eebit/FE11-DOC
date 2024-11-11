## Procs

FE11, like many other Fire Emblem games, features a "proc" system which is used to enable the game to do pseudo-parallel processing on an otherwise single-threaded CPU. This is something that has been very well detailed by others in the past:
* GBAFE - Nat  - https://feuniverse.us/t/guide-doc-asm-procs-or-6cs-coroutines-threads-fibers-funky-structs-whatever/3352
* PoR / RD - Nat - https://github.com/StanHash/FE9-DOC/blob/master/Procs.txt
* FE5 - Zane - https://feuniverse.us/t/doc-asm-wip-procs-the-fe5-version/3358
* FE14 - Circles - https://feuniverse.us/t/fe14-misc-notes-on-fates/16232/3

For the sake of brevity, I won't cover the entire scope of what procs are since I think Nat's guide is already extremely thorough and comprehensive, and I would just be repeating a lot of that. Please read their research on it, and consider this to be the DSFE extension of that research.

---

The DSFE proc system is, as far as I can tell, quite similar to the GBAFE proc system. It features the same general interfaces for starting/finding/ending procs, and the code itself appears to be relatively similar too.

There are a few differences. For example, each proc can have its own "function table"; a pointer to a list of callable functions that is stored at +0x00 of the proc. The purpose of this isn't clear to me yet but I hope to understand how this complements the existing script-based format.

So far, I haven't been able to identify things like what order the proc trees are executed in, or whether that is terribly important in DSFE.

The Proc Tree is located in RAM at `FE11:0x02190F28`.

The function at `FE11:0x02018C58` seems to be equivalent to FE8's `Proc_Init`. It iterates over 0x80 procs and initializes their fields to zero values. Based on that, it seems that the max proc count is 0x80 in FE11.

There appears to be 14 proc trees in FE11, based on the same `Proc_Init` function.

Many of the core functions for interacting with the procs are located in the "ITCM" (or "Instruction Tightly-Coupled Memory") region. One of the features of the Nintendo DS is to give fast access to a limited amount of code.

What this means is that, at the time when the game is loaded, there is some code gets loaded the end of the ROM and put into a fast-access area. Most of the game's code refers to the functions in that particular area. It makes it a touch more confusing for us as hackers to find references to that code (or at some point in the future, to modify it).

The Proc Function table is located at `FE11:0x020E7898` in the ROM. It gets copied into the ITCM region, to `FE11:0x01FFBBF8`, when the game is running. In Nat's doc, this aligns with the "Code Interpretation Table".

Below, you can the locations of the proc functions along with some names that indicate their purposes (in most cases, I've tried to reuse the FE8 proc names where they seem to line up).

```
[00] 0x02019358 - ProcCmd_DELETE
[01] 0x02019358 - ProcCmd_DELETE
[02] 0x02019368 // returns 0
[03] 0x02019370 // returns 1
[04] 0x02019378 // advance and return 1
[05] 0x0201938c - ProcCmd_SET_DESTRUCTOR (?)
[06] 0x020193b4 - ? maybe set name?
[07] 0x020193dc - ProcCmd_CALL_ROUTINE
[08] 0x020193fc - ProcCmd_CALL_ARG_ADV
[09] 0x02019420 - ProcCmd_WHILE_ROUTINE
[0A] 0x0201945c - ProcCmd_CALL_ARG_ADVCOND (advance conditionally)
[0B] 0x0201951c - ?
[0C] 0x02019620 - ProcCmd_LOOP_ROUTINE
[0D] 0x02019640 - ProcCmd_WHILE_EXISTS
[0E] 0x02019674 - ProcCmd_NEW_CHILD
[0F] 0x020196a0 - ProcCmd_NEW_CHILD_BLOCKING
[10] 0x020196cc - ProcCmd_NEW_PROC_IN_TREE (?)
[11] 0x020196f8 - ProcCmd_END_ALL (sarg skips an additional pointer @ 027E1268)
[12] 0x02019734 - ProcCmd_BREAK_ALL (sarg skips an additional pointer @ 027E1268)
[13] 0x02019954 - ProcCmd_NOP_1D ? label?
[14] 0x02019770 - ProcCmd_GOTO
[15] 0x0201979c - ProcCmd_GOTO_IF_YES (if larg = 0 or larg func returns non-zero, goto)
[16] 0x020197e8 - ProcCmd_GOTO_IF_NO (same but if larg func returns zero)
[17] 0x02019834 - ProcCmd_JUMP
[18] 0x02019874 - ProcCmd_SLEEP
[19] 0x020198a4 - ProcCmd_SET_MARK (?)
[1A] 0x020198c4 - (?) end children?
[1B] 0x020198f8 - (?) end children?
[1C] 0x0201992c - (?) end children?
[1D] 0x02019954 - ProcCmd_NOP_1D
[1E] 0x02019968 - start a specific child (blocking if larg not &1)
[1F] 0x020199b8 - start a specific child (blocking if larg not &1)
[20] 0x02019a08 - start a specific child (blocking if larg not &1)
[21] 0x02019a58 - start a specific child (blocking if larg not &1)
[22] 0x02019aa8 - start a specific child (blocking if larg not &1)
[23] 0x02019af8 - start a specific child (blocking if larg not &1)
[24] 0x02019b48 - ProcCmd_OVERLAY - load or unload overlay; overlay ID = sarg, load/unload = larg
[25] 0x02019b84 - ProcCmd_ - maybe reset overlays? Unload all except sarg? Load only sarg?
```
