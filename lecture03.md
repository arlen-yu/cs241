# Lecture 3: Machine Language / Assembly Language

Example: Add 42 to 52, store the result in $4.

Load immediately and skip (list):

```none
lis $d
000000 00000 00000 ddddd 00000 0101100

$d <- MEM[PC]; PC = PC + 4

lis $5            0x00002814
-> .word 42       0x0000002A

lis $7
word 52           0x00003814
add $4 $5 $7      0x00A71820
$31               0x03E00008
```

## Tools

* cs241 wordasm
  * input: lines in hexadecimal (32 bits)
  * output: binary
  * see a1 for usage

Simulator mips.twoints:

`$> mps.twoints <mps file>`

> A1 requires writing machine language in hexadecimal - this is tedius and error prone. Let's make our life simpler and abstract over machine language using what is called *Assembly Language*

Assembly Language: A textual representation of machine language

Assembly programs have to be "assembled" to binary

`cs241.binasm`

* MIPS Assembler
* assembly files `.asm`

## Back to MIPS Instruction Set

Example: Compute the absolute value of $1 into $1

```none
slt: set less than

slt $d, $s, $t
$d <- 1 if $s < $t
otherwise $d <- 0

beq: branch on equal

beq $s, $t, i (negative/positive)
$s == $t, then PC = PC + (i * 4)
```

```none
slt $2, $1, $0
beq $2, $0, 1
sub $1, $0, $1
jr $31

slt $2, $1, $0
bne $2, $0, 1
jr $31
sub $1, $0, $1
js $31
```

```none
unconditional branch: beq $0, $0, __
useful for something like while(true)
```

Example: calculate 13 + 12 + 11 .... + 1 and store result in $3

```none
pseudocode:

n <- 13
sum <- 0
repeat {
  sum += n
  --n
} until n == 0


lis $2
word 13
add $3, $0, $0
add $3, $3, $2
lis $1
word -1
add $2, $2, $1
bne $2, $0, -5 ! not -4 since PC has already incremented
jr $31

there's no need to set $1 to -1 within the loop.... so:

  lis $2
  word 13
  lis $1
  word -1
  add $3 $0 $0
  loop: add $3 $3 $2
        add $2 $2 $1
        bne $2 $0 -3
  jr $31
```

## Assembly Labels (Directives)

* when you define a label it is associated with the line of the instruction at that label
* the assembler computes the offset

```none
(label address - PC) / 4
```

## More

```none
lw: load word

lw $t i($s)
$t <- MEM[$s + i]
```

```none
sw: store word

sw $t i($s)
MEM[$s + i] <- $t
```

```none
Special address 0xffff000C

storing a value to this address will cause the least significan byte to stdout

lis $1
word 0x ffff000C
lis $2
word 67
sw $2, 0($1)        outputs C to stdout
```

```none
Special address 0xffff0004

loading a vlue from this address will read the next keystroke or input from stdin

lis $1
word 0xffff0004
lw $2, 0($1)
```
