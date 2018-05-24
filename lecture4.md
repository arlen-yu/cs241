# Lecture 4: More MIPS Assembly, Procedures

`mult $s, $t` - where does the result go?

* The result could be longer than 32 bits
* result is stored in `hi`:`lo`
  * higher 32 bits, lower 32 bits

`multu` - unsigned multiplication

```none
mfhi $d - whatever the value in hi is, move it into $d
mflo $d
```

div/divu for signed and unsigned division

`div $s $t` - $s / $t, and $s % $t

* lo stores the quotient, hi stores the remainder

```none
example:

$1 contains the address of a 32 bit array of ints
$2 contains the number of elements in this array

Read array[3] into $3
I want $1 + (3 x 4)

lis $5
.word 3
lis $4
.word 4
mult $4 $5
mflo $5
add $5 $1 $5
lw $3 0($5)
jr $31

a better way to solve this:

lw $3 12($1)
jr $31
```

## Simulator: `mips.array`

* `$1` - start address of array
* `$2` - # of elements in array

## Procedures in Assembly

### Problems and Concerns

1. The same registers are accessible to all code
    * The procedure called might overwrite
    * Need a way to protect register values

2. How to make a procedure call & return from a procedure

#### Option: Divide registers between procedures

* You might run out of registers
* Think of a recursive function - you're screwed

#### Option: each callee (called procedurer) should garuntee it leaves register values unchanged

* Backup original values of registers
* Where? RAM
  * Reserve and use a chunk of RAM memory in a systematic way
  * MIPS loader helps setup up $30 to just past the last avaliable memory spot

```none
|CODE|
|RAM |
    <--- $30
```

```none
consider f calls g, g calls h

when h is about to return, restore original register value

when g is about to return - restors original register values for f

$30
|g stores values of f registers|
$30
```

Memory is being used as a stack

* callee pushes register values at the start
* callee pops register values just before returning
* `$30` always points to the top of the stack

```none
    |   |
    |$30 - 4   |
$30 |   |
    |used memory   |

sw $8 -4($30)
lis $4
.word -4
add $30 $30 $8
```

### What it looks like

Procedures are created by using a label and then writing the procedure body.

Failed attempts to call a procedure:

```none
    beq $0 $0 foo
back: .....

foo:
----
----
----
beq $0 $0 back

the problem with this: how do you return? you can jump to the label 'back', but this is not a function call. you hard coded the return - no matter where you call it, it will always return the to same place
```

```none
lis $7
.word foo
jr $7

foo:
----
----
----
....
```

Problem: need to remember what the return address of a procedure call is, i.e. where to resume once procedure returns

Solution: `jalr` - Jump And Link Register

`jalr $s`

* stores in $31 the current value of PC (the next instruction)
* updates PC to $s

```none
lis $7
.word foo
jalr $7
.
.
.
.
jr $31 <- doesn't work as intented, it'll take us back to our jalr call!



foo:
----
----
----
jr $31

solution to this: before doing a jalr, store a backup of the current $31, i.e. push $31, then after, pop $31
```

### Parameter passing

* Through registers
* Through memory

Common convention: few parameters, use registers. It's important that procedures are documented with expectations.

```none
* sum1toN
* $1 scratch (should be preserved)
* $2 input number, N (should be preserved)
* $3 output (don't preserve)

sum1toN:
  sw $1 -4($30)
  sw $2 -8($30)

  lis $1
  .word 8
  sub $30 $30 $1

  add $3 $0 $0
  lis $1
  .word -1
  loop: add $3 $3 $2
        add $2 $2 $1
        bne $2 $0 loop
  lis $1
  .word 8
  add $30 $30 $1
  lw $1 -4($30)
  lw $2 -8($30)
  jr $31
```

### Preview: MIPS Assembler

```

input (assembly file) ------ asembler converts ---> output .mips file (binary)
```