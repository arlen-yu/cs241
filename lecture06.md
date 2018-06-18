# Lecture 6 - Assembler (cont'd), Formal Languages

## Last Time

```none
Creating a symbol table

0     main: lis $2
4     .word 13
8     lis $1
12    .word -1
16    add $3, $0, $0
      loop:
        ; you can add multiple labels to the same line
20      begin: top: odd $3, $3, $2
24    add $2, $2, $1
28    bne $2, $0, loop
32    end: jr $31

main -> 0
loop -> 20
begin -> 20
top -> 20
end -> 32
```

```none
add $3, $2, $4

                  000000 sssss ttttt ddddd 00000 100000
0 << 26           000000  0...0 0...0 0...0 0...0 0...0
2 -> 00010 << 21  0...0 00010  0...0 0...0 0...0 0...0
4 -> 00100 << 16  0...0 0...0  00100 0...0 0...0 0...0
3 -> 00011 << 11  0...0 0...0 0...0 00011 0...0 0...0
32 -> 100000      0...0 0...0 0...0 0...0 0...0 100000

so you "or" everything together and get:

000000 00010 00100 00011 00000 100000
```

In `c++`: `int instr = 0 << 26 | 2 << 21 | 4 << 16 | 3 << 11 | 32;`

We want to print it out.

```c++
// Can't we just do:
int instr = 0 << 26 | 2 << 21 | 4 << 16 | 3 << 11 | 32;
cout << instr;
```

Problem: prints the decimal value not binary (ascii encoding of each decimal digit is output)

Consider the following:

```c++
int x = 65;
char c = 65;
cout << x << c;
// Output 65A
```

This tells us that the conversion to ascii encoding is *not* made when using `char`. So instead:

```c++
int instr = 0 << 26 | 2 << 21 | 4 << 16 | 3 << 11 | 32;
char c = instr >> 24; // 32 - 8 = 24, chars are 8 bits, we want the last 8 bits
cout << c;
c = instr >> 16;
cout << c;
c = instr >> 8;
cout << c;
c = instr; // char takes the least significant 8 bits, so furthest right
cout << c;
```

## Another example

Want to encode `beq $1, $2, -3$`

```none
          000100 sssss ttttt iiiiiiii  iiiiiiii
4 << 26   000100 0...0 0...0 0......0  0......0
1 << 21   0....0 00001 0...0 0......0  0......0
2 << 16   0....0 0...0 00010 0......0  0......0

-3 is tricky

proposed solution: see photos.
```

# New Topic (A4) - Formal Languages

* Specification: MIPS reference sheet
* Recognition: dont by the assembler (is the program valid?)
* Low-level languages have simple structure (easy to do recognition and synthesis)

## High-level languages have complex structure

* Harder to recognize
* Harder to synthesize
* How do we handle the increased complexity?

IDEA: *use formal theory in string recognition*.

* This makes it applicable to any general langauge

Formal language: *mathematically precise way of specifying a language*.

* Noam Chomsky - linguist, philosopher, social/political activist

## Definitions

* **Alphabet**: a finite set of symbols
  * `{0, 1}, {0, 1, ... 9}, {a, ... x, y, z}`
  * Symbol doesn't have to be a simple character: `{add, jr, beq, $1, ...}`
* **Word** (over alphabet): a finite sequence of symbols
  * `01101` over `{0, 1}`
  * `42` over `{0, ... 9}`
  * `hello` over `{a, ... z}`
* **Empty Word**: a word with no symbols
* **Language**: a possibly infinite set of words
  * L = all binary strings of length 8
    * `|L|` = 2^8 (finite language)
  * L = all prime numbers in decimal
    * infinite language
  * Empty language `{}`, no words
  * Empty language containing just the empty word `{ę}`

## Recognition

* A yes/no answer whether a given input program follows a language specification
  * Is the input a valid word in the language?
* How to automatically regonize whether an input word is in a language?
  * Depends on how complex the language is
  * `{a ^{2n}b, n \geq 0}` easy
  * `{all valid MIPS assembly programs}` easy
  * `{all valid c/c++/java programs}` harder
  * Can classify languages based on the type of recognition algorithm needed for that language

```none
easy      finite languages
          regular languages
          contexst free languages
          context sensitive languages
          recursive languages
hard      indecidable languages - no recognition algorithm
```

## Finite Languages

* A finite set of words
* **Specification** = list out all the words
* **Recognition** = compare input word to each word in the language

Example: `L = {cat, car, cow}`. Write a regonizer by scanning the input word exactly wonce and without storing previous characters
