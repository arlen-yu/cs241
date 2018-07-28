# Lecture 23: Automatic Memory Management, Loaders

## Last time

* Explicit memory management
  * Freelist algorithm
* Fragmentation: repeated allocation/deallocation can lead to "holes" in memory

## Helping with Fragmentation

* To avoid/reduce fragmentation, there are some heuristics

```none
|///|20|///|15|////|100|
```

* Want to allocate 10 bytes
  * First fit: choose the first free block that can accomodate the request
  * Best fit: allocate from the smallest free block that can accomodate the request
  * Worst fit: allocate from the block that leaves the biggest hole

## Automatic Memory Management

```c++
int f() {
  {
    myObject 01 = new myObject()
  } // 01 out of scope now, memory automatically reclaimed
}
```

* Garbage collectors have to be smart and conservative

```c++
int f() {
  myObj 02 = NULL;
  if (...) {
    myObj 01 = new myObj();
    02 = 01;
  } // 01 goes out of scope, but the object is not yet garbage
} // 02 out of scope, garbage collect the object
```

## GC algorithm: Mark & Sweep

* Mark phase - discover reachable part of the heap
  * Identify pointers on the stack pointer to keep memory
  * Follow pointers to the objects (reachable)
  * If reachable/discovered objects have pointers, follow those pointers as well and mark
  * Stop when no new pointers are discovered
* Sweep phase - All parts of the heap not marked are deemed free memory, remove marks

## Reference Counting

* Garbage collector maintains a reference count for each heap object
* Requester tracking each pointer operation so that the references can be updated
* Benefit: no need to stop program for GC
* Disadvantage: extra bookkeeping
* Limitation: cannot detect cycles 

## Copying GC/Stop & Copy

* Split the heap into two: 

```none
from: | |         to: | |
```

* Allocate from "from"
* When you run out of memory in "from", copy reachable objects from "from" to "to", swap "from" & "to"
* Disadvantage: can only use half of the memory
* Benefit: automatic defragmentation, defragments every time you copy from "from" to "to"

## Generational GC

* The older the object the less frequent the attempt to grabage collect
* Frequent FB (using any of the GC algorithms) for younger generations

## Loaders

* A loader is a program that loads a program from a file into RAM & prepares it for execution

```none
OS 1.0
repeat:
  p <- choose program to run
  loader(P) // assumes program is at address 0
  jr $0
  beq $0, $0, repeat


Loader 1.0
  for (i = 0; i < length of program; ++i)
    MEM[4i] = file[i]
```

* Assumption: all programs can be loaded at address 0 is rather niave
  * Running multiple programs?

```none
OS 2.0
repeat:
  p <- choose program to run
  $3 <- loader(p) ; returns starting address of where program was loaded
  jr $3
  beq $0, $0, repeat

Loader 2.0
  Let N be the total memory needed by program (codeLen + stack size + heap size)
  a <- findFreeRam(N) // returns starting address of N free bytes
  for (i = 0; i < codeLen; ++i)
    MEM[4i + a] = file[i]
  $30 <- a + N
```

Places where labels are used:

1. In branches, `beq $0, $0, B`
    * This is not a problem, this is calculated as a difference between 2 addresses
2. `.word <label>`
    * Our assembler replaced `.word <label>` with the address of the label from the symbol table
    * But we started counting at address 0
    * But now we're loading the program at `a`
    * Solution: add `a` to each line that represents a `.word <label>`
      * This is called *relocate*
    * But since machine code is just a sequence of bits. the loader has no way of knowing which line used to be a `.word <label>`
    * Assembler must tell the loader which lines need relocation
    * Assemblers don't just output machine code, they output object code
    * Object file: machine code + additional info needed by loaders (relocation entities) & linker (imports/exports)

