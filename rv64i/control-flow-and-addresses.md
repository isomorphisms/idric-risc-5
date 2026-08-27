# RV64I control flow and address formation

RV64I does not have ARM-style condition flags. Conditional branches compare registers directly, and integer comparisons that need to survive as values use ordinary destination registers. That difference matters to an ARM/Thumb-following backend.

## `LUI` — load upper immediate

`LUI rd, imm20` forms a value from the 20-bit U-immediate in the upper part of a 32-bit quantity, places zeros in the low 12 bits, and on RV64I sign-extends that 32-bit result to 64 bits.

It is the basic tool for constructing constants whose significant bits do not fit in `ADDI`'s signed 12-bit immediate. Assemblers often hide combinations of `LUI` and other instructions behind `li`; the backend should reason about the real instructions instead.

## `AUIPC` — add upper immediate to PC

`AUIPC rd, imm20` forms the same U-immediate shape as `LUI`, adds it to the address of the `AUIPC` instruction, and writes the result to `rd`.

Together with `ADDI`, loads/stores, or `JALR`, it is the basic PC-relative address-building primitive. This is important for position-independent references: the backend can materialize an address relative to where the code actually landed rather than assuming an absolute address.

## `JAL` — jump and link

`JAL rd, offset` writes the address of the following instruction (`pc + 4`) to `rd` and jumps to the PC-relative target. The J-immediate represents an even byte offset with a range of roughly ±1 MiB.

Using `rd = x1` (`ra` in the standard ABI) gives the ordinary direct-call shape. Using `rd = x0` discards the return address and gives an unconditional PC-relative jump. The `j` assembler pseudoinstruction is therefore not a separate architectural instruction.

## `JALR` — jump and link register

`JALR rd, rs1, imm12` writes `pc + 4` to `rd`, adds the sign-extended 12-bit immediate to `rs1`, clears bit 0 of that computed target, and jumps there.

This is the register-indirect control-flow primitive used for indirect calls and returns. The common `ret` spelling is a pseudoinstruction for a suitable `JALR` using `ra`; it is not an additional ISA primitive.

## Conditional branches

All six base conditional branches compare two registers and either take or do not take the PC-relative branch. There is no intervening flags register.

### `BEQ` — branch if equal

`BEQ rs1, rs2, target` branches when the two register values are bitwise equal.

This directly expresses the comparison that Thumb often represents as a flag-setting compare followed by `BEQ`.

### `BNE` — branch if not equal

`BNE rs1, rs2, target` branches when the register values differ.

A loop counter can therefore use `ADDI counter, counter, -1` followed by `BNE counter, x0, loop`; no zero flag has to be produced or preserved.

### `BLT` — branch if less than, signed

`BLT rs1, rs2, target` interprets both operands as signed two's-complement 64-bit integers and branches when `rs1 < rs2`.

Use it when the source comparison is signed. It should not be substituted for `BLTU` merely because the bit patterns happen to agree for positive test values.

### `BGE` — branch if greater than or equal, signed

`BGE rs1, rs2, target` performs the signed comparison `rs1 >= rs2`.

Together `BLT` and `BGE` cover the primitive signed ordering tests; other source-level relations can be formed by operand reversal or control-flow inversion.

### `BLTU` — branch if less than, unsigned

`BLTU rs1, rs2, target` compares the 64-bit register contents as unsigned integers and branches when `rs1 < rs2`.

This distinction matters for addresses, sizes, and explicitly unsigned source values.

### `BGEU` — branch if greater than or equal, unsigned

`BGEU rs1, rs2, target` compares the operands as unsigned integers and branches when `rs1 >= rs2`.

As with the signed pair, this and `BLTU` are enough to derive the other unsigned relational branches without adding new architectural comparisons.

## Thumb comparison

The most important conceptual difference is not the spelling of the branch instructions but where the predicate lives. Thumb commonly uses a flag-setting arithmetic or compare instruction and then a branch that consumes `N/Z/C/V`. RV64I has no general condition-code register: `BEQ`, `BNE`, `BLT`, `BGE`, `BLTU`, and `BGEU` perform their comparisons directly.

When a comparison must become a reusable 0-or-1 value instead of an immediate branch decision, use `SLT`/`SLTU` or related integer constructions described in the register-operation notes.