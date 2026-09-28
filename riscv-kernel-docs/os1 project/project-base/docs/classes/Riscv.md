# Riscv

[Documentation index](../README.md) · [Repository README](../../../../README.md)

| Property | Details |
|---|---|
| Files | [`h/riscv.hpp`](../../h/riscv.hpp) · [`src/riscv.cpp`](../../src/riscv.cpp) · [`src/supervisorTrap.S`](../../src/supervisorTrap.S) |
| Kind | Static utility class (never instantiated) |
| Layer | **ABI** — the single door from user code into the kernel |
| Collaborates with | [`TCB`](TCB.md), [`Timer`](Timer.md), [`Sem`](Sem.md), [`MemoryAllocator`](MemoryAllocator.md), [`MyConsole`](MyConsole.md) |

## Responsibility

Connects the processor's trap mechanism to kernel services. It exposes supervisor CSR helpers, installs the assembly entry point, dispatches system calls, handles timer and console interrupts, and terminates the QEMU guest on unhandled traps or normal shutdown.

## Interface groups

| Members | Purpose |
| --- | --- |
| `r_*/w_*` for `scause`, `sepc`, `stvec`, `stval`, `sip`, `sstatus` | Read/write supervisor control and status registers |
| `ms_sip`, `mc_sip`, `ms_sstatus`, `mc_sstatus` | Set/clear register bits |
| `BitMaskSip` | Names SSIP, STIP, and SEIP masks |
| `BitMaskSstatus` | Names SIE, SPIE, and SPP masks |
| `supervisorTrap()` | Assembly trap entry |
| `popSppSpie()` | Performs the initial `sret` transition for a thread |
| `endProgram()` | Writes the QEMU shutdown value to the mapped exit device |

Private helpers include `handleSupervisorTrap`, `handeEcall` (spelling retained from source), `updateResult`, and `printError`.

## Trap frame and return value

`supervisorTrap.S` reserves 256 bytes on the current thread's stack and saves x0–x31 in eight-byte slots. It calls the C++ handler, restores registers, adjusts `sp`, and returns with `sret`.

Arguments arrive in `a0`–`a4`. `a0` carries the operation number and eventually the return value. `updateResult` stores into the saved x10/a0 slot at offset 80 from x8. **x8 is `s0`/the frame pointer, not `sp`**. Correctness depends on the expected compiler frame and inlining behavior; `-fno-omit-frame-pointer` is part of the build contract.

## Dispatch cases

| `scause` | Action |
| --- | --- |
| `8`, `9` | User/supervisor `ecall`; advance saved `sepc` by 4, dispatch the call, restore status and PC |
| `0x8000000000000001` | Supervisor software interrupt used as the platform timer notification |
| `0x8000000000000009` | Supervisor external interrupt; claim and service console IRQ, then complete it |
| Other | Optionally print a user-mode diagnostic, then terminate the guest |

The timer branch clears SSIP, advances `Timer`, and dispatches when the running thread exhausts its slice. It preserves the interrupted PC without advancing it. The console branch drains received bytes into the input buffer.

## Privilege and fault behavior

For a user thread, `popSppSpie` clears SPP, sets SPIE, points `sepc` at the return address, and executes `sret`. Kernel threads retain the supervisor return context established by kernel execution.

Unexpected user exceptions do not merely kill one thread: the current implementation shuts down the whole guest. The diagnostic is generic and does not identify every possible trap accurately. Shared address space means this mechanism should not be presented as full memory isolation.

Unknown syscall numbers leave the return slot unchanged. Argument validation is limited, and the assembly interface depends on the compiler's register/frame behavior. See the [syscall reference](../api/CApi.md) and [implementation notes](../implementation-notes.md).
