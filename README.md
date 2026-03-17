# LC4 Disassembler — Dynamic Memory & File I/O in C
---

## File Structure

```
.
├── lc4.c                    # Main driver — orchestrates load, disassemble, print
├── lc4_memory.c             # Linked list implementation
├── lc4_memory.h             # Linked list node struct & function declarations
├── lc4_loader.c             # Binary .obj file parser & endianness handler
├── lc4_loader.h             # Loader function declarations
├── lc4_disassembler.c       # Reverse assembler — opcode → mnemonic translation
├── lc4_disassembler.h       # Disassembler function declarations
├── Makefile                 # Multi-target build: compile → link → clean
├── mytest.asm               # Custom integration test (hand-written LC4 assembly)
└── mytest2.asm              # Custom integration test #2
```

> Source code is omitted from this repository to comply with Penn's academic integrity policy.

---

## Overview

This project is a fully functional **LC4 binary disassembler** written in C. Given a compiled LC4 `.obj` binary file, the program parses its raw bytes, builds an in-memory representation of the program, and reconstructs human-readable LC4 assembly source code — the reverse of what an assembler does.

The LC4 architecture is a 16-bit educational ISA used throughout Penn's systems curriculum.

---

## What It Does

| Stage | Description |
|---|---|
| **Load** | Opens and parses a binary `.obj` file, handling big-endian to little-endian byte swapping |
| **Store** | Inserts each instruction into a sorted, singly linked list ordered by memory address |
| **Disassemble** | Decodes 16-bit opcodes and operands back into their assembly mnemonics (e.g., `ADD`, `LDR`, `JSRR`) |
| **Output** | Prints the fully annotated memory listing with addresses, raw hex values, labels, and assembly text |

---

## Technical Highlights

### Dynamic Memory Management
The core data structure is a custom **singly linked list** (`row_of_memory`) allocated entirely on the heap. Each node stores:
- A 16-bit memory address
- The 16-bit instruction/data contents
- An optional label string (heap-allocated)
- A disassembled assembly string (heap-allocated)

All heap memory is explicitly freed at program exit with zero leaks, as confirmed by Valgrind.

### File Parsing & Endianness
Binary `.obj` files are read byte-by-byte. The loader correctly handles **endianness conversion** (`swap_endianness`) to reconstruct 16-bit values from the big-endian file format into the host machine's representation.

### Reverse Assembly (Disassembler)
`reverse_assemble()` decodes every instruction class in the LC4 ISA by masking and shifting the raw 16-bit opcode — covering arithmetic, logical, memory, branch, and subroutine instructions — and writes the corresponding assembly string back into each linked list node.

### Build System
A hand-written `Makefile` compiles the project into three independently linked object files (`lc4_memory.o`, `lc4_loader.o`, `lc4_disassembler.o`) before linking into a final `lc4` executable using `clang -Wall -g`.

---

## Skills Demonstrated

- **C programming** - pointers, pointer-to-pointer (`**head`), manual heap allocation (`malloc`/`free`), string manipulation
- **Data structures** - ordered linked list with insert, search-by-address, search-by-opcode, and delete operations
- **Systems programming** - binary file I/O, endianness handling, bitwise masking/shifting for instruction decoding
- **Memory safety** - Valgrind-clean with zero leaks or invalid reads across all test cases
- **Build tooling** - multi-target Makefile with separate compile and link stages

---

## Test Results

```
UNIT TESTING SUBSCORE:        50.0 / 50.0
INTEGRATION TESTING SUBSCORE: 50.0 / 50.0
Valgrind issues:              0

TOTAL SCORE: 100.0 / 100.0
```

Unit tests covered `add_to_list`, `search_address`, `search_opcode`, `delete_list`, `open_file`, and `reverse_assemble` across 39 individual test cases. Integration tests ran end-to-end disassembly against 9 reference programs including OS code, jump instructions, and mixed CODE/DATA segments.
