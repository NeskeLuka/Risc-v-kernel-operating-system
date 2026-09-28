# Console

[Documentation index](../README.md) · [Repository README](../../../README.md)

| Property | Details |
|---|---|
| Files | [`h/syscall_cpp.hpp`](../../h/syscall_cpp.hpp) · [`src/syscall_cpp.cpp`](../../src/syscall_cpp.cpp) |
| Layer | C++ API — adapts `getc` / `putc` |
| Kernel counterpart | [`MyConsole`](../classes/MyConsole.md) |

## Responsibility

A stateless application API for single-character console input and output. Both operations are static and forward directly to the global C-style functions.

## Interface

| Operation | Behavior |
| --- | --- |
| `static char getc()` | Calls `::getc()`; waits for an input character when the receive buffer is empty |
| `static void putc(char c)` | Calls `::putc(c)`; may wait for space in the output buffer |

There is no stream object, formatting engine, line reader, flush method, or per-instance state. Test-side utilities such as `printString` build richer output by repeatedly calling the character API.

## Data path

Input travels from the receive device register, through the console interrupt handler and `MyConsole::inputBuffer`, to the `getc` syscall and this wrapper.

Output travels through the `putc` syscall into `MyConsole::outputBuffer`. The supervisor output worker later polls device readiness and transmits the byte. Returning from `putc` means the byte has been enqueued, not necessarily displayed.

```cpp
char c = Console::getc();
Console::putc(c);
```

This echoes one character in a user thread. A sequence of `putc` calls is not an atomic line write; application threads may need their own semaphore to keep messages together.

## Constraints

The interface returns `char`, despite the C header declaring `EOF = -1`. The current implementation does not provide an explicit EOF protocol comparable to hosted standard I/O. Do not substitute hosted `getc(FILE*)` or assume libc semantics.

The wrapper inherits the kernel buffer's fullness, synchronization, and shutdown limitations. It has no status return for output failures.

Related: [MyConsole](../classes/MyConsole.md), [BoundedBuffer](../classes/BoundedBuffer.md), [System calls](CApi.md).
