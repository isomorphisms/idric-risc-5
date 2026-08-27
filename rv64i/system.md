# RV64I memory ordering and environment instructions

These instructions sit at the edge between ordinary integer computation and the execution environment. They are part of the base integer architecture, but their useful effect depends strongly on the surrounding machine or operating-system contract.

## `FENCE` — order memory and I/O observations

`FENCE` orders selected predecessor memory/I/O operations before selected successor operations as observed by other harts or external devices. Its encoded predecessor and successor sets distinguish reads, writes, input, and output.

This is not a generic "flush everything" instruction and should not be emitted merely as superstition around ordinary loads and stores. A backend should introduce fences because a memory-ordering, concurrency, or device-I/O contract requires them.

`FENCE.TSO` is a constrained form within the `FENCE` encoding space used for TSO-style ordering. It should not be confused with `FENCE.I`.

## `ECALL` — environment call

`ECALL` requests service from the current execution environment by raising an environment-call exception. The base ISA defines the architectural trap; it does not define Linux syscall numbers or argument registers.

For this repository's initial Linux userspace target, the selected ABI convention puts the system-call number in `a7`, arguments in `a0` through `a5`, and the result in `a0`. Those register conventions belong to the Linux/ABI boundary around `ECALL`, not to the semantics of the instruction itself.

The first no-libc executable can therefore use a very small architectural surface: build constants/addresses with RV64I integer instructions and use `ECALL` for Linux `write` and `exit`.

## `EBREAK` — breakpoint exception

`EBREAK` raises a breakpoint exception so that a debugger or other execution environment can take control according to the surrounding privilege/debug rules.

For ordinary generated application code it is primarily useful as an intentional trap/debug boundary rather than a computational primitive.

## What is not in base `I`

### `FENCE.I`

`FENCE.I` synchronizes instruction fetch with earlier stores to instruction memory. It belongs to the separate `Zifencei` extension, not to RV64I itself.

That separation matters for executable code generation: knowing that RISC-V systems commonly provide `FENCE.I` is not permission for an RV64I-only backend to emit it.

### CSR instructions

`CSRRW`, `CSRRS`, `CSRRC`, and their immediate forms belong to `Zicsr`. They are not hidden members of the base `I` set.

### Privileged instructions

Supervisor- and machine-mode instructions, page-table control, interrupt return, SBI interaction, and similar operations belong to privileged architecture or platform layers. The initial Idriç RISC-V target is Linux user mode and does not gain those instructions merely by running on a processor that implements them.

## Backend rule

Keep three questions separate:

1. Is the instruction architecturally known?
2. Is it included by the selected ISA string and execution environment?
3. Has the Idriç backend actually emitted and tested it?

For `FENCE`, `ECALL`, and `EBREAK`, the answer to the first two can be yes under RV64I while the third remains deliberately false until a generated fixture proves otherwise.