# Lecture 13: Bottom Up Parsing

```none
S => \alpha_1 => ... x
```

1. Start reading input left to right
2. If we see the RHS of a rule, replace it with the LHS

```none
Example grammar:

S' -> |- S -|
S -> AyB
A -> ab
A -> cd
B -> z
B -> wx

Example input:

|- abywx -|
```

Two actions at our disposal:

1. Shift: push the next input symbol on to the stack
2. Reduce: pop the RHS of a rule and push the LHS

|Consumed input|Remaining input|Stack|Action|
|-|-|-|-|
|`e`|`|-abywx-|`|`{}`|`shift |-`|
|`|-`|`abywx-|`|`{|-}`|`shift a`|
|`|-a`|`bywx-|`|`{|-, a}`|`shift b`|
|`|-ab`|`ywx-|`|`{|-, a, b}`|`reduce A->ab`|
|`|-ab`|`ywx-|`|`{|-, A}`|`shift y`|
|`|-aby`|`wx-|`|`{|-, A, y}`|`shift w`|
|`|-abyw`|`x-|`|`{|-, A, y,w }`|`shift x`|
|`|-abywx`|`-|`|`{|-, A, y,w, x }`|`reduce B->wx`|
|`|-abywx`|`-|`|`{|-, A, y, B}`|`reduce S -> AyB`|
|`|-abywx`|`-|`|`{|-, S}`|`shift -|`|
|`|-abywx-|`|`e`|`{|-, S, -|}`|`reduce S' -> |- S -|`|
|`|-abywx-|`|`e`|`{S'}`||

* Parse is accepted if remaining input is empty and stack contains just S'.
* Derivation: Rules used in the reduce steps (in reverse)
  * `S'=>|-S-|=>|-AyB-|=>|-Aywx-|=>abywx-|`
* Remember: RIBS - Reduce it Before Shifting
* Bottom up parsing gives rightmost derivation
* Invariant: `stack + remaining input = \alpha_i` 🌟

The key challenge in bottom up parsing is deciding if the stack contents can be reduced. FOrmalizing when the RHS of a rule is on the stack.

```none
Simpler grammar:

S' -> |- E -|
E -> E + T
E -> T
T -> id
```

When do I reduce using `E -> E + T`? When `E + T` is on the top of the stack

Notation for tracking how much of a RHS is on the stack:

* Item: a production/rule with a bookmark somewhere on the RHS of the rule
  * Represented by a dot `*`

```none
E -> *E + T
"fresh item", none of the RHS is on the stack

PUSH E

E -> E*+T
E is on the TOS

PUSH +

E -> E +*T
E and + are on TOS

PUSH T

E -> E + T*
"reducible item", the entire RHS is on the stack
```

* This gives us a DFA!
  * Run the stack content through the DFA
  * Take the action depending on which state you land

Consider:

```none
S' -> *|- E -| ---`|-`--> S' |-*E-|             -> E -> T*
                          E -> *E + T        -> T -> id*
                          E -> *T
                          T -> *id
                            |
                            |
                            v
                        S' -> |-E*-|
                        E -> E*+T       -> E -> E +*T
                            |              T -> *id
                            |                 |
                            v                 v
                        S -> |- E -| *      E -> E + T*
```

![Pic from class](/assets/dfa.JPG)

* Start state is a fresh item for the rule for the start symbol
* Create a transition to a new state on the symbol following the bookmark, advance the bookmark
* If the updated item has a nonterminal right after the bookmark, recursively add fresh items for that nonterminal
* Repeat until no new states are discovered

Item: Run the stack contents through the DFA, if we end up on a state that has a reducible itemm reduce using the rule. Otherwise, shift.

Parse: `|- id + id + id -|`

|Stack|Read input|..|Action|
|-|-|-|-|
|`{}`|`e`|`|-id+id+id-|`|We end up in state 1, not reducible, shift `|-`|
|`{|-}`|`|-`|`id + id + id -|`|We end up in state 2, not reducible, shift `id`|
|`{|-, id}`|`|- id`|`+id+id-|`|We end up in state 6, reduce `T->id`|
|`{|-, T}`|`|- id`|`+id+id-|`|End up in state 5, reduce `E->T`|

* This is not efficient. At each step we run the entire stack content through the DFA
* Observation: shift/reduce only changes the topmost few elements/symbols on the stack
* Let's also keep track of what state the current stack ontents lead to

![Pic from class](/assets/example.jpg)
![Pic from class](/assets/examplep2.jpg)