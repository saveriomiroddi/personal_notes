Memory layout:

- 4K RAM (0x1000)
- 0x000-0x1FF: Originally, O/S (512 bytes)
- 0xEA0-0xEFF: reserved (misc)
- 0xF00-0xFFF: reserved (display refresh)

Registers:

- 16 (16 bits): V0-VF; VF is also for flags
- VF
  - on additions: carry flag
  - on subtractions: "no borrow" flag
  - on draw instruction: set on pixel collision

Stack:

- stored address when subroutines are called
- original: 48 bytes (12 levels)

Timers:

- two; 60 Hz, count down to 0
- delay timer: R/W, for game events
- sound timer: for effects; when non-zero, the beeping sound is emitted

Input:

- via hex keyboard (?)
- keys 0 to F
- directions are typically 8, 4, 6, 2
- three opcodes detect input
  1. skip instruction if specific key is pressed
  2. skip instruction if specific key is not pressed
  3. wait for keypress, and store in a register

Graphics

- 64x32, monochrome
- graphics are drawn via sprites (8px wide, 1->15 high)
- sprites XOR with the screen pixels
- pixel collision: VF set when screen pixel is unset due to collision with sprite set pixel

#### Opcodes (SUPER-CHIP)

- NNN: address
- NN: 8-bit constant
- N: 4-bit constant
- X and Y: 4-bit register identifier
- PC : Program Counter
- I : 16bit register (For memory address) (Similar to void pointer)
- VN: One of the 16 available variables. N may be 0 to F (hexadecimal)

