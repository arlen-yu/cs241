# Lecture 12: Top Down Parsing, Bottom Up Parsing

## Last Time

```none
Product(A, a) = { A -> B | a \in First(\beta) } U { A -> B | Nullable(\beta). \alpha \in Follow(A) }
```

```none
result = {}
for i in 1 to n
  if Y_i \in N (non-terminal)
    result U= First(Y_i)
    if not nullable[Y_i] break
  else // Y_i is a terminal
    result U= { Y_i }
    break
return result
```

## Computing Follow

```none
Follow(A) = { a | s' =>* \alphaAa\beta }
initialize Follow[A] = {}, \forall A \in N\{S'}
repeat
  for each rule A -> B_1 ... B_k
    for i 1 to n
      if B_i \in N
        follow[B_i] U= FIrst*(B_i+1, ... B_n)
        if all of B_i+1, ... B_n are nullable (including i = n)
          Follow[B_i] U= Follow(A)
repeat until nothing changes
```

Consider:

```none
1. S' -> |- S -|
2. S -> bSd
3. S -> pSq
4. S -> c
5. S -> le
6. c -> ê
```

|Iteration|0|1|2|
|-|-|-|-|
|S|`{}`|`{-|, d, q}`|`{-|, d, q}`|
|C|`{}`|`{-|, d, q}`|`{-|, d, q}`|

||`|-`|`-|`|b|d|p|q|l|
|-|-|-|-|-|-|-|-|
|S'|1|||||||
|S||4|2|4|3|4|4|
|C||6||6||6|5|

## LL(1) Parser

* Left to right scan (L)
* Leftmost derivation (L)
* 1 is the number of symbols we're looking ahead (1)
* A grammar is `LL(1)` if no combination of non-terminal and terminals contains more than 1 rule in the predict table
* Example exam question: Is this parser LL(1)? Explain your answer.
* We can also have `LL(k)` parsers.

Consider:

```none
expr -> id
expr -> expr op expr
op -> + | - | * | /
```

Let's compute `Predict[expr][id] = ?`

```none
expr -> id
  id \in First{id} is true, meaning the rule is in the entry ✔️

expr -> expr op expr
  id \in First{expr op expr}
  id \in First{expr}
    First{expr} \subset {id} ✔️

NOT LL(1)
```

Tips for finding LL(1):

* A nullable symbol should *not* have the same terminal in its first and follow set.

```none
S -> AB
A -> ë
A -> a
B -> a

Not LL(1), Predict[A][a] = {A -> ë, A -> a}
```

* There should be only one way to derive ë for a nullable symbol

```none
S -> AB
A -> ë
A -> C
C -> ë
B -> a

S => AB => B => a
S => AB => CB => B => a

Problem: Predict[A][a] contains both rules - NOT LL(1)
```

* A non-terminal should not generate a terminal though 2 different productions/rules

```none
A really good example 🌟
E for expr, T for term

E -> E + T | T
T -> T * F | F
F -> a | b | c ...

This is the unambigious grammar with higher precedence for *.

E => T => F => a
E => E + T => T + T => F + T => a + T
```

## In fact, left recursive grammars are **never** LL(1)

Let's use a right recursive grammar:

```none
E -> T + E | T
T -> F & T | F
F -> a | b | c ...
E => T + E => F + E => a + E
E => T => F => a
```

Factor our the common prefix for each rule

```none
E -> TE'
E' -> + E | ë
T -> FT'
T' -> *T | ë
F -> a | b | c ...


E => TE'
  => T
  => FT'
  => F
  => a

```