# Lecture 21: Code Gen (last notes), Compiler Optimizations

What if wlp4 code contains procedure: `int init() { ... }`

* Label for procedure would clash with the import init
* Need to come up with a unique label for this procedure
  * Prepend a string/character to each procedure's name
  * For init call the label `finit` (marmo tests probs)

## Compiler Optimizations

* Huge field of research
* Typically the goal is to reduce running time of compiled code
* CS241 - reduce the size of the compiled code
* Key requirement: the behavior of the program should be unchanged

### Constant Folding

Expression `1 + 2`

```mips
lis $3
.word 1

sw $3, -4($30)
sub $30, $30, $4
lis $3
.word 2

lw $8, 0($30)
add $30, $30, $4
add $3, $8, $3
```

* 9 Instructions, 2 words

### Constant Propagation

```c++
int x = 1;
return x + x;
```

* Assume x is at offset = 12.

```mips
lis $3
.word 1
sw $3, -12($29)

lw $3, -12($29) ; left operand
push($3)

lw $3 -12($29) ; right operand
pop($8)

add $3, $8, $3
```

* 10 instructions/words

If the compiler can determine that variable has a constant value (at compile time) then we can replace the use of the variable with the constant value. (i.e. the problem becomes 1 + 1 = 2).

* If a variable is never used don't definie/initialize it

### Common Sub-expression Elimination

* Consider expression `x + x`

```mips
lis $3, -12($29) ; left operand
add $3, $3, $3
; this is not constant propogation
; does not require that x's value be known
```

* But you need to be careful with `foo() + foo()`, what if `foo()` has side effects? If you optimize with sub-expr elimination you change the behavior.

```mips
; consider (a + b) * (a + b)
mult $3, $3
mflo $3
; just write a function to compare the two branches of the tree and see if they're identical
```

```mips
; consider a = x * y ... b = x * y;
; could do b = a if a is unchanged... tricky.
```

### Dead Code Elimination

* If there's code that is garaunteed to not run, don't generate it

```c++
int x = 10;
int  (x < 20) {
  // you know this is always executed
} else {
  // you know you don't have to generate this
}
```

* This doesn't really help with compile time, only size
* No need to generate code for procedures that are never called

### Register Allocation

* Using the stack to store params and variables is inneficient for both the time and the size of the program
* Registers are quicker, require no other instructions
* Registers 14 - 28 are currently unused in our coding strategy

Which variables should get dedicated registers?

* If you're optimizing for running time, give priority to variables in loops

### Strenght Reduction

* Running time optimization
  * multiplication costs much more than addition
    * Instead of multiplying by 2, add the value by itself!

### Peephole Optimizations

* Works on generated output MIPS assembly
* Instead of outputting to stdout, output to a data structure
* Can analyze the generated assembly
  * Peephole: look at x # of instructions at a time

Suppose:

* You see a `push($3)`
* Then  some instructions (X)
* You see a `pop($8)`

If the code at (X) does not use `$8`, the push and pop are unecessary! Instead:

```mips
add $8, $3, $0
; then some other instructions (X)
```

### Procedure Specific Optimizations

#### Method Inlining

```c++
int  f(int x) {
  return x + x;
}

int wain (int a, int b) {
  return f(a);
}
```

* Replace the call to f wit hthe body of f
* For size optimizations:
  * Method inlining only makes sense if the size of the method is less than the code needed to call the method
  