# Lecture 5: MIPS Assembler

* read in the string, and make sense of it
* translate into binary

## Stages


1. Analysis - make sense of the input, understanding the meaning
2. Synthesis - output equivilant machine instructions in binary

### Analysis

1. Tokenize/scanning/lexical analysis

Convert input string into a sequence of *tokens*. Tokinization is already done for you in A3.

```none
add   $1,    $2,    $3,

ID    REG   REG     REG
        COMMA COMMA   COMMA
```

C++ skeleton:

```c++
class Token {
  std::string kind; // the type of token ID/REG/COMMA
  std::string lexeme; // the actual input characters making up this token
  public:
  // scanner produces a vector<vector<Token>>
}
```

2. Parse: make sense of what each line means

Make sense of what each line means.

* Simple to do for MIPS assebly
* Do it in an ad-hoc way

Depending on the type of instruction, check syntax:

```c++
if (token[0].getKind() == 'ID' && token[0].getlexeme() == 'add') {
}
```

Possible errors:

* incorrect number of registers
* incorrect number of commas
* incorrect range for registers
* unexpected ID/NUM

It is much easier to check for correct syntax rather than all possible incorrect syntax

### Synthesis

1. Produce within the assember the 32 bits representing the instruction
2. Output these bits

* How to deal with labels?

```none
begin: add $1, $2, $3

this makes life a little bit more complicated.
```

Use of label in `beq`/`bne` has to be converted to an instruction offset.

```none
address of where label was defined - PC
---------------------------------------
                  4
```

### Algorithm

* initialize `PC <- 0`
* increment PC by 4 for each instruction and `.word`.

When a label is defined, store the "lexeme" & current PC in a map.

```none
std::map<std::string, int>
```

1. If the label is used in a `beq`/`bne`, use the formula
2. `.word <label>` just output the address whose label was defined

The map is a *Symbol Table*. `label names ----> addresses`.

Possible errors:

* cannot define labels more than once
  * before inserting into a symbol table, check that a mapping does not already exist
* using a label not in the symbol table
  * but!! labels can be dfined after they are used!
  * `beq $1, $0, end .......... end: jr $31`

Solution: write a multi-pass assebler

```none
Pass 1
  - tokenize
  - parse (syntax checks wherever possible)
  - create symbol table
Pass 2
  - check uses of labels
  - generate output
    1. produce the 32-bit representation internall
    2. output the bits
```

### Manipulating bits

```none
Left Bit Shift
x    <<    n
0001 <<   0     0001
0001 <<   1     0010
0001 <<   2     0100

1000 >>   1     1100 one gets padding to the left...

x/2^n

0100
0010
0001
0000
```

```none
Bitwise And (&)

a   b   a & b
0   0     0
0   1     0
1   0     0
1   1     1

Bitwise Or (|)

a   b   a | b
0   0     0
0   1     1
1   0     1
1   1     1
```

Encoding an add instruction:

```none
add $3, $2, $4
    $d  $s  $t

000000 sssss ttttt ddddd 00000 100000

int opcode = 0        00000 ------- 000000000
int funcCode = 32     000 ---------000 100000
int s = 2             000-----------000000010
int t = 4             0----------------000100
int d = 3             0-----------------00011

opcode << 26        now |6 bits|0000000...
s << 21                 |0000||s bits|00....
t << 16                 |000||0...||t bits|.....
d << 11                 |0000||000||0000||dbits|0.....|

funcCode (unshifted)    |0...||0...||0....||0...||0...||funcCode|

int instr = opCode << 26 | s << 21 | t << 16 | d << 11 | funcCode


```
