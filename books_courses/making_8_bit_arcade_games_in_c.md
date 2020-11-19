# Making 8-bit Arcade Games in C

- [Making 8-bit Arcade Games in C](#making-8-bit-arcade-games-in-c)
  - [6. The VIC Dual Hardware](#6-the-vic-dual-hardware)
  - [7. Siege Game](#7-siege-game)
  - [8. Shift and Rotate](#8-shift-and-rotate)
  - [9. The Programmable Sound Generator](#9-the-programmable-sound-generator)
  - [10. Binary-Coded Decimal](#10-binary-coded-decimal)
  - [11. Pointers](#11-pointers)
  - [12. The Midway 8080 Hardware](#12-the-midway-8080-hardware)
  - [13. Game: Cosmic Impalas](#13-game-cosmic-impalas)
  - [14. Interrupts](#14-interrupts)
  - [16. Initializing memory](#16-initializing-memory)
  - [17. The Galaxian Hardware](#17-the-galaxian-hardware)
  - [18. Game: Solarian](#18-game-solarian)
  - [20. Integers and You](#20-integers-and-you)
  - [23. Game: Crowded Orbit](#23-game-crowded-orbit)
  - [25. Miscellaneous C topics](#25-miscellaneous-c-topics)
  - [28. Run-Length Encoding](#28-run-length-encoding)
  - [Errata](#errata)

## 6. The VIC Dual Hardware

Immediate value prefixes:

```c
palette & 0b00010000
palette & 0x10
```

Memory address functions:

```c
// Set memory address of `cellram`
byte __at (0xe000) cellram[28][32]

// I/O port 0x40 operations (sends 1 to I/O port 0x40)
// "Special Function Register"
__sfr __at (0x40) palette
palette = 1
if (palette & 0x10) {}
```

Memory functions (`string.h`):

```c
memcpy(dest, source, sizeof(array));
memset(addr, value, numvalues)
```

Macros:

```c
#define PE(fg,bg) (((fg)<<5) | ((bg)<<1))
```

Typedefs:

```c
typedef unsigned char byte;
typedef enum { false, true } bool;
```

Inline assembler:

```c
void start() {
__asm
        LD SP,#0xE800 ;
        DI            ;
__endasm
}
```

## 7. Siege Game

Structs:

```c
// Explicit
typedef struct {
  byte x;
  byte x;
} Player;

Player players[2];

// Inline

struct {
  byte x;
  byte x;
} players[2];

players[0].x = 15;
```

In C, constants and macros are often defined in upper case.

XOR:

```c
  unsigned char val = 0xFF ^ 0x80;
```

## 8. Shift and Rotate

Bit shift:

```c
score <<= 1;
```

translates (in theory, not in practice, due to optimizations) to:

```asm
ld   iy,#_score
sla  0 (iy)       ; left shift low byte
rl   1 (iy)       ; left rotate (9-bit) high byte with carry
```

`RL` is used because `SLA` doesn't set the bit 0 to carry.

## 9. The Programmable Sound Generator

Negate a value:

```c
~value
```

`Constant folding`: when it's possible to compute values at compile-time, it's done by the compiler. This is an advantage of inlining functions.

Avoid `unreferenced variable` warnings:

```c
void pizza(int a)
{
  a;      // !!
__asm
  ; blahblah
__endasm;
}
```

## 10. Binary-Coded Decimal

(not really interesting)

## 11. Pointers

Assign an immediate value (address) to a pointer:

```c
byte* ptr = (byte*)0x4800;
```

Interesting pointer arithmetic (!!):

```c
while (*string)
  printf(x++, y, *string++);

// the inner block equals to:

putchar(x, y, *string);
x++;
string++;
```

## 12. The Midway 8080 Hardware

Interesting macro usage:

```c
#define LOCHAR 0x20
#define CHAR(ch) ((ch)-LOCHAR)

char ch = 'b';
const byte* src = &fontTable[CHAR(ch)][0];
```

`volatile` signals the optimizer that consecutive reads of the variable can return different values:

```c
volatile __sfr __at (0x3) bitshift_read;
```

## 13. Game: Cosmic Impalas

Bit fields: allocate only bits, although they're generally slower than using full bytes:

```c
struct {
  byte xAxis:1;
  byte yAxis:1;
} MarchMode
```

`memmove`: use when source and destination superpose:

```c
memmove(source - 1, source, sizeof(MyStruct))
```

## 14. Interrupts

`__interrupt`  generates boilerplate for handling interrupts (`EI`, `PUSH` all regs, code, `POP` all regs, `RETI`)

```c
void scanline224() __interrupt;
```

`RETI` must be used when returning from an interrupt.

## 16. Initializing memory

Variables allocation:

- `int uninit_value`:              `_DATA` seg.
- `const int const_value = 0x123`: `_CODE` seg., with value assigned
- `int init_value = 0x7f`:         `_INITIALIZED` seg., without value; the value is in the `_INITIALIZER` seg., and copied on program start

Differences `const char[]` vs. `string`:

1. `const char[]` is not null terminated
2. `const char[]` is an aggregate type, so although it's coerced into a pointer when used, **its size is its length, not the pointer size**

## 17. The Galaxian Hardware

`memset` **may not be suitable on all platforms**. For example, in the Galaxian HW, it sets the first location to the given value, the copies it to the other locations; since read from certain areas has unspecified behavior, garbage will be written.

## 18. Game: Solarian

When a struct is meant to be stored in arrays, it's best to pad it, so that its size is a power of two. Subarrays also perform best when their size is a power of two.

## 20. Integers and You

Unsigned divisions by two can't be optimized, because of sign extension:

```
11110111 -> -9  // first bit is sign
11111011 -> -5
11111101 -> -3
11111110 -> -2
11111111 -> -1
```

they never go to zero!

The following generates suboptimal code:

```c
char in_range(unsigned char x, unsigned char start, unsigned char size) {
  return x >= start && x <= start+size;
}
```

due to **integer conversion rules**! If 8-bit values are added/subtracted, an intermediate 16-bit value is used. If a platform doesn't efficiently handle 16 bit value, the operation will be large and slow.

More performing version:

```c
char in_range(unsigned char x, unsigned char start, unsigned char size) {
  return (unsigned char)(x-start) <= size;
}
```

note that this is not only faster due to the data type optimization, but also due to clever redundancy of the first comparison.

## 23. Game: Crowded Orbit

Allocate/deallocate memory:

```c
Actor* a = (Actor*) malloc(sizeof(Actor));
free(a);
```

the return data type of `malloc` is `(void*)`, so we need to convert.

Function pointer, via typedef:

```c
typedef void ActorUpdateFn(struct Actor*);
ActorUpdateFn* update_fn; // pointer to function
```

the function pointer can point to any function having the same function signature as `ActorUpdateFn`.

Standard definition (from the web):

```c
// Using referencing operator
int (*myFunctionPtr)(int,int) = &myFunction;
int result = (*myFunctionPtr)(2, 3);

// Not using it
int (*myFunctionPtr)(int,int) = myFunction;
int result = myFunctionPtr(2, 3);
```

An interesting feature to use is a pointer to pointer, typical of linked lists:

```c
Klazz** previous = &current->next;
current = *previous;

if (current->removed)
  *previous = current->next;
  // ...
```

When testing collision detection, we perform a radius test (d < √(x²+y²)). When this is expensive, the "Manhattan distance" (d < x+y) can be used as cheap filter.

## 25. Miscellaneous C topics

Unions (ugly as ---t):

```c
union {
struct { sbyte dx,dy; } laser;
struct { byte exploding; } enemy;
} u;
```

## 28. Run-Length Encoding

Packet types:

- `Run`: string of identical bytes
- `Copy`: straight copy of the packet bytes

## Errata

To future-proof the code examples from compiler updates, some code has been changed.

Some interrupt handers must start at a specific address, and the SDCC compiler currently does not have an easy way to force this. Therefore, the current approach is to put padding bytes between functions so that when lined up in memory, the interrupt handlers are at the right address.

We’ve modified the Galaxian-Scramble demo game to make this more predictable. We insert padding bytes at the end of the start() routine’s `__asm` block:

```
.ds   0x66 - (. - _start)
```

The .ds directive means “insert N bytes here.” `(. - _start)` is the current program counter subtracted from the `_start` label. We do this to convert the program counter to an absolute constant (stuff specific to the SDCC assembler). Then we subtract that value from 0x66, which is where we want our next function to reside.

We also use the `__naked` function decorator so that the compiler doesn’t insert a RET instruction or stack frame instructions, modifying our code length.

The end result of this is that our next C function definition will reside at address 0x66, where we want it.

The `bcd_add()` function has also been changed, as the inline assembly made assumptions about the code that are not portable across compiler flags. It also now uses the `__naked` decorator.

In the VIC Dual sections, RAM is defined as starting at $8000, but the code defines it at the mirrored location $c000. Note that it is a 32 x 32 array (1024 bytes) but only 28 rows (896 bytes) are visible.

p. 41: The line `typedef unsigned char sbyte` should define a `signed char` type, not `unsigned char`.

Shift/Rotate Chapter: The function definition for halve_score has a redundant `LD IY,#_score` instruction.

On page 115, the field `_unused` is padding for the `AttackingEnemy` struct on page 114, but there is no such field in the example code.
