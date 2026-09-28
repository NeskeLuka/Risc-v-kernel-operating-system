# MyConsole

[Documentation index](../README.md) · [Repository README](../../../../README.md)

| Property | Details |
|---|---|
| Files | [`h/MyConsole.hpp`](../../h/MyConsole.hpp) · [`src/MyConsole.cpp`](../../src/MyConsole.cpp) (RX side lives in [`Riscv::handleSupervisorTrap`](Riscv.md)) |
| Kind | Static utility / singleton (private constructor, static buffers) |
| Named `MyConsole` | to avoid clashing with the user-facing C++ API class [`Console`](../api/Console.md) |
| Collaborates with | [`BoundedBuffer`](BoundedBuffer.md), [`Sem`](Sem.md), [`TCB`](TCB.md), [`Riscv`](Riscv.md) |

## Responsibility

Owns the kernel's console buffers and the body of the supervisor-mode output worker. The application-facing [Console](../api/Console.md) class merely wraps syscalls; `MyConsole` sits on the device side.

## Interface and state

| Member | Purpose |
| --- | --- |
| `inputBuffer` | Public static pointer to the receive `BoundedBuffer` |
| `outputBuffer` | Public static pointer to the transmit `BoundedBuffer` |
| `init()` | Allocates both buffers with `BUFFER_SIZE = 1024` |
| `consoleThreadBody(void*)` | Infinite transmit-worker loop |
| `outputBufferEmpty()` | Delegates to the output buffer's `isEmpty()` |

Construction is private. Initialization is expected once during boot; it has no idempotence guard, failure handling, or matching shutdown routine.

## Input path

A console external interrupt reaches `Riscv::handleSupervisorTrap`. It claims the interrupt from the PLIC and, for `CONSOLE_IRQ`, repeatedly reads the receive register while the status bit reports data. Each byte enters `inputBuffer`. A user `getc` syscall removes a byte, blocking when none are available.

Input is filled by the interrupt handler, **not a separate input thread**.

## Output path

A user `putc` syscall puts one character into `outputBuffer`, blocking if no space is available. The dedicated worker removes a character, polls `CONSOLE_TX_STATUS_BIT` until the device is ready, then writes `CONSOLE_TX_DATA`.

An empty output queue blocks the worker through `Sem::wait`. Once it has a character, the worker **busy-polls hardware readiness**. Buffering moves device polling out of the user syscall; it does not eliminate polling.

## Constraints

`outputBufferEmpty` inherits the full/empty ambiguity of `BoundedBuffer::isEmpty`. Even a truly empty software queue does not prove that the character already removed by the worker has reached the device.

The receive handler uses blocking `put` without a full-buffer drop/defer policy. Both directions also depend on buffer synchronization and semaphore lifetime behavior. These limitations are covered in [implementation notes](../implementation-notes.md).

Related: [Riscv](Riscv.md), [BoundedBuffer](BoundedBuffer.md), [Sem](Sem.md).