| Opc.  | Type    | C Pseudo                                               | Explanation                                                                                                                                                                                                                                                                                                                                                                             |
| ----- | ------- | ------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0NNN  | Call    |                                                        | Calls machine code routine (RCA 1802 for COSMAC VIP) at address NNN. Not necessary for most ROMs.                                                                                                                                                                                                                                                                                       |
| 00E0  | Display | disp_clear()                                           | Clears the screen.                                                                                                                                                                                                                                                                                                                                                                      |
| 00EE  | Flow    | return;                                                | Returns from a subroutine.                                                                                                                                                                                                                                                                                                                                                              |
| 1NNN  | Flow    | goto NNN;                                              | Jumps to address NNN.                                                                                                                                                                                                                                                                                                                                                                   |
| 2NNN  | Flow    | *(0xNNN)()                                             | Calls subroutine at NNN.                                                                                                                                                                                                                                                                                                                                                                |
| 3XNN  | Cond    | if(Vx==NN)                                             | Skips the next instruction if VX equals NN. (Usually the next instruction is a jump to skip a code block)                                                                                                                                                                                                                                                                               |
| 4XNN  | Cond    | if(Vx!=NN)                                             | Skips the next instruction if VX doesn't equal NN. (Usually the next instruction is a jump to skip a code block)                                                                                                                                                                                                                                                                        |
| 5XY0  | Cond    | if(Vx==Vy)                                             | Skips the next instruction if VX equals VY. (Usually the next instruction is a jump to skip a code block)                                                                                                                                                                                                                                                                               |
| 6XNN  | Const   | V_x_ = NN                                              | Sets VX to NN.                                                                                                                                                                                                                                                                                                                                                                          |
| 7XNN  | Const   | V_x_ += NN                                             | Adds NN to VX. (Carry flag is not changed)                                                                                                                                                                                                                                                                                                                                              |
| 8XY0  | Assign  | V_x_=Vy                                                | Sets VX to the value of VY.                                                                                                                                                                                                                                                                                                                                                             |
| 8XY1  | BitOp   | Vx=V_x_ \| V_y_                                        | Sets VX to VX or VY. (Bitwise OR operation)                                                                                                                                                                                                                                                                                                                                             |
| 8XY2  | BitOp   | Vx=V_x&_V_y_                                           | Sets VX to VX and VY. (Bitwise AND operation)                                                                                                                                                                                                                                                                                                                                           |
| 8XY3ᵃ | BitOp   | Vx=Vx^Vy                                               | Sets VX to VX xor VY.                                                                                                                                                                                                                                                                                                                                                                   |
| 8XY4  | Math    | Vx += Vy                                               | Adds VY to VX. VF is set to 1 when there's a carry, and to 0 when there isn't.                                                                                                                                                                                                                                                                                                          |
| 8XY5  | Math    | Vx -= Vy                                               | VY is subtracted from VX. VF is set to 0 when there's a borrow, and 1 when there isn't.                                                                                                                                                                                                                                                                                                 |
| 8XY6ᵃ | BitOp   | Vx>>=1                                                 | Stores the least significant bit of VX in VF and then shifts VX to the right by 1.ᵇ                                                                                                                                                                                                                                                                                                     |
| 8XY7ᵃ | Math    | Vx=Vy-Vx                                               | Sets VX to VY minus VX. VF is set to 0 when there's a borrow, and 1 when there isn't.                                                                                                                                                                                                                                                                                                   |
| 8XYEᵃ | BitOp   | Vx<<=1                                                 | Stores the most significant bit of VX in VF and then shifts VX to the left by 1.ᵇ                                                                                                                                                                                                                                                                                                       |
| 9XY0  | Cond    | if(Vx!=Vy)                                             | Skips the next instruction if VX doesn't equal VY. (Usually the next instruction is a jump to skip a code block)                                                                                                                                                                                                                                                                        |
| ANNN  | MEM     | I = NNN                                                | Sets I to the address NNN.                                                                                                                                                                                                                                                                                                                                                              |
| BNNN  | Flow    | PC=V0+NNN                                              | Jumps to the address NNN plus V0.                                                                                                                                                                                                                                                                                                                                                       |
| CXNN  | Rand    | Vx=rand()&NN                                           | Sets VX to the result of a bitwise and operation on a random number (Typically: 0 to 255) and NN.                                                                                                                                                                                                                                                                                       |
| DXYN  | Disp    | draw(Vx,Vy,N)                                          | Draws a sprite at coordinate (VX, VY) that has a width of 8 pixels and a height of N pixels. Each row of 8 pixels is read as bit-coded starting from memory location I; I value doesn’t change after the execution of this instruction. As described above, VF is set to 1 if any screen pixels are flipped from set to unset when the sprite is drawn, and to 0 if that doesn’t happen |
| EX9E  | KeyOp   | if(key()==Vx)                                          | Skips the next instruction if the key stored in VX is pressed. (Usually the next instruction is a jump to skip a code block)                                                                                                                                                                                                                                                            |
| EXA1  | KeyOp   | if(key()!=Vx)                                          | Skips the next instruction if the key stored in VX isn't pressed. (Usually the next instruction is a jump to skip a code block)                                                                                                                                                                                                                                                         |
| FX07  | Timer   | Vx = get_delay()                                       | Sets VX to the value of the delay timer.                                                                                                                                                                                                                                                                                                                                                |
| FX0A  | KeyOp   | Vx = get_key()                                         | A key press is awaited, and then stored in VX. (Blocking Operation. All instruction halted until next key event)                                                                                                                                                                                                                                                                        |
| FX15  | Timer   | delay_timer(Vx)                                        | Sets the delay timer to VX.                                                                                                                                                                                                                                                                                                                                                             |
| FX18  | Sound   | sound_timer(Vx)                                        | Sets the sound timer to VX.                                                                                                                                                                                                                                                                                                                                                             |
| FX1E  | MEM     | I +=Vx                                                 | Adds VX to I. VF is not affected.ᶜ                                                                                                                                                                                                                                                                                                                                                      |
| FX29  | MEM     | I=sprite_addr\[Vx\]                                    | Sets I to the location of the sprite for the character in VX. Characters 0-F (in hexadecimal) are represented by a 4x5 font.                                                                                                                                                                                                                                                            |
| FX33  | BCD     | set_BCD(Vx);*(I+0)=BCD(3);*(I+1)=BCD(2);*(I+2)=BCD(1); | Stores the binary-coded decimal representation of VX, with the most significant of three digits at the address in I, the middle digit at I plus 1, and the least significant digit at I plus 2\. (In other words, take the decimal representation of VX, place the hundreds digit in memory at location in I, the tens digit at location I+1, and the ones digit at location I+2.)      |
| FX55  | MEM     | reg_dump(Vx,&I)                                        | Stores V0 to VX (including VX) in memory starting at address I. The offset from I is increased by 1 for each value written, but I itself is left unmodified.ᵈ                                                                                                                                                                                                                           |
| FX65  | MEM     | reg_load(Vx,&I)                                        | Fills V0 to VX (including VX) with values from memory starting at address I. The offset from I is increased by 1 for each value written, but I itself is left unmodified.ᵈ                                                                                                                                                                                                              |

- ᵃ) The logical opcodes 8XY3, 8XY6 and 8XYE were not documented in the original CHIP-8 specification, as all the 8000 opcodes were dispatched to instructions in the 1802's ALU,
     and not located in the interpreter itself; these three additional opcodes were therefore presumably unintentional functionality.
- ᵇ) CHIP-8's opcodes 8XY6 and 8XYE (the bit shift instructions), which were in fact undocumented opcodes in the original interpreter, shifted the value in the register VY and
     stored the result in VX. The CHIP-48 and SCHIP implementations instead ignored VY, and simply shifted VX.
- ᶜ) Most CHIP-8 interpreters' FX1E instructions do not affect VF, with one exception: The CHIP-8 interpreter for the Commodore Amiga sets VF to 1 when there is a range overflow
    (I+VX>0xFFF), and to 0 when there isn't. The only known game that depends on this behavior is Spacefight 2091! while at least one game, Animal Race, depends on VF not being
    affected.
- ᵈ) In the original CHIP-8 implementation, and also in CHIP-48, I is left incremented after this instruction had been executed. In SCHIP, I is left unmodified.