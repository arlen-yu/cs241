# Lecture 24: Loading & Linking

## MERL: MIPS Executable Relocatable & Linkable Format

### MERL header

  ```none
  0x1000..002 ;beq $0, $0, 2
  |         | ;addres where MERL file ends
  |         | ;addres where code segment ends

  a way to tell the loader the start and end of the MERL table
  ```

### Code Section

* Assembled code like before

### MERL footer/table

* Relocation entries (for now)
* Each entry has:
  1. Format code, value 1
  2. Address of line that needs relocation

The MERL Cookie (`beq $0, $0, 2`) makes the MERL object file executable without relocation.

### Tools

* cs241.binasm does not produce merl
* cs241.relasm (relocation assembler)
  * cs241.relasm < in.mips > out.mips (relocatable)
* `cs241.merl 0x12345678 < out.merl > out.mips`
  * `0x12345678` is a relocation address
  * `out.mips` is a relocated mips file, meaning header and footer have been removed
* `mips.(twoints|array)` can be give na non-zero starting address
  * `mips.twoints out.mips 0x12345678`

### Loader (with built in relocation)

```none
input: merl file

read(); // ignore merl cookie

endMod <- read()

codeLen <- read() - 12 // need to subtract the 3 words that form the header

a <- findFreeRAM(codeLen + heap + stack)

for(i = 0, i < codeLen, i+= 4)
  MEM[a + i] = read()
; only loading the code (not header/footer)

i <- codeLen + 12

while (i < endMode)
  format <- read()
  if (format != 1)
    ERROR
  else
    rel <- read() // address that needs relocation
    MEM[a + rel - 12] += a - 12
```

```none
lis $2
.word 12
jr $2
jalr $31

this is not relocatable as it explicitly uses addresses and not labels for addresses
```

### Linkers

* Use available libraries e.g. `alloc.merl`, `print.merl`
* We would like to break bigger programs into multiple `.asm` files

### Approach 1

* `cat one.asm two.asm three.asm | cs241.binasm`
  * Works!
  * Angy change requires assembling the entire project
* We would like to be able to seperately assemble each file and then merge them
* But since all these files cannot be all loaded at address 0, we relocate output

```none
one.asm -> relasm -> one.merl
two.asm -> relasm -> two.merl
three.asm -> relasm -> three.merl
```

* Can't just `cat` these together, because of the headers and footers
* Need a smarter tool
  * LINKER 🔌
* Cleverly merge multiple merl files
* Issue: what if a file uses a label it expects to be defined externally?
  * We can tell the assembler that we are using a label defined externally through an import directive
  * If we want to expose a label for others to use, we can use the export directive
* The assembler will convert a `.word <input label>` into an Externa lSymbol Reference (ESR) entry in the MERL table

### ESR entry

```none
.word 0x11 ;format specifyer for ESR
.word address ; the address where the imported label is used
.word <length of label>
.word <character 1 in ASCII>
.word <character 2 in ASCII>
...
.word <character n in ASCII>
```

* FOr each exported label, the assembler created an External Symbol Definition

### ESD entry

```none
.word 0x05 ; format specifyer for ESD
.word < where label is defined>
.word <length of label>
.word <character 1 in ASCII>
.word <character 2 in ASCII>
...
.word <character n in ASCII>
```

* Need to understand linker algorithm for exam, see the avaliable slides ⭐️
* If an ESR is resolved, a REL (relocation) entry should exist in the MERL table
* All ESDs remain
* All unresolved ESRs remain. If you try to execute a MERL file with unresolved ESRs it won't execute, since it hasn't been completely linked!

*End*.
