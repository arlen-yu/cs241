# Lecture 22: Optimizations, Memory Management

## Tail Call Optimizations

* Reduce stack usage

```c++
int fact(int n, int a) {
  if (n == 0) {
    return a;
  } else {
    return fact(n-1, n*a);
  }
}
```

## Tail Recursive Function

* Tail recursion: when there is nothing left to do after the recursive call
* When the recursive call returns, the call stack content is not needed
* Reuse the call frame for the recursive call

```none
code(proceude ...)
  = ID.lexeme:
    + sub $29, $30, $4
    + code(decls)
    + code(stmts)
    + code(expr)
    + "add $30, $29, $4"
    + "jr $31"
```

```none
code(factor -> ID(expr1, ... exprn))
  = code(expr1)
    + sw $3, param1.offset($29)
    + ...
    + code(exprn)
    + sw $3 paramn.offset($29)
    + add $30, $29, $4
    + lis $5
    + .word funcName
    + jr $5
```

## Supporting Function Overloading

```none
int f(int a) {...}
int f(int a, int *p) {...}
```

* Labels generated for these would not be unique
* Name mangling - not set strategy
  * Same combination of a unique string, func name + type of parameter
  
```none
F_f_i
F_f_ip
```

* As there is no set name mangling strategy, different compilers do different things
  * difficult to link code produced by different compilers
* C -> no func overloading, compiler does not do name mangling
* C++ -> supports functino overloading, compiler must do name mangling
* How can c++ code call C function foo?
  * in c++ code write `extern "C" int foo(int x);`
* What if we want to write c++ code (function bar) that will be then used by c code
  * int c++ code `extern "C" int bar(int x) { .. }`
  * cannot overload an extern function

## Memory Management of the Heap

* Stack allocated data is only alive for the scope of a variable
* To avoid losing such data either 
  * Copy it to a different scope
  * Use memory other than the stack -- the heap

```none
*d = new C{ .. }
```

* Objects allocated in the heap live on even after the stack allocated variable pointing to the object goes out of scope
* Who/when is the heap memory reclained?

1. Implicit/automtatic "memory management" (garbage collection), racket/java
2. explicit memory management c/c++wlp4, programmers must deallocated

How is the heap implemented?

* Many algorithms
* Freelist algorithm

Maintain a linked list of all the free blocks of memory in the heap

* Where does this linked list reside? Use the free part of the heap
* Assume heap is 1K (1024 bytes)
* Suppose 16 bytes are requested
  * Allocated 20 bytes (16 + 4), the 4 is the space for 1 int

## Limitation of a freelist algorithm

* Can lead to fragmentation
  * Allocate 20, allocate 40, free 20, allocate 5 and you end up with a hole of 15
  * Continuous allocation/deallocation can create holes
  * Can cause allocation requests to fail because the free memory is not located contiguously