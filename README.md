# Idriç RISC-V follower backend

This repository follows accepted `Idric` semantics and ARM/Thumb-proven backend seams, fixtures, oracles, rejection boundaries, and verification structure. It uses RISC-V's own registers, comparisons, branches, ABI, encodings, and execution environment.

Current status: architecture/ABI planning and instruction documentation; no RISC-V code generation is claimed yet.

## XLEN scope

The existing first hosted lane is deliberately:

- RV64I, little-endian;
- ELF64 RISC-V;
- LP64 integer/soft-float psABI;
- Linux userspace;
- no libc or general runtime for the first oracle.

That tuple was selected by the prior assistant because it gives a cheap QEMU/Linux bootstrap. The user requested base-`I` follower work but did not choose RV64 as canonical, and an earlier embedded roadmap names RV32.

Therefore:

- issue #1 freezes only the hosted RV64 lane;
- issue #3 owns its one-byte direct executable oracle;
- issue #5 requires RV32I parity before the project claims a generic RISC-V backend;
- XLEN applicability must remain explicit in architecture knowledge and lowering;
- RV64-only `LD`, `SD`, `LWU`, `*W`, LP64, and ELF64 facts must not leak into shared IR.

Complete architecture knowledge remains separate from `required`, `emitted`, and executable-`tested` support. RefC/C is not a fallback or backend stage.

