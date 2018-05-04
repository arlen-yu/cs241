# Lecture 1

## What is a sequential program

* Not concurrent/parallel (single threaded)

### Goal: understand how sequential programs work

* Describe the relationship between high-level programming languages and machine level data representations
  * Different abstractions

```none
C, C++, Racket, Scala, Java -----
            |                   |
            v                   |
    Assembly Language           | [compiler]
            |                   |
            v                   |
    Machine Language  <---------|
```

* You will get to write programs that read programs and output programs

## Data Representations

### Terminology

* `bit` - a single digit in binary, can be `0`, `1`
* `byte` - a sequence of 8 bits
* `word` - machine specific, dependent sequence of bits, on a 32 bit machine, 32 bits make a word
* `nibble` - 4 bits

```none
Example string 10000011
-> -3 signed binary
-> one million & eleven
-> 67 unsigned binary
-> -61 2's complement number
-> character C
-> garbage
```

### Decimal Representation

```none
1  0   0   0   0   1   1

10^6  10^4    10^2    10^0
  10^5    10^3    10^1
```

### Binary Representation

```none
1   0   0   0   0   1   1
2^6   2^4       2^2     2^0
  2^5      2^3      2^1
```

### Converting from decimal to binary

```none
67 = 64 + 2 + 1
   = 1x2^6 + 1x2^1 + 1x2^0

For binary representation, just keep dividing by 2 and keep track of the remainder
```

### Hex Representation

```none
123456789ABCDEF

Write the decimal value 42 in base 16:

42 = 32 + 10
   = 2x16^1 + 10
   = HEX 2A

Opposite direction:

F4A = Fx16^2 + 4x16^1 + Ax16^0
    = 15x16^2 _ 4x16^1 + 10x1
    = 3914
```

```none
Possible 32 bit machine instruction
1100  1010  1101  0000  0100  1011  0010  0001

 12    10   13    0     4       11    2    1
 C    A     D     0     4       B     2    1

 0xCAD04B21
```

### Numeric Ranges

Given n bits, what is the range of numbers we can represent? `2^n`

```none
n = 3, 2^3 = 8

    unsigned binary   sign magnitude
000      0                  +0
001      1                  +1
010      2                  +2
011      3                  +3
100      4                  -0
101      5                  -1
110      6                  -2
111      7                  -3
```

In general, for n bits, the unsigned binary range is 0 to 2^n - 1

#### How to represent negative numbers

Idea: reserve a bit to represent the sign, and use the reset to represent the magnitude (called the **sign & magnitude** representation).

Problem: two representations for 0.