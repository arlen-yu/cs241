# Lecture 7: Regular Languages

* Built using finite languages
* Allow union
  * L1 U L2 = { x | x \in L1, x \in L2 }
  * L1 = { dog, cat }, L2 = { fish, ë }, L1L2 = { dogfish, dog, catfish, cat }
* Allow repitition
  * L* = { ë } U { xy | x \in L*, y \in L}
  * L = {a, b}, L* = { ë, a, b, aa, ab, ba, bb }

## Regular Expressions

* One way of specifying a regular language

```none
ø no word
ë empty word
a (a \in \Sigma) word containing symbol a
```

*Concatenation*: R1R2 (r1 + r2 are regular expressions) :: words matched by R1 followed by word matched by R2

*Union/choice/alternation* R1|R2 :: words matched by R1 or R2

*Repitition* R* :: 0 or more repititions of words matched by R

Regular expressions are a convenient shorthand for specifying regular languages. Not very convinient for recognition. Another way to specify/recognize Regular language sis to use DFA.

## Deterministic Finite Automata

```none
                        ------------------------------ b loops back
                    a  |                              |
-> x (start state) --> Y (accepted/final state) ------
```

* Transition from state X to Y on symbol a Š(X, a) = Y
* Transition from state Y to Y on symbol b Š(Y, b) = Y
* L = { a, ab, abb, abbb, ... } infinite language!

### Example

```none
Σ = {a} L = { even number of as }

In RE: (aa)*

---> [[ ]] ----a---->
      |            [ ]
      <-----a------

```

```none
Σ = {0, 1}
L = { words of even length }

---> [] <----0---> []
     ^              ^
     |              |
     1              1
     |              |
     v              v
     []<----0---->[[]]


     another way

---->[[even]] <----0, 1---->[odd]
```

Regular Languages are often used in the scanning phase

```none
Σ = { a, b, ... z, 0, 1, ... 9, : }
L = { valid MIPS labels }

                              <-------- a-z, A-Z, 0-9
-->[] -----a - z, A - Z ---->[]-------| ----- : ------->[[]]
```

```none
Σ = { $, 0, 1, ... 9 }
L = { MIPS registers }

                                    <----- 0-9
--->[] -- $ --> [] --- 0-9 ---> [[]]-----|

Some problems: recognizes $00, $1234567, need to restrict somehow:


--->[] -- $ --> [] --- 0-9 ---> [[]] --- 0-9 ---> [[]]

But: allows $99, $00. Instead:

--->[]--$-->[] --- 0, 4-9 ---> [[]]
            |------1, 2 ---> [[]] ---- 0-9 ---> [[]]
            |----- 3 -------> [[]] ---- 0, 1 ----> [[]]
```

## Formal Definition of DFAs

A DFA has 5 components (Σ, Ø, q_0, A, Š):

* Σ - alphabet
* Ø - set of states
* q_0 - unique start state
* A - A \in Ø, set of accepting states
* Š: Ø x Σ -> Ø

## DFA recognition algorithm

* input DFA `(Σ, Ø, q_0, A, Š)`
* input word `w = a_1a_2...a_n`
* output: `is w \in L(DFA) ?`

```none
state <--- q_0
for i in 1 to n:
  state <-- Š(state, a_i)
end for

if state \in A
  return true
else
  return false

NOTE: we assume the presence of a hidden/implicit error state. I.e. if a trnasition is not defined on a given symbol, we transition to this error state and stay there
```

>Kleene's Theorum: A language is regular if there exists a DFA specifying it.

So if you're asked to prove that a language is regular, you can draw a DFA for it.

## DFA's with actions (Finite Transducers)

```none
Σ = { 0, 1 }
L = { binary numbers with no leading 0s } = {0, 1, 10, 11, ...}

      --- 0 -> [[]]
--> []            <-----
      --- 1 --> [[]] ---| 1
                 ^ |
                 | | 0
                 | |
                 ---

We can associate actions/code to a transition in a DFA

      --- 0/n=0 -> [[]]
--> []            <-----
      --- 1/n=1 --> [[]] ---| 1/n=2n+1
                    ^ |
                    | | 0/n=2n
                    | |
                    ---
```

```none
Decimal integers with no leading zeros:
      ---0--->[[]]
--->[]              <--------
      --- 1-9--->[[]] ------| 0-9

Hexidecimal integers prefixed with 0x

                                              <------- 0-9
---->[]---0--->[]---x--->[]--- 0-9, A-F --->[[]] ----| A-F

If you combine the two, you get an NFA (no longer deterministic since you have two choices with 0)
```

## NFA

* Non-deterministic Finite Automata
* We are going to look for *any* path to accept state
* Keep following *all* paths until the end of input

### Formal Definition of NFA

* NFA has 5 components (Σ, Ø, q_0, A, Ś) where Ś is the component that is different
* Ś: Ø x Σ ---> P(Ø) (set of states)

