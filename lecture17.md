# Lecture 17: Code Generation

```none
--input-->SCANNER---tokens--->PARSER---parse tree---> CONTEXT SENSITIVE A. ----parse tree & sym table --->CODE GENERATION--->tangent language (mips assebly!)
```

## Code Generation

* Optimize
  * Efficiency (running time of compiled program)
* Be correct
* Ease of implementation
* Size of the produced input

```c++
int wain(int a, int b) { // a: $1, b: $2
  return a; // Convention: result in $3
}
```

Equivilant MIPS:

```none
add $3, $1, $0
jr $31
```

```c++
int wain(int a, int b) {
  return b;
}
```

Equivilant MIPS:

```none
add $3, $2, $0
jr $31
```

* The parse tree is the same for both of these programs!
  * Sort of
  * Will need to keep track of where the values of each variables are stored

## Storing values of variables

* Option 1: Reserve a register for each variable in the program
  * Keep track of this within the compiler
  * Can store it as a part of the symbol table

|.|.|.|
|-|-|-|
|a|int|$1|
|b|int|$2|

Does not work in the general case:

```none
WLP4 has 
unlimited #         -------map------>   MIPS has
of variables                            32 registers
```

Ideally, we would use as many registers as we could and have a backup strategy.

* We can use the stack... similification: **just use the stack for all variables**

```none
int wain (int a, int b) { return a; }

lis $4
.word 4
sw $1, -4($30)
sub $30, $30, $4
sw $2, -4($30)
sub $30, $30, $4

; where's the value for a? need to add address to symbol table

;var name | type | offset w.r.t. $30
;   a     | int  |         4
;   b     | int  |         0

lw $3, 4($30) ;load a into $3
add $30, $30, $4
add $30, $30, $4
jr $31
```

**Problem**: Offset depends on the number of variables in the procedure

* Not a huge problem as all variables are declared at the start of the procedure

|.|.|.|
|-|-|-|
|a|int|8|
|b|int|4|
|c|int|0|

**Problem**: We are likely to use the stack for storing other data (like temporary values), this will update register $30

* Compiler will need to update the offsets
* This results in inefficient compiler (compile time increases, not runtime!)

Use another register to keep track of where the params/variables were pushed. Lets use `$29` as a frame pointer.

```c++
int wain (int a, int b) {
  int c = 0;
  retiurn a;
}
```

```none
lis $4
sub $29, $30, $4
sw $1, 0($29)
sw $2, -4($29)
sw $0, -8($29)
sub $30, $30, $4
sub $30, $30, $4
sub $30, $30, $4
lw $3, 0($29)

;cleanupcode
update $30

jr $31
```

|.|.|offset wrt $29|
|-|-|-|
|a|int|0|
|b|int|-4|
|c|int|-8|

$29 is set at the start of the procedure, **then not updated**.

## More complicated programs

```c++
int wain (int a, int b) {
  return a + b;
}
```

Strategy: generate code for a subtree `A->\gamma` by first generating code 
for `\gamma` and then using it to generate code for `A`.

Convention: Value produced by an expression (and subexpressions) will be stored in $3.

## Code for `a+b`

* Generate code for a first, `$3 <-- code(a)`
* Generate code for b, `$3 <-- code(b)` but this overwrites value of a
* Need a temporary register to store "pending computation"

```none
$3 <-- code(a)
add $5, $3, $0
$3 <-- code(b)
add $3, $5, $3

Needed one extra register to hold the temporary
```

```c++
int wain(int a, int b) {
  int c = 0;
  return a - (b - c);
}
```

```none
$3 <-- code(a)
add $5, $3, $0
$3 <-- code(b)
; would be INCORRECT to do the subtraction rn, because of the parenthesis
; cant' do that!!
add $6, $3, $0
$3 <-- code(c)
sub $3, $6, $3
sub $3, $5, $3
```