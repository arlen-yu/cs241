# Lecture 11: Top Down Parsing

```none
S => \alpha_1 => \alpha_2 => ... => x

Example Grammar
s` -> |- S -|
S -> AyB
A -> ab
A -> cd
B -> z
B -> wx

input: |- abywx -|
```

1. Look at the current `\alpha_1` one symbol at a time left to right. While the symb ols are terminals, match to the input.
2. When we hit the first non-terminal. choose a rule to apply.
3. Replace the LHS with the RHS of the rule to generate `\alpha_{i+1}`

```none
S` => |- S -| => |- AyB -|
```

We reached a non-terminal `A`. Look ahead to the next yet to be matched input symbol.

```none
Predict(A, a), i.e. Predict[A][a]
```

This will predict which rule to apply based on a non-terminal in a look ahead terminal.

```none
Say it returns A -> ab (step 2)

Step 3:
... |- AyB -| => |-abyB -|
```

We reached another non-terminal `B`. Lookahead `w`. Get `Predict[B][w] = B -> wx`.

```none
S` => |- S -| => |- AyB -| => |- abyB -| => |- abywx -|
```

Reached the end of input and haven't reached any non-terminals -- parsing is done!

## Optimizing our algorithm

An `\alpha_i` looks like `xA\beta` where `x` is a string of terminals, `A` is the leftmost non-terminal. Once `x` has been matched, discard it, i.e. only keep track of `A\beta`.

Another optimization we can make is to use a stack to keep track of `\alpha_i`, i.e. keep `A\beta` on the stack.

```none
|     |
|A    |
|\beta|
```

| Consumed Input|Remaining Input|Stack|Notes|
| ------------- |-------------|-----|---|
|e|`|-abywx-|`|`|-, s, -|`||
|`|-`|`|-abywx-|`|`S, -|`| While the top of the stack is a non-terminal, match with input, pop `-|`|
|`|-`|`abywx-|`|`S, -|`|Predict[S][a] = S -> AyB, Pop S, push B, push y, push A|
|`|-`|`abywx-|`|`a, b, y, B, -|`| Predict[A][a] = A -> ab, pop A, push ab (reversed)|
|`|-a`|`bywx-|`|`b,y,B,-|`| pop a|
|`|-ab`|`ywx-|`|`y, B, -|`| pop b|
|`|-aby`|`wx-|`| `B, -|`|pop y|
|`|- aby`|`wx`|`w, x, -|`|Predict[B]{w] = B-> wx, pop B, push wx (reversed|
|`|-abyw`|`x-|`|`x, -|`| pop w|
|`|-abywx`|`-|`|`-|`|popx|
|`|-abywx-|`|`e`|`e`|pop `-|`|

## Top Down Parsing Invariant

```none
Consumed Input + Stack Contents = \alpha_i
```

I.e. the consumed input combined with the stack contents is a step in the derivation.

## When does parsing fail

1. `Predict[A][a]` contains 0 rules or more than 1 rule
2. Top of the stack is a terminal which does not match the leftmost symbol from the remaining input

## Computing the Predict Table

`Predict[A][a]` is computing the rules that are applicable when the top of the stack is `A` and the lookahead is `a`. Should contain rules of the form `A->\beta` such that `\beta` derives somethign that begins with `a`.

```none
Predict[A][a] = { A -> \beta | a \in First(\beta) }
```

```none
First(\beta) = { a | \beta =>* aX }
Note that it's a set of terminals

Predict(A, a) = { A -> \beta | \beta =>*aX }
```

But this does not account for when `\beta` derives the empty string, i.e. `B =>* e` so `A =>* e`. Updated definition:

```none
Predict(A, a) = { A -> \beta | a \in First(\beta) } U { A -> B | Nullable(\beta), a \in Follow(A) }
```

```none
Follow(A) = { a | S' =>* \alphaAa\beta }
```

```none
Nullable(\beta) = true if B =>* e, false otherwise
```

## Computing Nullable

```none
initialize Nullable[A] = false \forall A \in Non Terminals
repeat
  for each rate B -> B_1, ... B_k
      if k = 0 or Nullable[B_1] = Nullable[B_2] = ... Nullable[B_k] = true
        Nullable[B] = true
until nothing changes
```

Example:

```none
Grammar:

S' -> |- S -|
S -> bSd
S -> pSq
S -> C
C -> \alphaC
C -> ë
```

||Iteration 0|Iteration 1|Iteration 2|Iteration 3|
|--|--|--|--|--|
|S'|false|false|false|false|
|S|false|false|true|true|
|C|false|true|true|true|

## Computing First(A)

```none
First(A) = { a | A=>*a\gamma}

initialize First[A] = {} \forall A \in N (non terminal)
repeat:
  for each rule A -> B_1,...B_k
    for i = 1...k
       if B_i is a terminal "a":
        First[a] U= {a}
        break
      else // B_i not a terminal
        FIrst[A] U= First[B_i]
        if not nullable[B_i] break
until nothing changes
```