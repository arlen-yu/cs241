# Lecture 2

```none
a - b
a + (-b)
a + (N - b)
a + (N - 1 - b) + 1
```

2's complement can be computed by flipping the bits and adding 1

## Converting out of a 2's complement value

* a negative value will have most significant bit set to 1

```none
          11111 1011

flip:     0000 0100
add 1:    0000 0101

since we know it was negative, it's -5 in decimal
```

### Approach 2

* treat the number as an unsigned binary
* if the first bit was 0, done
* if not, subtract 2^n i.e. N

```none
0111 < 1001 ?
7 < 9 true
7 < -7 false

Requires signed/unsigned comparisons
```

```none
1000011 (67)

ASCII Encoding
- could be upper case C
- 7 bit encoding
- unicode
```

## Machine Language

* set of machine instructions

Machine Instructions: Sequence of bits that have exactly one meaning on a partition architecture

### MIPS 32

* 32 bit instructions

### Control Unit

* fetch/execute instructions
* timing unit
* communicating with peripherals

### ALU: Arithmetic logic unit

* operations/comparisons

### Memory

* registers -> fast
* RAM -> slower

### 32 General Purpose Registers

* 32 bit registers
* need 5 bits to mention any register
  * 2 ^ 5 = 32
* $0 always contains 0
* $30, $31 (addresses to RAM)

### RAM

* much slower than registers
* a lot more memory than registers
* visualize as a byte array
* often we will use addresses that are multiples of 4
* operations can only be performed on values in register
  * might need to load data from RAM
  * might need to store data back to RAM
* A program has a code section & other data sections

### How does the processor know which part of memory is code

Short answer: it doesn't.

### Special Register: *PC (Program Counter)*

* PC is 32 bits
* contains the address of the next instruction to execute
* a program called the loader loads the program in memory and sets PC to the start address of where the code is
* convention: program is loaded at start address 0, then `PC <- 0`

### Fetch/Execute Cycle

* implemented in hardware

```none
PC <- 0
loop {
  IR <- MEM[PC]     fetches 32 bit into Instruction Register (IR)
  PC = PC + 4
  decode            what is the instruction we just fetched
  excecute
}
```

* how does a program terminate?
  * the loader places the "return address" into $31
  * to stop a program must jump to this return address

### Machine Instructions

* 18 instructions
* 32 bits

#### Add instruction

`$d = $s + $t` is `add $d, $s, $t`

```none
Add value in $5 with value in $7 and store result in $3

add $3 $5 $7

from MIPS reference sheet:
              $3 in binary
000000 00101 00011 00000 1000000
    $5 in binary

    convert to nibbles:

0000|00   00|101   0|0111  0001|1   000|00   10|0000

0x00A71820
```

```none
copy value stored in $3 to $8

add $8 $3 $0

000000 00011 000000 01000 00000 100000

.... 0x00604020

clear a register: add $3 $0 $0
```

#### JR Instruction

`jr $s`, format is `000000 sssss 00000 00000 00000 001000`

* sets PC to $s
* i.e. take the value in $s as an address
* recall, PC is set to the next instruction to execute
* to end, `jr $31`

```none
000000 11111 00000 00000 00000 001000

0x03E00008
```

#### Sub

```none
sub $d, $s, $t means $d = $s - $t

000000 sssss ttttt ddddd 00000 100010

sub $3 $0 $1 negates the value in register 1, puts the value in register 3
```
