# Gameboy Instruction-set Summary ! (In pseudocode)

## Introduction
aight so while i was making my gameboy emu, i kinda had to scramble a lil to find sources which had all the gameboy instructions and actually explain what they did.

it was a lil messy at first, and some parts were just missing, so i was like fucc it why not just write a summary of all the instructions ? 

this instruction summary is all done in pseudocode, so it'll follow some kinda c-ish python-ish syntax.

ill use some semantics, which will be explained down below. do note that this pseudocode is not at all cycle accurate ! anyways i hope i do an aight job at coding.

lastly, i reccommend that you don't start implementing your instructions with this, as it does lack some important shit.
i intend it to be as a basic reference for instruction behavior :D

## Semantics and Macros Used
here are the semantics used:

- `r8` - any 8-bit cpu register, like `A`
- `r16` - any 16-bit cpu register, like `BC`
- `u8` - an unsigned 8-bit value immediate
- `u16` - an unsigned 16-bit value immediate
- `var` - any value of any size at all
- `A, B, C, D, E, H, L, F` - any of the 8 u8 cpu registers
- `AF, BC, DE, HL` - any of the 4 u16 cpu registers
- `Z, S, HC, C` - any of the 4 1-bit flags located in `F`
- `cc` - An instruction condition in the form of a flag
- `[u16]` - a byte pointed to in memory by any u16
- `PC` - the u16 program counter in the cpu
- `SP` - the u16 stack pointer in the cpu
- `IME` - the interrupt master switch in the cpu
- `i8` - a *signed* 8 bit value
- `u3` - an unsigned 3-bit value immediate
- `0xXX` - a value denoted in hex form
- `0bXXXX` - a value denoted in binary form

here are the macros used:

```
PushStack (u16)
{
    [-- SP] = u16 >> 8
    [-- SP] = u16 & 0xff
}
```
```
PopStack ()
{
    var lo_byte = [SP ++]
    var hi_byte = [SP ++]

    return (hi_byte << 8) | lo_byte
}
```

notes:
- all cycles mentioned here are in t-cycles (1 m-cycle is 4 t-cycles)
- immediate values that pop outta nowhere in instructions are values fetched by the cpu, they come after the instruction and are in little-endian order.
- in instructions that use them, `u3` values are not fetched by the cpu, and are encoded into the opcodes themselves

## Table Of Contents

## Instruction Summaries

### ADC A, r8
```
var sum = A + r8 + C

Z = (sum & 0xff) == 0
S = 0
HC = ((A & 0xf) + (r8 & 0xf) + C) > 0xf
C = sum > 0xff

A = sum & 0xff
```
cycles: 4

### ADC A, [HL]
```
var sum = A + [HL] + C

Z = (sum & 0xff) == 0
S = 0
HC = ((A & 0xf) + ([HL] & 0xf) + C) > 0xf
C = sum > 0xff

A = sum & 0xff
```
cycles: 8

### ADC A, u8
```
var sum = A + u8 + C

Z = (sum & 0xff) == 0
S = 0
HC = ((A & 0xf) + (u8 & 0xf) + C) > 0xf
C = sum > 0xff

A = sum & 0xff
```
cycles: 8

### ADD A, r8
```
var sum = A + r8

Z = (sum & 0xff) == 0
S = 0
HC = ((A & 0xf) + (r8 & 0xf)) > 0xf
C = sum > 0xff

A = sum & 0xff
```
cycles: 4

### ADD A, [HL]
```
var sum = A + [HL]

Z = (sum & 0xff) == 0
S = 0
HC = ((A & 0xf) + ([HL] & 0xf)) > 0xf
C = sum > 0xff

A = sum & 0xff
```
cycles: 8

### ADD A, u8
```
var sum = A + u8

Z = (sum & 0xff) == 0
S = 0
HC = ((A & 0xf) + (u8 & 0xf)) > 0xf
C = sum > 0xff

A = sum & 0xff
```
cycles: 8

### ADD HL, r16
```
var sum = HL + r16

S = 0
HC = ((HL & 0xfff) + (r16 & 0xfff)) > 0xfff
C = sum > 0xffff

HL = sum & 0xffff
```
cycles: 8

### ADD HL, SP
```
var sum = HL + SP

S = 0
HC = ((HL & 0xfff) + (SP & 0xfff)) > 0xfff
C = sum > 0xffff

HL = sum & 0xffff
```
cycles: 8
bytes: 1

### ADD SP, i8
```
var sum = SP + i8

Z = 0
S = 0
HC = ((SP & 0xf) + (i8 & 0xf)) > 0xf
C = (sum & 0xff) < (SP & 0xff)

SP = sum & 0xffff
```
cycles: 16

### AND A, r8
```
var res = A & r8

Z = res == 0
S = 0
HC = 0
C = 0

A = res
```
cycles: 4

### AND A, [HL]
```
var res = A & [HL]

Z = res == 0
S = 0
HC = 0
C = 0

A = res
```
cycles: 8

### AND A, u8
```
var res = A & u8

Z = (res & 0xff) == 0
S = 0
HC = 0
C = 0

A = res
```
cycles: 8

