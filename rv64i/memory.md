# RV64I memory instructions

RV64I has explicit load and store widths. The address for these instructions is `rs1 + sign_extend(imm12)`. Loads widen the memory value into a 64-bit integer register; stores take the low bits of a 64-bit register and write only the selected memory width.

This is one of the places where RV64I fits the same low-level sensibility as Thumb: bytes, halfwords, words, and doublewords remain explicit architectural choices even though the general integer registers are 64 bits wide.

## Byte loads

### `LB` — load byte, sign-extended

`LB rd, offset(rs1)` reads 8 bits from memory and sign-extends that byte to 64 bits in `rd`.

Use it for signed 8-bit data. A stored byte whose top bit is 1 becomes a negative 64-bit two's-complement value after the load.

### `LBU` — load byte, zero-extended

`LBU rd, offset(rs1)` reads 8 bits and fills the upper 56 bits of `rd` with zero.

Use it for unsigned bytes, character/octet data, masks, and any other situation where the memory byte is a number in `0..255`.

## Halfword loads

### `LH` — load halfword, sign-extended

`LH rd, offset(rs1)` reads 16 bits from memory and sign-extends the result to 64 bits.

This is a genuine 16-bit memory operation. "Halfword" here means 16 bits regardless of RV64I's 64-bit register width.

### `LHU` — load halfword, zero-extended

`LHU rd, offset(rs1)` reads 16 bits and zero-extends the result to 64 bits.

This is the natural load for an unsigned `u16` memory representation such as an RGB565 pixel. The value occupies 16 bits in memory even though subsequent ordinary integer arithmetic occurs in a 64-bit register.

## Word loads

### `LW` — load word, sign-extended

`LW rd, offset(rs1)` reads 32 bits from memory and sign-extends the 32-bit result to 64 bits.

On RV64I this sign extension is part of the instruction semantics; it is not merely an ABI convention added afterward.

### `LWU` — load word, zero-extended

`LWU rd, offset(rs1)` reads 32 bits and zero-extends the result to 64 bits.

Use it when the memory object is explicitly unsigned 32-bit data and the upper half of the destination must be zero.

## Doubleword load

### `LD` — load doubleword

`LD rd, offset(rs1)` reads the full 64-bit doubleword into `rd`.

In RISC-V terminology, a **word is always 32 bits** and a **doubleword is always 64 bits**, even on RV64. That fixed vocabulary keeps `LW` and `LD` unambiguous.

## Stores

### `SB` — store byte

`SB rs2, offset(rs1)` writes the low 8 bits of `rs2` to memory.

The remaining bits of `rs2` are irrelevant to the store. This makes truncation at the memory boundary explicit.

### `SH` — store halfword

`SH rs2, offset(rs1)` writes the low 16 bits of `rs2` to memory.

For a packed 16-bit surface, this is the direct RISC-V analogue of Thumb `STRH`. A pixel address such as `base + y*stride + x*2` can terminate in one `SH` without pretending that the processor has 16-bit general-purpose registers.

### `SW` — store word

`SW rs2, offset(rs1)` writes the low 32 bits of `rs2` to memory.

It does not write the upper 32 bits of the source register.

### `SD` — store doubleword

`SD rs2, offset(rs1)` writes all 64 bits of `rs2` to memory.

This is the natural full-register-width store in RV64I.

## Width table

| Memory width | Signed load | Unsigned load | Store |
| --- | --- | --- | --- |
| 8 bits | `LB` | `LBU` | `SB` |
| 16 bits | `LH` | `LHU` | `SH` |
| 32 bits | `LW` | `LWU` | `SW` |
| 64 bits | `LD` | same bits, no widening needed | `SD` |

## Alignment and endianness

This repository's initial target tuple is little-endian. The instruction names themselves select width, not byte order.

The backend should also avoid making accidental assumptions about misaligned accesses. Whether a misaligned load/store completes invisibly or traps depends on the execution environment and selected platform requirements. Generated structures and surfaces should therefore keep naturally aligned accesses when the data layout allows it.

## Why halfwords matter here

The presence of `LH`, `LHU`, and `SH` is separate from both compressed instructions and FP16. It means the ISA regards a 16-bit memory object as a normal primitive boundary. That is enough to make `u16`, packed pixels, small table entries, and other narrow storage forms first-class backend concerns without claiming 16-bit register arithmetic.