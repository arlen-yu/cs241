# Lecture 19: Code Gen (continued)

Last time:

```none
code(statement -> IF test statement, else statement)
  = code(test) +
    "beq $3, $0, else"
    + code(statement1)
    + "beq $0, $0, endif"
    + "else"
    + code(statement2)
    + "endif"
```

```none
code(statement -> while test statement)
  = "start:"
    + code(test)
    + "beq $3, $0, endloop"
    + code(statements)
    + "beq $0, $0, start"
    + "endloop:"
```

## Pointers

* Allocate an array using new
* Take the address of a variable

```none
int wain(int a, int b) {
  int *x = NULL;
  int y = z;
  x = &y;
  return *x;
}
```

Support:

* NULL
* Dereference
* Address-of
* Comparisons
* Pointer arithmetic
* Allocation
* Deallocation

```none
code(factor -> NULL)
  = "add $3, $0, $0"
```

* We could make Null have the value 0
* We would like dereferecing NULL to crash our program
  * Choose an address not divisible by 4

```none
code(factor -> NULL)
  = "add $3, $11, $0" (recall $11 contains 1)
```

```none
code(factor -> * expr) gaurunteed to have type int*
  = code(expr) ; $3 contains the address to dereference
  + "lw $3, 0($3)"
```

## Pointer Comparisons

Consider `code(test -> expr1 < expr2)`. We need to know whether its comparison for ints or pointers, since pointers comparison is unsigned. Use the `typeOf` function you wrote in A7.

```none
code(test -> expr1 < expr2)
  = code(expr1)
  + "add $5, $3, $0"
  + code(expr2)
  // if int comparison:
  + "slt $3, $5, $3"
  // else if ptr comparisons
  + "sltu $3, $5, $3
```

## Pointer Arithmetic

```none
expr1 -> expr2 + term
        int + int
        int* + int --> this operation adds 4 x int value{1}
        int + int*

{1}: consider some array a, *(a+2) will go forward 2 * 4 = 8
```

```none
code(expr1 -> expr2 + term) ; int* + int case
  = code(expr2)
  + push($3) ; need to use stack since you don't know whats in term{1}
  + code(term)
  + "mult $3, $4"
  + "mflo $3"
  + pop($8) ; register 8 contains int*, register 3 contains 4 * int
  + "add $3, $8, $3"


{1}: term could use way more than 32 registers
```

```none
code(expr -> expr2 - term)
        int - int ✔️
        int* - int ✔️
        int - int* ❌
        int* - int* ✔️
  = code(expr2)
    + push($3)
    + code(term)
    + pop($8)
    + "sub $3, $8, $3"
    + div $3, $4
    + mflo $3
```

```none
code(statement -> lvalue becomes expr semi)
```

An lvalue has 3 rules

1. lvalue -> ID e.g. a = 5
2. lvalue -> ( lecture ) recurse on the inner lvalue
3. lvalue -> STAR factor, e.g. *(a+2) = 5

```none
code(statement -> STAR factor becomes expr)
  = code(expr)
  + push($3)
  + code(factor) ; garunteed $3 is the address to store at
  + pop($8)
  + sw $8, 0($3)
```

```none
code(factor -> AMP lvalue)
```

3 rules for lvalue:

1. lvalue -> (lvalue) recurse
2. lvalue -> ID e.g. &a

* Note: variables whose address is taken must reside in memory **not registers**.

```none
code(factor -> & ID)
  = "lis $3"
  + ".word offset" (from the symbol table)
  + "add $3, $29, $3"
```

3. lvalue -> STAR factor e.g. &* factor

```none
code(factor -> &*factor2)
  = code(factor2)
```

new/delete

* need a way to represent the heap

Good news: we have provided alloc.merl exports init/new/delete procedures. The main prologue should input these procedures.

```none
./wlp3gen < in.wlp4i > out.asm
cs241.linkasm < out.asm > out.merl
cs241.linker out.merl print.merl alloc.merl > final.merl  <--- THE ORDER MATTERS
cs241.merl 0 < final.merl > final.mips
```