### BIT u3, r8
```
Z = ((u8 >> u3) & 1) == 0
S = 0
HC = 0
```
cycles: 8

### BIT u3, [HL]
```
Z = (u8 >> u3) & 1) == 0
S = 0
HC = 0
```
cycles: 12

notes for BIT instructions: the u3 is not fetched, it is encoded in the opcodes themselves.

### CALL u16
```
PushStack(PC)
PC = u16
```
cycles: 24

### CALL cc, u16
```
if (cc)
    CALL u16
// The u16 fetch always happens
```
cycles: if cc: 24, else: 12

### CCF
```
S = 0
HC = 0
C = !C
```
cycles: 4

### CP A, r8
```
var res = A - r8

Z = (res & 0xff) == 0
S = 1
HC = (A & 0xf) < (r8 & 0xf)
C = A < r8
```
cycles: 4

### CP A, [HL]
```
var res = A - [HL]

Z = (res & 0xff) == 0
S = 1
HC = (A & 0xf) < ([HL] & 0xf)
C = A < [HL]
```
cycles: 8

### CP A, u8
```
var res = A - u8

Z = (res & 0xff) == 0
S = 1
HC = (A & 0xf) < (u8 & 0xf)
C = A < u8
```
cycles: 8

### CPL
```
A = ~A

S = 1
HC = 1
```
cycles: 4

### DAA
```
var res = 0
var should_set_c = 0

if (HC || (!S && (A & 0xf) > 0x9))
    res |= 0x6

if (C || (!S && A > 0x99))
    res |= 0x60
    should_set_c = 1

A = A + (S ? -res : res) & 0xff

Z = A == 0
HC = 0
C = should_set_c
```
cycles: 4

### DEC r8
```
var res = r8 - 1

Z = (res & 0xff) == 0
S = 1
HC = (res & 0xf) == 0xf

r8 = res & 0xff
```
cycles: 4

### DEC [HL]
```
var res = [HL] - 1

Z = (res & 0xff) == 0
S = 1
HC = (res & 0xf) == 0xf

[HL] = res & 0xff
```
cycles: 12

### DEC r16
```
r16 = (r16 - 1) & 0xffff
```
cycles: 8

### DEC SP
```
SP = (SP - 1) & 0xffff
```
cycles: 8

### DI
```
IME = 0
```
cycles: 4
### EI
```
IME = 1
```
cycles: 4

notes for EI: the effect of enabling IME only shows the instruction after. idk y but it do

### HALT
```
if (([0xff0f] & [0xffff]) & 0x1f)
    // Halt bug somewhere here ...
else
    // Stays halted
    PC = (PC - 1) & 0xffff
```
cycles: -

notes for HALT: this instruction is a big WIP ! it doesn't include the halt bug ... for now ? 
also, there are multiple ways to implement this instruction, so my method might not be the best.

### INC r8
```
var sum = r8 + 1

Z = (sum & 0xff) == 0
S = 0
HC = (sum & 0xf) == 0

r8 = sum & 0xff
```
cycles: 4

### INC [HL]
```
var sum = [HL] + 1

Z = (sum & 0xff) == 0
S = 0
HC = (sum & 0xf) == 0

[HL] = sum & 0xff
```
cycles: 8

### INC r16
```
r16 = (r16 + 1) & 0xffff
```
cycles: 8

### INC SP
```
SP = (SP + 1) & 0xffff
```
cycles: 8

### JP u16
```
PC = u16
```
cycles: 16

### JP cc, u16
```
if (cc)
    JP u16
// The u16 fetch always happens
```
cycles: if cc: 16, else: 12

### JP HL
```
PC = HL
```
cycles: 4

### JR i8
```
PC = (PC + i8) & 0xffff
```
cycles: 12

### JR cc, i8
```
if (cc)
    JR i8
// The i8 fetch always happens
```
cycles: if cc: 12, else: 8

### LD r8, r8
```
r8 = r8
```
cycles: 4

notes for LD r8, r8: the 2 r8s are different ... u knew that right ?

### LD r8, u8
```
r8 = u8
```
cycles: 8

### LD r16, u16
```
r16 = u16
```
cycles: 12

### LD [HL], r8
```
[HL] = r8
```
cycles: 8

### LD [HL], u8
```
[HL] = u8
```
cycles: 8

### LD r8, [HL]
```
r8 = [HL]
```
cycles: 8

### LD [r16], A
```
[r16] = A
```
cycles: 8

### LD [u16], A
```
[u16] = A
```
cycles: 16

### LDIO u8, A
```
[0xff00 | u8] = A
```
cycles: 12

### LDIO C, A
```
[0xff00 | C] = A
```
cycles: 8

### LDIO A, u8
```
A = [0xff00 | u8]
```
cycles: 12

### LDIO A, C
```
A = [0xff00 | C]
```
cycles: 8

### LD A, [r16]
```
A = [r16]
```
cycles: 8

### LD A, [u16]
```
A = [u16]
```
cycles: 16

### LD [HL+], A
```
[HL] = A
HL = (HL + 1) & 0xffff
```
cycles: 8

