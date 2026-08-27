# RV64I register-to-register integer operations

These instructions take their operands from integer registers. Ordinary RV64I operations work at XLEN=64. The `*W` family deliberately performs a 32-bit operation and then sign-extends its 32-bit result to 64 bits.

A recurring contrast with Thumb is that arithmetic here does not leave behind general condition flags. Comparisons that need to become data use `SLT` or `SLTU`; branches compare registers directly.

## `ADD` — add

`ADD rd, rs1, rs2` adds the two 64-bit operands and writes the low 64 bits of the sum. Integer overflow does not trap and no overflow flag is produced.

Source-language overflow policy therefore belongs above the raw instruction: wrap, check, saturate, or prove impossible as required by the source semantics.

## `SUB` — subtract

`SUB rd, rs1, rs2` computes `rs1 - rs2` modulo 2^64 and writes the result to `rd`.

Unlike Thumb `SUBS`, this does not simultaneously publish zero/negative/carry/overflow condition flags.

## `SLL` — shift left logical

`SLL rd, rs1, rs2` shifts `rs1` left by the amount in the low 6 bits of `rs2`. Vacated low bits are zero-filled and bits shifted beyond bit 63 are discarded.

Only the low 6 bits of the shift-count register matter on RV64I.

## `SLT` — set if less than, signed

`SLT rd, rs1, rs2` compares the operands as signed 64-bit two's-complement integers and writes exactly `1` when `rs1 < rs2`, otherwise `0`.

This is one of the cleanest examples of RISC-V making a predicate an ordinary integer value rather than condition-code state.

## `SLTU` — set if less than, unsigned

`SLTU rd, rs1, rs2` performs the same operation using unsigned comparison and writes `0` or `1`.

It is useful for unsigned source integers, bounds, sizes, address-like quantities, and several small Boolean constructions.

## `XOR` — exclusive OR

`XOR rd, rs1, rs2` computes the bitwise exclusive OR of the two 64-bit operands.

It is useful both as ordinary bitwise arithmetic and for toggling selected bit fields.

## `SRL` — shift right logical

`SRL rd, rs1, rs2` shifts `rs1` right by the amount in the low 6 bits of `rs2`, filling high positions with zeros.

This is the natural variable-count right shift for unsigned values and bit fields.

## `SRA` — shift right arithmetic

`SRA rd, rs1, rs2` shifts right by the low 6 bits of `rs2` while copying the source sign bit into the vacated high positions.

It is the signed two's-complement counterpart to `SRL`.

## `OR` — bitwise OR

`OR rd, rs1, rs2` computes the bitwise OR of the two 64-bit operands.

Masks and packed representations can use it to combine already-positioned fields.

## `AND` — bitwise AND

`AND rd, rs1, rs2` computes the bitwise AND of the two 64-bit operands.

It is the general register-mask primitive for selecting fields, clearing bits, and implementing narrow-value boundaries when the mask does not fit an immediate.

# RV64I 32-bit `*W` register operations

All of these read the low 32 bits needed by the operation and write a 32-bit result sign-extended to 64 bits.

## `ADDW` — add word

`ADDW rd, rs1, rs2` adds the low 32-bit operands, keeps the low 32 bits of the sum, and sign-extends that result to 64 bits.

It gives explicit 32-bit wrapping addition semantics on the 64-bit machine.

## `SUBW` — subtract word

`SUBW rd, rs1, rs2` subtracts the low 32-bit operands, keeps the low 32 bits, and sign-extends the result to 64 bits.

As with `ADDW`, no condition flags are generated.

## `SLLW` — shift left logical word

`SLLW rd, rs1, rs2` shifts the low 32 bits of `rs1` left using the low 5 bits of `rs2` as the count, then sign-extends the low 32-bit result to 64 bits.

The 5-bit shift count reflects the 32-bit operand width.

## `SRLW` — shift right logical word

`SRLW rd, rs1, rs2` logically shifts the low 32 bits of `rs1` right by the low 5 bits of `rs2` and then sign-extends the resulting 32-bit value to 64 bits.

The final sign extension is part of every RV64I `*W` result convention.

## `SRAW` — shift right arithmetic word

`SRAW rd, rs1, rs2` arithmetically shifts the low 32-bit signed value right by the low 5 bits of `rs2`, then sign-extends the result to 64 bits.

This preserves a 32-bit signed right-shift interpretation even though the architectural destination register is 64 bits wide.

## What is absent from `I`

There is deliberately no base `MUL`, `DIV`, or remainder family here; those belong to the `M` extension. There is also no base integer halfword arithmetic family corresponding to `ADDW`. Halfwords are first-class memory widths through `LH`, `LHU`, and `SH`, while 16-bit arithmetic is expressed through ordinary integer operations plus the source language's required extension/masking/truncation semantics.