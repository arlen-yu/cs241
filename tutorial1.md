# Tutorial 1

## Converting unsigned number to one from binary

1. write the number
2. divide by 2, remember quotient and remainder
3. repeat 2 using quotient until you reach 0
4. write remainders from bottom to top - that's your binary number

```none
e.g.

23 / 2 = 11 rem 1
11 / 2 = 5 rem 1
5 / 2 = 2 rem 1
2 / 2 = 1 rem 0
1 / 2 = 0 rem 1
```

## Converting binary to decimal

Trivial

## Converting unmbers to and from two's compliment binary

### Q: What is `-x`

A: A number which, when added to x, `= 0`

```none
Let's say we have 8 bit number (must know how many bits we have)

      10000001  <- -127
      01111111  <- 127
      --------
      10000000

-x is the number such that x + (-x) = 0 mode 2
```

### Converting -5 to 2-bit 2's compliment

1. encode positive number (5)
2. flip every bit (1's compliment)
3. add 1

```none
0101
1010
1011
----
  -5

8 + 2 + 1 = 11
11 + 5 = 16 congruent to 0 mod 2^4
11 = -5 in 4 bit two's compliment notation
```

```none
encode -128 in 8-bit 2's compliment binary

```