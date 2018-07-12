# Lecture 20: Code Gen (cont'd), Optimizations

## Code Gen for new/delete

* Link alloc.merl
* Remember alloc.merl must be specified last
* In generated assembly
  * import init
  * import new
  * import delete

## Procedure init

* Initialized heap memory
* Call once
* `wain`'s 1st param is `int*`, set `$2` to the length of array
* If `wain`'s first param is an int, `$2` must be 0
  * `$2` contains second parameters so store before clearing it

## Procedure new

* Call it like any other procedure
* Number of words to allocate in `$1`
* Return pointer to allocated memory in `$3`
* Return 0 if allocation fails

```none
code(new int[expr])
  = code(expr)
  + "add $1, $3, $0" ; # of expr in $3, move to $1
  + call(new)
  + "bne $3, $0, 1"
  + "add $3, $11, $0" ;so that of new fails, we have a null value
```

## Procedure delete

* Takes a pointer to be deleted in `$1`

```none
code(delete [] expr)
  = code(expr)
  + "beq $3, $11, skipDelete"
  + "add $1, $3, $0"
  + call(delete)
  + "skipDelete:"
```

## Procedures

* Code for `wain` must appear first
  * since assembly programs begin execution at first instruction

```none
- mainline
- wain
- prologue
- | wain |
- epilogue
- f:
  - prologue
  - f body
  - epilogue
- g:
  - prologue
  - g body
  - epilogue
```

* Each procedure is simply a leabel
* Prologue for wain
  * imports
  * register conventions:
    * `$11` -> `
    * `$4` -> 4
  * set `$2` for wain
  * store `$1`, `$2` (optional)
  * Call init

* Epilogue for wain
  * reset `$30` (stack ptr) -> convention for mainline
  * `jr $31`

* Procedure specific prologue
  * No needs for imports and setting the regiester `$11`, `$4` etc.
  * Set frame pointer `$29`
  * Store params and local vars
  * Preserving callers register values

* Procedure specific epilogue
  * Restore caller's registers
  * Restore `$29`? `$30`?
  * `jr $31`

* Store/Restore registers
  * Which ones?
    * All of them? Correct, inefficient
    * All those that are used
  * By  whom?

## Caller-saves vs Callee-saves

* Suppose f calls g
* Caller-saves: f saves registers it still needs
* Callee-saves: g saves registers it overwrites
* Previous approach: caller saves `$31` (no choice)
* Who saves `$29`? Both caller/callee saves are possible

## Callee Saves

* f calls g
* g has the following tasks (prologue)
  * Store f's $29
  * Store f's registers that we overwrite
  * Set own $29
* In which order?
* Convention: `$29` points in the first free word on g's prologue, the stack when the procedure is called:

```none
sw $29, -4($30)
sub $30, $30, $4
add $24, $30, $0
```

## Caller Saves

```none
push($29)
push($31)
call
pop($31)
pop($29)
```

## Parameters


* \# of parameters could exceed # of free registers
  * Need a more general solution
  * Aka just use the stack

```none
code(factor -> ID(expr1, expr2, ... exprn))
  = code(expr1) + push($3) + code(expr2) + push($3)
    + push($29) + push($31)
    + ... + code(exprn) + push($3)
    + "lis $9" + ".word ID.lexeme"
    + pop($31)  + pop($29) + pop n times (the arguments)
```

```none
code(procedure -> INT ID params.....)
  = "ID.lexeme:"
  + "sub $29, $30, $4" ; set the frame pointer
  + store caller's registers (assuming callee saves)
  -> code(decls)
  -> code(statements)
  -> code(expr)
  + restore registers
  + "add $30, $29, $4" + "jr $31"
```
