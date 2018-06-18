# Lecture 8: Regular Languages (cont'd)

## Last time

* Regular Expressions
* DFA
* NFA

An NFA/DFA has 5 components

```none
Σ - alphabet
Ø - set of states
q_0 - initial state
A - set of accept states
Š - transitional function

DFA: Š: Ø x Σ -> Ø
NFA: Š: Ø x Σ -> P(Ø)
```

## NFA Recognition

* Input: NFA(Σ, Ø, q_0, A, Š)
* Output: yes/no is `w \in L(NFA)`?

```none
states <- { q_0 }
for i in i to n
  states <- U (s \in states) Š(s, a_i)
end for

if states and a != 0
  return true
else
  return false
```

A language is regular if you can write an NFA for it. Every NFA can be written as a DFA.

## Subset Construction

```none
L = { cab } U { strings over { a, b, c} with even number of a's }

      [2] --a-->[3] --b-->[[4]]
    c/
[[1]]----b, c----->[[6]]<--->b, c
    a\          a___ />
      -----[5]</
            ^
            |b, c
            v

input: caba

Read        Not Read            States
ê             caba                {1}
c             aba                 {2, 6}
ca            ba                  {3, 5}
cab           a                   {4, 5}
caba          empty               {6}

State 6 is an accepting state - word is in the language!
```

```none
Create a DFA state for each set of NFA states that we could be at having seen some part of the input

      ---{5}
    a/
    /
-[1]--b--->{6}
  \
  c\------->{2, 6}

     -----{5}<-->b, c
  a /      a\\
-[2]----b---{6} <--b, c-->
  \
   ------c--- {2, 6}
```

* **e-transitions**
  * A free pass sto the next state
  * Does not consume an input symbol
* **e-closure**
  * A set of states you can reach using 0 or more e-transitions

## Kleene's Theorem

A language is regular if there exists an RE, DFA, NFA, ÊNFA for the language.

## Informal RE => ÊNFA

```none
ø           ->[]
ê           ->[[]]
a           ->[]--a-->[[]]
```

## Recognition vs Scanning

* Recognition - yes/no, `is w \in L`
* Scanning - given an input string, can you break the string into substring where each substring is a word in a language
  * Input: string s
  * Output: tokens w1, w2, ... wn such that
    * s = w1w2...wn
    * `wi \in L`

```none
Start with a dFA that recognizes a single word from L

    <----e/omit token-------[[]]
-->[]
    <----e/omit token--------[[]]
```

## Making tokenization unique

* Be greedy
* Use up as much of the input as possible
  * Delay taking the epsilon transition
* input: `aaaa`
  * Tokenization: emit `aaa` followed by `ERROR`

## Maximum Munch Algorithm

* Run the DFA for L
* If stuck at accepting state, continue. Otherwise, backtrack the DFA and input to last seen accept state
* Output a token corresponding to accept state
* Reset to start state

*Gap in notes*.

