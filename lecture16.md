# Lecture 16: Context Sensitive Analysis Type Checking

The typeof the LHS if the typeof the RHS

```none
E : T
------
(E) : T
```

* `&3x`
  * we can only take the address of an `lvalue`
    * enforced by the WLP4 CFG

## Dereference

```none
E: int *
--------
*E : int
```

## Allocation

```none
E: int
-----
new int[E]: int *
```

* The expression `new int[E]` has type `int *`, if `E` has type `int`

```none
          expr
        /   |   \
```

![June 26](/assets/june26.JPG)

## Subtraction

```none
E1: int E2: int
---------------
E1 - E2: int


E1: int* E2: int
----------------
E1 - E2: int*

E1: int* E2: int*
-----------------
E1 - E2: int  the number of numbers in between the range
```

## Procedure Calls

```none
f(T1, T2, ...Tn) in top level symbol table, check typof Ei = Ti for all i
----------------------------------------------------------------------------
f(E1, E2, ... En): int
```

## Type Checking Statements

* Expressions produce values and have a type
* Statements must be "well-typed"

```none
E: T
------
well-typed(E)
```

* An expression is well-typed if a type can be inferred from the expression

## Comparisons

* E.g. `E1 == E2`
* Comparisons in the WLP4 can only appear in a test (if, while)
* Comparisons are well typed if the subexpressions have the same type

```none
E1: T   T2: T
-------------
Well Typed(E1===E2)
```

## Assignment

```none
E1:T      E2:T
---------------
well-typed(E1==E2)
```

* In an assignment statement, the RHS can be any well typed expression
* The LHS must also be an lvalue, lvalues represent a location i nmemory
* Variables are lvalues

```none
int *a = NULL;
a = new int[10]l
*a = 10;
```

* A dereferenced pointer is also an lvalue
* The WLP4 CFG enforces that the LHS is an lvalue

## Print Statement

```none
E:int
----------
well-typed(println E)
```

## Deallocation

```none
E: int*
well-typed(delete[]E)
```

## Statement Sequence

* A sequence of 0 statements is well-typed

```none
well-typed(S1) well-typed(S2)
------------------------------
well-typed(S1S2)

well-typed(T) well-typed(S)
----------------------------
well-typed(while(T) { S })
```