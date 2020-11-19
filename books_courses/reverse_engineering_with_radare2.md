# Reverse Engineering With Radare2

- [Reverse Engineering With Radare2](#reverse-engineering-with-radare2)
  - [1.5 Target Application](#15-target-application)
  - [2.7 Radare2 Syntax](#27-radare2-syntax)
  - [2.8 Configuration](#28-configuration)
  - [2.9 Binary Infos](#29-binary-infos)
  - [2.10 Navigating in the binary](#210-navigating-in-the-binary)
  - [2.11 Cross References](#211-cross-references)
  - [2.12 Runtime Debugging](#212-runtime-debugging)
  - [2.13 Patching](#213-patching)
  - [2.14 Cutter the R2 GUI](#214-cutter-the-r2-gui)

## 1.5 Target Application

File informations: `file <filename>`; `not stripped`: (some) debug information, eg. function names, is present

## 2.7 Radare2 Syntax

`?`: help; can be used in combination with almost anything, eg. `a?`
`afl`: (analysis) Functions -> List; we first need to analyze the binary, otherwise, nothing's printed
`aaa`: Analyze -> All -> Autoname
`q`: quit (current context); useful when one gets lost - tap until reaching the base prompt

## 2.8 Configuration

`e`: (evaluable) Vars; displays them
`e`: Visual (evaluable) Vars; visual mode (interactive)
`e <variable> =?`: show allowed values for the variable

## 2.9 Binary Infos

`i`: Informations (about the binary)
`ii`: Informations about Imports; very useful about getting a general idea of what the program does (uses)

`<command>~<pattern>` Greps the output of the command (not regex!)

`iz`: Strings in the data section; very important - one of the first things to look at (use second col for x-refs)

## 2.10 Navigating in the binary

`s <reference>`: Seek to reference; can be address or symbolic, eg. `sym.main` (program main address)
`pdf [@<reference>]`: Print Disassemble Function; if `@<reference>` is not provided, it prints the current function.
                      at the top, the variable names and addresses are printed
`? <value|operation>`: Prints various interpretations/informations related to the value, which could point to something interesting (including, a string at that location)
`CC <comment> [@address]`: add a Comment to the specified address (or current one, if unset)

## 2.11 Cross References

`axt <reference>`: (Analyze) find cross (X) references To the reference
`pd <lines>@<reference>`: Print Disasseble the given number of lines from the reference
  - `$$` is the reference to the current location

## 2.12 Runtime Debugging

`r2 -d <filename>`: Start R2 in debugging mode
`do`: (Debug) restart (Open) process
`db`: (Debug) list Breakpoints
`db [<reference>`]: (Debug) put Breakpoint
`dc`: Debug Continue
`v`: Visual mode (has several modes)
  - `p`: rotate (Print) modes (hex, disasm, debug, words, buf)
    - in disasm mode
      - breakpoints are marked with `b`
      - `V`: flow graph for the current function!
        - `?` help!
  - `?`: help
  - `q` exit (Quit) current context
  - `S`: Step over
  - `s`: Step in
  - `:`: enter command
    - `px <location>` Print heX dump of location (eg. `4@ebp + 0x8` = 4 bytes starting at EBP+8)

The flow graph can also be displayed directly using `VV` (interactive) or `agf` (non-interactive).

Functions (??which ones??) can be found in the manpages! Eg. `sys.read` -> `man read`.

Remember that arguments are pushed in reverse order (eg. first pushed is last argument).

## 2.13 Patching

`wao <operation>` : patch (Write Assembler) modification to the current Opcode.
  - sample operations: `nop, jz`, `trap`, `recj` (invert jump condition!!!)

`r2 -w`: required in order to perform writes to the binary; writes will be written straight away!!
`s <reference>`: seek to reference (change the current reference (not related to execution!))
`wa <instruction>`: patch (Write) modification to the current Opcode

## 2.14 Cutter the R2 GUI

- `Flare On Challenge`
  - http://www.flare-on.com -> `cd PastResults` -> `cd 2018` -> `cd Downloads` -> download zipfile -> open `03_FLEGGO.7z` -> unzip `FLEGGO.zip` (pwd: `infected`)
- In Graph view, `Ctrl+Mouse scroll` zooms in/out
- On an address, `x`: find cross (X) references
- The arrows (top left) are used from browsing back/forward
- On a function, `N`: rename function (available in menu)
- The `Sections`/`Resources` windows are under `Window` -> `Info`
- There is a `Console` window
- `psx <address>`: Print string with escaped chars
