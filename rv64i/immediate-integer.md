# RV64I integer operations with immediates

These instructions combine a register value with a constant encoded in the instruction. Ordinary arithmetic/logic immediates use a sign-extended 12-bit immediate. Shift immediates encode a shift amount instead.

RV64I has no arithmetic condition flags. These instructions write their ordinary result to `rd`; a later branch examines registers directly.

## `ADDI` — add immediate

`ADDI rd, rs1, imm12` adds the sign-extended 12-bit immediate to `rs1` and writes the low 64 bits of the sum to `rd`.

This is one of the most useful base instructions. `ADDI rd, rs1, 0` performs a register copy, and `ADDI rd, x0, imm12` constructs a small signed constant. The assembler spellings `mv` and many small `li` cases are pseudoinstructions built from `ADDI`.

## `SLTI` — set if less than immediate, signed

`SLTI rd, rs1, imm12` compares `rs1` with the sign-extended immediate as signed 64-bit integers. It writes exactly `1` when `rs1 < imm` and `0` otherwise.

This is an explicit predicate value in an ordinary integer register, unlike an ARM condition flag.

## `SLTIU` — set if less than immediate, unsigned

`SLTIU rd, rs1, imm12` sign-extends the immediate to XLEN and then compares the two operands as unsigned integers. It writes `1` for true and `0` for false.

A common consequence is that `SLTIU rd, rs1, 1` can test whether `rs1` is zero, which assemblers expose through a pseudoinstruction such as `seqz`.

## `XORI` — exclusive OR immediate

`XORI rd, rs1, imm12` XORs `rs1` with the sign-extended immediate.

Because `-1` sign-extends to all one bits, `XORI rd, rs1, -1` computes bitwise NOT. The assembler `not` spelling is therefore a pseudoinstruction rather than another architectural operation.

## `ORI` — OR immediate

`ORI rd, rs1, imm12` bitwise-ORs `rs1` with the sign-extended immediate.

It is useful for setting known bit fields when the needed mask fits the immediate representation.

## `ANDI` — AND immediate

`ANDI rd, rs1, imm12` bitwise-ANDs `rs1` with the sign-extended immediate.

It is the direct primitive for masks that fit the immediate field: clearing bits, selecting low fields, and turning a wider register value into a small predicate or packed subfield.

## `SLLI` — shift left logical immediate

`SLLI rd, rs1, shamt` shifts `rs1` left, fills low bits with zero, and discards bits shifted beyond bit 63. RV64I uses a 6-bit shift amount, permitting shifts from 0 through 63.

This is ordinary 64-bit logical shifting; it does not set flags.

## `SRLI` — shift right logical immediate

`SRLI rd, rs1, shamt` shifts right and fills vacated high bits with zero.

Use it when the value is unsigned or when a pure bit-field shift is wanted.

## `SRAI` — shift right arithmetic immediate

`SRAI rd, rs1, shamt` shifts right while replicating the original sign bit into the vacated high positions.

This preserves the sign behavior of a two's-complement signed integer under right shift.

# RV64I 32-bit `*W` immediate operations

The `W` forms operate on a 32-bit value and always sign-extend the resulting low 32 bits back to 64 bits. They exist because a 64-bit register machine still needs efficient 32-bit source semantics.

## `ADDIW` — add immediate word

`ADDIW rd, rs1, imm12` adds the sign-extended immediate to `rs1`, keeps the low 32 bits of the sum, and sign-extends that 32-bit result to 64 bits.

`ADDIW rd, rs1, 0` is therefore a useful explicit 32-to-64 sign-extension operation; assemblers commonly call that `sext.w`.

## `SLLIW` — shift left logical immediate word

`SLLIW rd, rs1, shamt` shifts the low 32 bits of `rs1` left by a 5-bit shift amount, keeps a 32-bit result, and sign-extends it to 64 bits.

The upper half of the original source register is not part of the 32-bit operation.

## `SRLIW` — shift right logical immediate word

`SRLIW rd, rs1, shamt` logically shifts the low 32 bits right by a 5-bit amount, then sign-extends the resulting 32-bit value to 64 bits.

The final sign extension is part of the `*W` contract even though the internal shift itself is logical.

## `SRAIW` — shift right arithmetic immediate word

`SRAIW rd, rs1, shamt` arithmetically shifts the low 32 bits right by a 5-bit amount and sign-extends the 32-bit result to 64 bits.

This is the word-width analogue of `SRAI`.

## Backend note

The `*W` instructions should not be emitted merely because the target is RV64. They become useful when the source operation specifically has 32-bit semantics and the sign-extended 32-bit invariant matters. Likewise, a `u16` source value does not imply a nonexistent `*H` arithmetic family: narrow halfword width is explicit at loads/stores, while ordinary arithmetic can occur in the 64-bit register domain with masking/truncation where required.