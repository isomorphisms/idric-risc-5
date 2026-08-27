# Idriç RISC-V backend

Architecture-first follower backend for Idriç on 64-bit RISC-V.

ARM/Thumb remains the lead backend. This repository should copy compiler seams and executable test shapes only after they are established there, rather than opening a competing frontend/runtime design.

Current status: **architecture and ABI planning only; no RISC-V code generation is claimed yet.**

The first target boundary is deliberately narrow:

- RV64I, little-endian;
- ELF64 RISC-V;
- `LP64` integer/soft-float psABI;
- Linux userspace;
- no libc or runtime dependency for the first executable oracle.

`ARCHITECTURE.md` records the pinned specifications, the distinction between complete architecture knowledge and emitted/tested instructions, and the deliberately tiny first executable surface.

First milestones:

1. #1 — pin ISA, psABI, and Linux userspace boundaries;
2. #2 — build complete pinned architecture knowledge without treating it as backend support;
3. #3 — bootstrap the first no-libc RV64I executable as an ARM follower.
