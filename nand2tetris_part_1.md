# Coursera: NAND2Tetris (Part 1)

- [Coursera: NAND2Tetris (Part 1)](#coursera-nand2tetris-part-1)
  - [Week 1](#week-1)
  - [Week 2](#week-2)
  - [Week 3](#week-3)
  - [Week 4](#week-4)
    - [4.4 Hack language](#44-hack-language)
      - [ASM general concepts](#asm-general-concepts)
      - [Registers](#registers)
      - [Instructions format](#instructions-format)
      - [Instructions](#instructions)
        - [@](#)
        - [C instruction](#c-instruction)
    - [4.5 I/O](#45-io)
      - [Screen](#screen)
      - [Keyboard](#keyboard)
    - [4.6 Programming (I)](#46-programming-i)
      - [Assignments and basic arithmetic](#assignments-and-basic-arithmetic)
      - [Symbols](#symbols)
    - [4.6 Programming (II)](#46-programming-ii)
      - [Branching](#branching)
      - [Variables](#variables)
      - [Iteration](#iteration)
    - [4.7 Programming (III)](#47-programming-iii)
      - [Pointers](#pointers)
      - [I/O](#io)

## Week 1

Commutative ab=ba
Associative a(bc) = (ab)c
Distributive a@(b#c) = (a@b)#(a@c)
De Morgan NOT(a@b) = NOT(a)#NOT(b)

Disjunctive normal form formula
- build AND conditions for rows with value 1
- build OR condition with all of them
- simplify

NAND 0=only 11

(logic gates diagram)

Hardwire Representation of an AND gate

Example of XOR gate diagram

HDR diagram of the XOR gate

VHDL, verilog

multiplexor: a and b, selector decides a or b as out
demultiplexor: a; selector decides if goes into a or b

a0 is the least significant bit
a15 is the most significant bit

## Week 2

2's complement: 2^n - x, eg.: -1 = 2^3 - 1 = 15 = 1111

negation: -x = 2^n + x = 1 + (2^n-1 + x) = 1 + NOT(x)

add 1: keep adding  1 from the right, until a 0 is met

ALU: f= +/&

## Week 3

Clock: Oscillator

Combinatorial logic (doesn't depend on time)  (- -) Sequential (depends on time)

Clocked data flip-flop: returns the input value at the previous cycle

Triangle symbol: the chip depends on time

1/n-bits register: n*CDDF with load

A cycle is represented with a tick/tock; both are required in order to get the full effect

Program counter: has load/inc/reset

## Week 4

- [Coursera: NAND2Tetris (Part 1)](#coursera-nand2tetris-part-1)
  - [Week 1](#week-1)
  - [Week 2](#week-2)
  - [Week 3](#week-3)
  - [Week 4](#week-4)
    - [4.4 Hack language](#44-hack-language)
      - [ASM general concepts](#asm-general-concepts)
      - [Registers](#registers)
      - [Instructions format](#instructions-format)
      - [Instructions](#instructions)
        - [@](#)
        - [C instruction](#c-instruction)
    - [4.5 I/O](#45-io)
      - [Screen](#screen)
      - [Keyboard](#keyboard)
    - [4.6 Programming (I)](#46-programming-i)
      - [Assignments and basic arithmetic](#assignments-and-basic-arithmetic)
      - [Symbols](#symbols)
    - [4.6 Programming (II)](#46-programming-ii)
      - [Branching](#branching)
      - [Variables](#variables)
      - [Iteration](#iteration)
    - [4.7 Programming (III)](#47-programming-iii)
      - [Pointers](#pointers)
      - [I/O](#io)

### 4.4 Hack language

#### ASM general concepts

Register types:

- data (only)
- address (only)

Addressing modes:

- register: `add r1, r2`
- immediate: `add r1, 73`
- direct: `add r1, $200`
- indirect: `add r1, [si]`

#### Registers

- `A`: addressing
- `D`: data
- `M`: selection -> always `RAM[A]`

#### Instructions format

```
`@val`                    # A := val
`M=val`                   # RAM[A] := val
[dest =] comp; [jump]     # ("C instruction")
```

```
// Comp

(0, 1, -1)
[-, !](D, A, M)
(D, A, M)(+, -)1
D(+, -)(A, M)
A-(D, M)
D(&, |)(A, M)

// Dest

(D,A,M), AM, AD, MD, AMD

// Jump (to M)

J(LT, LE, EQ, GE, GT, JNE, JMP)
JEQ = JZ
```

Example:

```
@56
D-1; JEQ
```

#### Instructions

##### @

Binary: 0 + <15 bit value>

##### C instruction

Binary: 111 a c[1..6] d[1..3] j[1..3]

- bits 1,2: unused
- a..c6: control bits

### 4.5 I/O

#### Screen

The screen is continuously refreshed by reading from the screen memory map.

Base address: 16384
256x512, 1 bit/pixel
32x16 bits per line

Represented via the `Screen` chip, with a 13-bits address (8k=256x512/16), and a 16-bits input (+load pin).

#### Keyboard

Keyboard has a memory map, too (address: 24576).

Represented by the `Keyboard` chip (16-bits).

As long as a key is pressed, the scan code appears in the map. No key = `0`.

### 4.6 Programming (I)

Each instruction is stored in ROM, and has an implicit line number (determined by the address); whitespaces aren't stored.

#### Assignments and basic arithmetic

```hdl
// D=10
@10
D=A

// D++
D=D+1

// D=RAM[17]
@17
D=M

// RAM[17]=D
@17
M=D

// RAM[17]=10
@10
D=A
@17
M=D

// RAM[5]=RAM[3]
@3
D=M
@5
M=D

// RAM[2]=RAM[0] + RAM[1]
@0
D=M
@1
D=D+M
@2
M=D
```

In order to terminate, one can:

```hdl
@<this address>
0;JMP // infinite loop
```

#### Symbols

Symbols are upper case; convenience for addressing the first 16 words.

- `R<0..15>` = value
- `SCREEN` = 16384
- `KBD` = 24576
- `SP` = 0
- `LCL` = 1
- `ARG` = 2
- `THIS` = 3
- `THAT` = 4

Example usage of `R<n>`:

```hdl
@R5  // = @5
M=D
```

### 4.6 Programming (II)

Each instruction is stored in ROM, and has an implicit line number (determined by the address); whitespaces aren't stored.

#### Branching

```hdl
// if R0 > 0
//   R1 = 1
// else
//   R1 = 0

@R0
D=M

@POSITIVE  // address label!
D; JGT

R1
M=0
@END       // use a label!
0; JMP

(POSITIVE)
R1
M=1

(END)
@END
0; JMP
```

#### Variables

Can use `@<varname>` to have a memory location automatically assigned. This makes the program *relocatable*.

```hdl
// temp = R1
// R1 = R0
// R0 = temp

@R1
D=M
@temp    // set an available memory location
M=D

@R0
D=M
@R1
M=D

@temp
D=M
@R0
M=D

(END)
@END
0; JMP
```

#### Iteration

Template:

```hdl
//   n = <val>
//   i = 0
// LOOP:
//   if i == n goto END
//   <operation>
//   i = i + 1
//   goto LOOP
// END:

@<val>   // n = <val>
D=A
@n
M=D
@i       // i = 0
M=0

(LOOP)
@i       // if i == n, goto end
D=M
@n
D=D - M
@END
D; JEQ

// <operation>

@i       // i = i + 1
M=M+1

@LOOP    // goto loop
0; JMP

(END)
@END
0; JMP
```

Example:

```hdl
//   n = R0
//   i = 1
//   sum = 0
// LOOP:
//   if i > n goto STOP
//   sum = sum + i
//   i = i + 1
//   goto LOOP
// STOP:
//   R1 = sum

@R0
D=M
@n
M=D
@i
M=1
@sum
M=0

(LOOP)
@i
D=M
@n
D=D - M
@STOP
D; JGT

@sum
D=M
D=D + M
@sum
M=D
@i
M=M+1

@LOOP
0; JMP

(STOP)
@sum
D=M
@R1
M=D

(END)
@END
0; JMP
```

### 4.7 Programming (III)

#### Pointers

```hdl
@100     // arr (len) = 100
D=A
@arr
M=D

@10      // n = 10
D=a
@n
M=D

@i       // i = 0
M=0

(LOOP)
@i       // if i == n ...
D=M
@n
D=D - M
@END     // ... goto stop
D; JEQ

// RAM[arr+i] = -1
@arr
D=M
@i
A=D + M
M=-1

@i       // i = i + 1
M=M+1

@LOOP    // goto loop
0; JMP

(END)
@END
0; JMP
```

#### I/O

RAM layout:

- (16k)    Data memory
- (8k)     Screen memory map
- (1 byte) Keyboard memory map
