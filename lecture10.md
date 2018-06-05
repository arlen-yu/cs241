# Lecture 10: Context Free Grammars (cont'd)

## Example

```none
N = { A B, C }

Σ, T = { a, b, c, d, e, f, g, h }

Productions
A -> BgC
B -> ab
B -> cd

C -> n
C -> ef

Start Symbol: A
```

Parse Tree: Structure for easy traversal of the parsed input

**Property**: A derivation uniquely defines a parse tree

```none
expr -> expr op expr
     -> expr op expr op expr
     -> id op expr op expr
     -> id - expr op expr
     -> id - id op expr
     -> id - id * expor
     -> id - id * id
```

**Property**: An input string can have more than one parse tree

```none
expr -> expr op expr
     -> id op expr
     -> id - expr
     -> id - expr of expr
     -> id - id op expr
     -> id - id * expr
     -> id - id * id
```

Same input string can lead to two different parse trees. We would like the input string to have a unique derivation/parse tree.

Leftmost derivation: choose to expand the leftmost non terminal `xAy => xBy, A->B` Where x is a string of terminals, A is a non-terminal, B and y are string of terminal/non-terminal.

## Ambigious Grammars

* A grammar is ambigious if there is a string for which there exists multiple parse trees while using the same derivation style
* Ambigiuity in a grammar does not matter for recognition
* However, the goal of parsing is to create a derivation (and therefore a parse tree)
  * We don't want ambiguity there

## Unambiguous Grammars

```none
expr -> expr op term | term
term -> id | (expr)
op -> + | - | * | /

operations have left to right precedence
```

```none
input: id - id * id
expr -> expr op term
     -> expr op term op term
     -> term op term op term
     -> id op term op term
     -> id - id * id
```

How about expr grammars with * and / given higher precendence (BEDMAS)

```none
expr -> expr PM term | term
term -> term MD factor | factor
PM -> + | -
MD -> * | /
factor -> id | (expr)
```

### Problem

* Proving that a CFG is unambigious is undecidable

## Parsing

### Two approaches to parsing

```none
S => ... => x (input program)
```

* **Top Down Parsing**
  * Start with `\alpha = S`, repeatedly do:
    * Replace some non-terminal A in `\alpha` with `\beta` where `A -> \beta \in \P` until `\alpha = x (input program)`
* **Bottom Up Parsing**
  * Start with `\alpha = x`, repeatedly do:
    * replace some `\beta \in \alpha` with A, where `A -> \beta in P` until `\alpha = S`

## Augmented Grammars

* So simplify parsing algorithms, it helps if the start symbol has just one node
* Any CFG can be "augmented"

```none
G' = { N U { S'}, T U {|-, -|}, P U {S' -> |-S-|}, S'}
```

## Top Down Parsing (by example)

```none
S' -> \alpha_1 -> \alpha_2 .... -> x
Derivation style: leftmost
S' -> |- S -|
S -> AyB
A -> ab
A -> cd

B -> z
B -> wx
```

```none
input |- a b y w x -|
start with S'

S' -> |- S -|

1. Look at current \alpha_i one symbol at a time left to right. While the symbols are terminal, match the symbols with the input.
2. When you hit a non-terminal, choose a rule to apply.
3. Replace the LHS of the rule with the RHS to generate \x_{i+1}

S' -> |- S -|

Step 1: Match |-
Step 2: Non-terminal is S, rule S -> AyB
Step 3: S' -> |- S -| -> |- AyB -|

Now do the same thing again:

Step 1: Match |-
Step 2: Rule A (but A has 2 rules!)
  - option 1, choose all of the rules (brute force) - expensive!
  - option 2, look ahead to the next input symbol
```