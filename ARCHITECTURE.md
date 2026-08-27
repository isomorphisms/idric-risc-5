# RISC-V architecture boundary

## Scope correction: hosted RV64 is one lane

The existing RV64I + LP64 + ELF64 + Linux tuple was selected by the prior assistant as a cheap QEMU/Linux oracle. The user requested base-`I` follower work but did not choose 64-bit as canonical, and the earlier embedded roadmap names RV32. Issue #1 freezes only this hosted lane; issue #5 requires an RV32I parity lane before the project claims generic RISC-V.

Keep XLEN explicit. RV64-only `LD`, `SD`, `LWU`, `*W`, LP64, ELF64, pointer-width, and data-model assumptions must not enter target-independent IR or RV32 lowering. RefC/C is not a fallback or backend stage.

This repository is intentionally a follower backend. `Idric` owns target-independent language and checked-IR semantics; ARM/Thumb establishes proven backend seams, fixture shapes, oracles, rejection boundaries, and verification structure first. Each RISC-V lane owns its actual XLEN, ISA profile, ABI, object format, and execution environment.

The central rule is that **knowing an instruction exists is not the same as supporting it in generated code**.

## Pinned specifications

### Unprivileged ISA

Normative architecture pin:

- RISC-V Instruction Set Manual, Volume I, official release `20260120`
- upstream release merge: `riscv/riscv-isa-manual@af9c6dee4218263d3006665b20c468f66d953f8d`
- https://docs.riscv.org/reference/isa/v20260120/unpriv/unpriv-index.html

The initial **hosted** executable base is RV64I 2.1, XLEN=64, little-endian.

The rest of the ratified unprivileged manual is still architecture knowledge. `M`, `A`, floating point, compressed instructions, bit manipulation, vectors, half precision, and other extensions are not erased merely because the first backend does not emit them.

### ELF psABI

Initial ABI pin:

- RISC-V ELF psABI `v1.0`
- `riscv-non-isa/riscv-elf-psabi-doc@74936a9e25dec3c5b297b19e4c37e97d14868b22`
- https://github.com/riscv-non-isa/riscv-elf-psabi-doc/releases/tag/v1.0

Initial named ABI: **LP64**.

For the first backend this means:

- ELFCLASS64;
- integer calling convention / soft-float ELF ABI;
- `a0`-`a7` are the eight integer argument registers;
- `a0` and `a1` also carry integer return values;
- `s0`-`s11` are callee-saved;
- `ra` is the conventional return-address register;
- `sp` is the stack pointer and is 16-byte aligned at standard procedure boundaries;
- `gp` and `tp` are fixed ABI registers, not general allocator temporaries.

`LP64D` is a later system-interoperability target, not the first executable requirement. An ABI capable of passing double-precision values does not imply that Idriç must use Float64 as its mathematical representation.

### Linux userspace

The first executable environment is Linux user mode with no libc and no general runtime dependency.

For direct Linux system calls on RISC-V:

- `ECALL` enters the execution environment;
- `a7` carries the system-call number;
- `a0`-`a5` carry up to six arguments;
- `a0` carries the result.

No supervisor-mode, machine-mode, SBI, page-table, interrupt, or kernel code is part of the first backend.

## Application-profile reference, not current target

RVA23U64 v1.0 is useful as the later application-class compatibility reference:

- https://docs.riscv.org/reference/rva23/rva23-profiles.html

It is deliberately much wider than the first compiler surface. RVA23U64 requires RV64I plus many extensions including `M`, `A`, `F`, `D`, `C`, `B`, `V`, `Zfhmin`, and `Zvfhmin`, among others. Full scalar `Zfh` and full vector `Zvfh` are expansion options rather than reasons to widen the first backend automatically.

Therefore this repository does **not** claim RVA23U64 conformance merely by targeting a machine that happens to implement it.

## Four separate states

Every instruction or extension should eventually be classifiable in four independent ways:

| State | Meaning |
| --- | --- |
| **known** | Present in the pinned architecture knowledge. |
| **required** | Required by a selected target/profile. |
| **emitted** | The Idriç RISC-V backend is allowed to generate it. |
| **tested** | Generated use has object/disassembly and executable semantic evidence. |

Do not replace these with one ambiguous `supported = true` flag.

An instruction can be known while not required, emitted, or tested. A profile can require an instruction that a narrower target deliberately does not select. An emitted instruction is not considered finished until generated output has executable evidence.

## Initial hosted target tuple

```text
ISA:          RV64I 2.1
XLEN:         64
endianness:   little
object:       ELF64 RISC-V
psABI:        LP64
execution:    Linux userspace
libc/runtime: none for the first oracle
privilege:    user mode only
```

Current emitted instruction set: **empty**.

That statement should remain literally true until generated RISC-V output is verified.

## First executable surface

Issue #3 owns the first executable milestone.

The initial one-byte-output bootstrap should require only a tiny real-instruction allow-list:

- `ADDI` — small constants and register moves;
- `AUIPC` + `ADDI` — PC-relative address materialization;
- `ECALL` — Linux `write` and `exit`.

Assembler pseudoinstructions do not expand the allow-list. Verification must inspect the resulting object/disassembly and record the real instructions.

After that executable gate is green, widen only when an already-proven ARM fixture requires it. The intended first RV64I scalar families are:

- constants/addressing: `LUI`, `AUIPC`, `ADDI`;
- integer arithmetic/logic: `ADD`, `SUB`, `AND`, `OR`, `XOR`;
- comparisons/branches: `SLT`, `SLTU`, `BEQ`, `BNE`, `BLT`, `BGE`, `BLTU`, `BGEU`;
- memory widths actually demanded by fixtures: `LB`, `LBU`, `LH`, `LHU`, `LW`, `LWU`, `LD`, `SB`, `SH`, `SW`, `SD`;
- direct calls/returns: `JAL`, `JALR`;
- RV64 `*W` operations only where preserving source `Int32` semantics requires them.

This list is a planned ceiling for the first scalar widening, not a claim that all of it is implemented.

## Explicitly later

The following are architecture knowledge from the beginning but executable code-generation work only after a concrete workload earns them:

- `M` multiply/divide;
- `A` atomics and memory-ordering work;
- `F`, `D`, `Zfh*` floating point;
- `C` compressed instructions;
- `B` bit-manipulation instructions;
- `V` and `Zv*` vectors;
- CSR/counter instructions;
- privileged instructions and SBI;
- bare-metal targets;
- dynamic linking, libc, and the LP64D distro boundary.

RISC-V-specific experiments may follow once the common Idriç semantics are executable, but they should not force source/IR design merely to keep this repository busy while ARM is still establishing the compiler.

## First milestones

1. **#1 Pin the RV64 ISA, psABI, and Linux userspace boundary.**
   Keep all upstream versions and the initial target tuple explicit.
2. **#2 Build complete pinned architecture knowledge without implying backend support.**
   Create a reproducible machine-readable inventory with provenance and separate `known` / `required` / `emitted` / `tested` state.
3. **#3 Bootstrap the first no-libc RV64I executable as an ARM follower.**
   Compile one ordinary `main`, emit the tiny allow-list, inspect the object, run it under `qemu-riscv64`, and require exact output.

After #3, port ARM-proven slices one at a time. Issue #5 must add the RV32 counterpart before generic RISC-V is claimed. Do not create a separate RISC-V frontend, runtime model, or numeric representation; keep each lane's necessary ABI/XLEN differences target-local.
