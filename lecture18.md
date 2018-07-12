# Lecture 18: Code Gen (continued)

## Generating code for expressions

```none
a - ( b - c )

code(a) ; $3 <-- a
add $5, $3, $0 ; $5 <-- a
code(b) ; $3 <-- b
add $6, $3, $0 ; $6 <-- b
code(c); $3 <-- c
sub $3, $6, $3 ; #3 <-- b - c
sub $3, $5, $3 ; $3 <-- a - (b - c )
```

## Use the stack instead

```none
code(a) ; $3 <-- a
push($3)
code(b) ; $3 <-- b
push($3)
code(c) ; $3 <-- c
pop($8) ; $8 <-- b
sub $3, $8, $3 ; b - c
pop ($8) ; $8 <-- a
```

## Coding Templates

* `code(factor -> ID) = lw $3, offset($29)`

```none
code(expr_1 -> expr_2 + term)
  = code(expr_2)
  + push($3)
  + code(term)
  + pop($8)
  + "add $3, $5, $3"
```

## PrintLn Statement

Semantics prints the int value in decimal followed by a new line. A2P6.A2P7 we write a print procedure.

## Runetime Environment: function/procedures/classes mode

Available by the compiler/OS for use.

* Our codegen will assume that a print procedure is avalaible at runtime
* Generate code to call this procedure
* Since the "print" label will be defined evetually we need to tell the assembler that we intent to use this external label
* Use the import assembler directive import print

## Procedure File print.merl (object file)

![July 3](/assets/july3.JPG)

* Print expects the value in $1
* Calling a procedure (using jalr) overwrite $31 (will need to preserve this)
* Store/restore registers we care about

```none
code(println expr)
  = code(expr)
  + "add $1, $3, $0"
  + push($31)
  + storeRegs()
  + "lis $10"
  + ".word print"
  + "jalr $10"
  + restoreRegs()
  + pop($31)
```

