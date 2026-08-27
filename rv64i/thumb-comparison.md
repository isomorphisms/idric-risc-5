# RV64I as a follower of the Thumb backend

The point of calling this backend a follower is not to imitate Thumb machine code. The useful rule is: preserve the same source semantics, compiler seams, and executable fixtures, then express the last step with the target's actual primitives.

## The condition-code difference

Thumb often combines arithmetic with condition-state production. A loop can look conceptually like:

```asm
subs r0, r0, #1
bne loop
```

`SUBS` changes the integer value and also updates condition flags. `BNE` consumes the zero-condition result.

RV64I instead keeps the arithmetic and branch predicate explicit:

```asm
addi a0, a0, -1
bne  a0, x0, loop
```

There is no general ARM-style `N/Z/C/V` condition register in RV64I. The branch compares its register operands directly.

When the predicate must survive as data rather than immediately controlling a branch, RV64I has instructions such as:

```asm
slt a0, a1, a2
```

which writes exactly `0` or `1` to an ordinary integer register.

This is particularly relevant to small state-machine and one-bit-predicate ideas: on RISC-V the fact often lives explicitly as a register value or as the comparison embedded in a branch rather than as hidden condition-code state.

## Narrow memory remains first-class

Both Thumb and RV64I have explicit byte and halfword memory operations.

| Meaning | Thumb example | RV64I |
| --- | --- | --- |
| load unsigned byte | `LDRB` | `LBU` |
| load signed halfword | `LDRSH` | `LH` |
| load unsigned halfword | `LDRH` | `LHU` |
| store byte | `STRB` | `SB` |
| store halfword | `STRH` | `SH` |
| store 32-bit word | `STR` | `SW` |

So an RGB565 surface remains a very natural test on both architectures. The shared semantic shape is:

```text
pixel : u16
address = base + y*stride + x*2
store low 16 bits of pixel at address
```

Thumb can end that operation with `STRH`; RV64I can end it with `SH`.

The arithmetic leading to that store does not have to be 16-bit. On RV64I, `LHU` places the 16-bit memory value into a 64-bit register with zero extension, ordinary 64-bit integer operations can manipulate it, and `SH` makes the 16-bit memory boundary explicit again.

## Do not conflate three 16-bit ideas

- A **halfword data object** is 16 bits and is handled by `LH`, `LHU`, and `SH` in RV64I.
- A **compressed instruction** is a 16-bit instruction encoding supplied by the optional RISC-V `C` extension. RV64I by itself does not include `C`.
- **FP16** is 16-bit floating-point data/arithmetic supplied through floating-point extensions such as `Zfh*`; it is unrelated to integer halfword loads/stores.

That separation is useful for Idriç. Choosing `u16` storage does not commit the language to compressed instructions or half-precision floating point.

## Rough primitive correspondences

These are semantic analogies, not claims of identical flag behavior or encoding:

| Job | Thumb | RV64I |
| --- | --- | --- |
| small constant / add immediate | `MOVS` / `ADDS` forms | `ADDI` |
| add registers | `ADD` / `ADDS` | `ADD` |
| subtract registers | `SUB` / `SUBS` | `SUB` |
| bitwise AND | `AND` / `ANDS` | `AND` |
| bitwise OR | `ORR` | `OR` |
| bitwise XOR | `EOR` | `XOR` |
| left shift | `LSL` / `LSLS` | `SLL` / `SLLI` |
| logical right shift | `LSR` / `LSRS` | `SRL` / `SRLI` |
| arithmetic right shift | `ASR` / `ASRS` | `SRA` / `SRAI` |
| unsigned halfword load | `LDRH` | `LHU` |
| halfword store | `STRH` | `SH` |
| equality branch | compare/flags + `BEQ` | `BEQ rs1, rs2` |
| signed less-than branch | compare/flags + `BLT` | `BLT rs1, rs2` |
| make signed less-than predicate | flag/conditional sequence | `SLT` |
| direct call | `BL` | `JAL` with link register destination |
| indirect return | `BX lr` or equivalent | `JALR x0, ra, 0` |

## RV64I versus LP64

`RV64I` answers an ISA question: which architectural integer instructions and 64-bit register model are available?

`LP64` answers an ABI/data-model question: how C-like scalar sizes, pointers, calling convention, ELF details, and register roles are arranged for software interoperability.

The backend can therefore study and implement RV64I instruction semantics without pretending that every ISA decision is an ABI decision, and vice versa.

## Follower rule

For each Thumb-proven slice:

1. keep the source operation and observable oracle fixed;
2. keep the target-independent IR meaning fixed unless RISC-V exposes a genuine semantic mismatch;
3. choose the smallest honest RV64I instruction sequence;
4. record the actual emitted instructions rather than assembler pseudoinstructions;
5. test the generated object and executable behavior.

The RISC-V backend should diverge from Thumb exactly where the ISA makes it diverge—condition codes are a good example—without turning that difference into a second language or frontend architecture.