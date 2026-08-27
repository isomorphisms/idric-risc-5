# RV64I base integer instruction set

This directory records the **RV64I 2.1** base integer ISA selected by this repository's architecture boundary. It is architecture documentation, not a claim that the Idriç backend already emits every instruction described here.

The important naming distinction is:

- `RV64` means XLEN is 64 bits: the ordinary integer registers are 64 bits wide.
- `I` is the base integer instruction set.
- `LP64` is an ABI/data-model choice and is not an instruction-set extension.
- `M`, `A`, `F`, `D`, `C`, `B`, `V`, `Zicsr`, `Zifencei`, and other extensions are separate from the `I` base unless explicitly selected.

## Architectural state

RV64I has 32 integer registers, `x0` through `x31`. `x0` always reads as zero and discards writes. The program counter is architectural state but is not an ordinary integer register. ABI names such as `zero`, `ra`, `sp`, `a0`, and `s0` are names assigned to those same registers by the calling convention.

The base integer ISA is deliberately small. This directory groups its instructions by the job they perform:

- [Control flow and address formation](control-flow-and-addresses.md): `LUI`, `AUIPC`, `JAL`, `JALR`, and the six conditional branches.
- [Memory](memory.md): byte, halfword, word, and doubleword loads/stores.
- [Immediate integer operations](immediate-integer.md): arithmetic, comparisons, logic, shifts, and RV64 `*W` immediate operations.
- [Register integer operations](register-integer.md): arithmetic, comparisons, logic, shifts, and RV64 `*W` register operations.
- [Memory ordering and environment calls](system.md): `FENCE`, `ECALL`, and `EBREAK`.
- [Thumb comparison](thumb-comparison.md): the parts that matter when this backend follows the ARM/Thumb work.

## Things deliberately not counted as extra base instructions

Assembler pseudoinstructions such as `li`, `mv`, `nop`, `not`, `neg`, `seqz`, `snez`, `j`, `jr`, `ret`, `call`, and `tail` are conveniences that assemble into real instructions. They should never silently widen the backend's emitted-instruction allow-list.

`FENCE.I` belongs to `Zifencei`, CSR instructions belong to `Zicsr`, multiplication/division belongs to `M`, atomics belong to `A`, floating point belongs to floating-point extensions, and compressed 16-bit instruction encodings belong to `C`.

## Three unrelated meanings of 16 bits

This distinction is worth keeping explicit:

1. **16-bit data**: `LH`, `LHU`, and `SH` operate on halfwords in memory.
2. **16-bit instruction encodings**: the optional `C` extension supplies compressed encodings; that is not part of RV64I itself.
3. **16-bit floating point**: half-precision floating point belongs to `Zfh*`-family extensions, not to the integer halfword instructions.

For Idriç, the first meaning is already useful even on a 64-bit machine. A `u16` value can have 16-bit storage and value semantics while ordinary arithmetic still takes place in 64-bit integer registers.

## Sources

Normative reference for this branch:

- RISC-V Instruction Set Manual, Volume I, official release `20260120`
- RV32I Base Integer Instruction Set, Version 2.1: https://docs.riscv.org/reference/isa/unpriv/rv32.html
- RV64I Base Integer Instruction Set, Version 2.1: https://docs.riscv.org/reference/isa/unpriv/rv64.html
- ISA extension naming: https://docs.riscv.org/reference/isa/unpriv/naming.html

The RV64I chapter defines the 64-bit additions and is read together with the RV32I base chapter, because RV64I inherits most base integer operations from RV32I.