### LD [HL-], A
```
[HL] = A
HL = (HL - 1) & 0xffff
```
cycles: 8

### LD A, [HL+]
```
A = [HL]
HL = (HL + 1) & 0xffff
```
cycles: 8

### LD A, [HL-]
```
A = [HL]
HL = (HL - 1) & 0xffff
```
cycles: 8

### LD SP, u16
```
SP = u16
```
cycles: 12

### LD u16, SP
```
[SP] = u16 & 0xff
[SP + 1] = u16 >> 8
```
cycles: 20

### LD HL, SP+i8
```
var sum = SP + i8

Z = 0
S = 0
HC = ((SP & 0xf) + (i8 & 0xf)) > 0xf
C = (sum & 0xff) < (SP & 0xff)

HL = sum & 0xffff
```
cycles: 12

### LD SP, HL
```
SP = HL
```
cycles: 8

### NOP
```
// ...
```
cycles: 4

### OR A, r8
```
var res = A | r8

Z = res == 0
S = 0
HC = 0
C = 0

A = res
```
cycles: 4

### OR A, [HL]
```
var res = A | [HL]

Z = res == 0
S = 0
HC = 0
C = 0

A = res
```
cycles: 8

### OR A, u8
```
var res = A | u8

Z = res == 0
S = 0
HC = 0
C = 0

A = res
```
cycles: 8

### POP r16
```
r16 = PopStack ()
```

### POP AF
```
AF = PopStack() & 0xfff0 // 0 out unused bits in F

Z = (F >> 7) & 1
S = (F >> 6) & 1
HC = (F >> 5) & 1
C = (F >> 4) & 1
```
cycles: 12

cycles: 12

### PUSH r16
```
PushStack(r16)
```
cycles: 16

### PUSH AF
```
F = (Z << 7) | (S << 6) | (HC << 5) | (C << 4)
PushStack (AF)
```
cycles: 16

### RES u3, r8
```
r8 &= ~(1 << u3)
```
cycles: 8

### RES u3, [HL]
```
[HL] &= ~(1 << u3)
```
cycles: 16

### RET
```
PC = PopStack()
```
cycles: 16

### RET cc
```
if (cc)
    RET
```
cycles: if cc: 20, else: 8

### RETI
```
RET
IME = 1
```
cycles: 16

notes for RETI: unlike EI, the effect of enabling IME shows immediately. idk y either but it do

### RL r8
```
var res = (r8 << 1) | C

Z = (res & 0xff) == 0
S = 0
HC = 0
C = (r8 >> 7) & 1

r8 = res & 0xff
```
cycles: 8

### RL [HL]
```
var res = ([HL] << 1) | C

Z = (res & 0xff) == 0
S = 0
HC = 0
C = ([HL] >> 7) & 1

[HL] = res & 0xff
```
cycles: 16

### RLA
```
var res = (A << 1) | C

Z = 0
S = 0
HC = 0
C = (A >> 7) & 1

A = res & 0xff
```
cycles: 4

### RLC r8
```
var res = (r8 << 1) | (r8 >> 7)

Z = (res & 0xff) == 0
S = 0
HC = 0
C = (r8 >> 7) & 1

r8 = res & 0xff
```
cycles: 8

### RLC [HL]
```
var res = ([HL] << 1) | ([HL] >> 7)

Z = (res & 0xff) == 0
S = 0
HC = 0
C = ([HL] >> 7) & 1

[HL] = res & 0xff
```
cycles: 16

### RLCA
```
var res = (A << 1) | (A >> 7)

Z = 0
S = 0
HC = 0
C = (A >> 7) & 1

A = res & 0xff
```
cycles: 4

AAAAAAAAAAAAAAAAAAAAAAAAAA

### RR r8
```
var res = (r8 >> 1) | (C << 7)

Z = (res & 0xff) == 0
S = 0
HC = 0
C = r8 & 1

r8 = res & 0xff
```
cycles: 8

### RR [HL]
```
var res = ([HL] >> 1) | (C << 7)

Z = (res & 0xff) == 0
S = 0
HC = 0
C = [HL] & 1

[HL] = res & 0xff
```
cycles: 16

### RRA
```
var res = (A >> 1) | (C << 7)

Z = 0
S = 0
HC = 0
C = A & 1

A = res & 0xff
```
cycles: 4

### RRC r8
```
var res = (r8 >> 1) | (r8 << 7)

Z = (res & 0xff) == 0
S = 0
HC = 0
C = r8 & 1

r8 = res & 0xff
```
cycles: 8

### RRC [HL]
```
var res = ([HL] >> 1) | ([HL] << 7)

Z = (res & 0xff) == 0
S = 0
HC = 0
C = [HL] & 1

[HL] = res & 0xff
```
cycles: 16

### RRCA
```
var res = (A >> 1) | (A << 7)

Z = 0
S = 0
HC = 0
C = A & 1

A = res & 0xff
```
cycles: 4

### SBC A, r8
```
```

## Closure
welp ... i hope this summary was of help in any way !

if you find any mistakes, inconsistensies, or have any feedback, please lemme know !

buh bye <3

## Credits
nectarboy - 2021