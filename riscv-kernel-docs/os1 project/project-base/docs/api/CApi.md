# System Call and Application API Reference

[Documentation index](../README.md) · [Architecture](../architecture.md)

Sources: [C-style declarations](../../h/syscall_c.h), [wrappers](../../src/syscall_c.cpp), [trap dispatch](../../src/riscv.cpp), and [C++ wrappers](../../src/syscall_cpp.cpp).

## Register convention

| Register | Meaning |
| --- | --- |
| `a0` on entry | Operation number |
| `a1`–`a4` | Up to four operation arguments |
| Saved `a0` on return | Result for operations that define one |

The wrapper executes `ecall`. Both user-mode and supervisor-mode ecalls enter the same dispatch table. Unknown operation numbers have no explicit error response: the saved return register remains unchanged.

Although called the “C API,” the supplied header contains C++ forward declarations and is compiled as C++. It is a C-style procedural interface, not a header that currently compiles as ISO C.

## Implemented operations

A dash means the wrapper does not use that argument. Integer result values below describe valid call paths; invalid pointers and stale handles are not comprehensively checked.

| Code | Function | `a1` | `a2` | `a3` | `a4` | Result / behavior |
| --- | --- | --- | --- | --- | --- | --- |
| `0x01` | `mem_alloc(size)` | Byte count | — | — | — | Payload pointer or null |
| `0x02` | `mem_free(ptr)` | Payload pointer | — | — | — | `0`, `-1` for null, or `-2` for selected free-list conflicts |
| `0x11` | `thread_create(handle, body, arg)` | Address of `thread_t` | Entry function | Entry argument | Preallocated stack pointer | Intended `0` success / `-1` failure; failure handling is incomplete |
| `0x12` | `thread_exit()` | — | — | — | — | Finishes current thread and dispatches; successful exit is not expected to return |
| `0x13` | `thread_dispatch()` | — | — | — | — | Voluntary scheduling point; no defined API result |
| `0x14` | `thread_resume(handle)` | Thread handle | — | — | — | Resume extension; no result |
| `0x15` | `thread_suspended(handle)` | Thread handle | — | — | — | Suspend extension; spelling matches the source |
| `0x21` | `sem_open(handle, init)` | Address of `sem_t` | Initial count | — | — | Intended `0` success / `-1` failure |
| `0x22` | `sem_close(handle)` | Semaphore handle | — | — | — | `0` on valid close; `-1` for null |
| `0x23` | `sem_wait(handle)` | Semaphore handle | — | — | — | Acquire one permit; may block |
| `0x24` | `sem_signal(handle)` | Semaphore handle | — | — | — | Release one permit |
| `0x25` | `sem_wait_n(handle, n)` | Semaphore handle | Permit count | — | — | Acquire n permits together; may block |
| `0x26` | `sem_signal_n(handle, n)` | Semaphore handle | Permit count | — | — | Release n permits; may wake several waiters |
| `0x31` | `time_sleep(time)` | Timer ticks | — | — | — | `0` after resumption; `-1` for zero |
| `0x41` | `getc()` | — | — | — | — | One character; blocks on empty input |
| `0x42` | `putc(c)` | Character | — | — | — | Enqueues a character; blocks on full output; no result |

Open-semaphore wait/signal paths return zero. They return `-1` when they observe the object's closed flag, subject to the close-lifetime defect documented in [Sem](../classes/Sem.md).

## Allocation ABI versus the assignment

The supplied assignment specifies bytes at the C API and block counts at the allocation ABI boundary. **This implementation passes bytes in `a1`** and performs rounding in `Riscv::handeEcall`. Direct ABI callers must follow the implementation's byte convention; do not assume it is identical to the assignment contract.

The internal `MemoryAllocator::mem_alloc` accepts block counts including metadata. Both the trap path and `kmalloc` add header size before rounding. A user zero-byte request is therefore not rejected in the same way as `kmalloc(0)`.

## Thread creation and handle lifetime

`thread_create` first calls `mem_alloc(DEFAULT_STACK_SIZE * sizeof(uint64))`, then passes that pointer as the fourth ABI argument. With the supplied constants, the request is 32 KiB.

A non-null entry body is enqueued during TCB construction. The API has no explicit join, detached-state flag, or user-controlled TCB destruction call. The collector later frees completed TCBs. A raw handle is therefore not a stable completion token after termination.

Do not directly `delete` a finished `thread_t`, since kernel reclamation also owns that object. Use application synchronization to coordinate work, and keep callback arguments alive until they are no longer used.

## Semaphore extensions

The C API supports multi-permit operations; the C++ `Semaphore` wrapper exposes only ordinary wait/signal. New calls can bypass queued requests if their smaller request fits the current available count, while `signal` wakes already-queued requests in FIFO order.

The initial count is narrowed to `unsigned short` by the trap handler. Null validation is present in close but absent in several other handle-based operations. Closing with active waiters is not currently safe.

## C++ mapping

| C++ facility | Underlying path |
| --- | --- |
| Global `new` / `new[]` | `mem_alloc` syscall |
| Global `delete` / `delete[]` | `mem_free` syscall |
| `Thread::start()` | `thread_create` |
| `Thread::dispatch()` | `thread_dispatch` |
| `Thread::sleep()` | `time_sleep` |
| `Thread::resume(target)` / `suspend(target)` | Resume/suspend extensions |
| `Semaphore` constructor/destructor | `sem_open` / `sem_close` |
| `Semaphore::wait()` / `signal()` | `sem_wait` / `sem_signal` |
| `PeriodicThread::run()` | Activation followed by `Thread::sleep(period)` |
| `Console::getc()` / `putc()` | Global character syscalls |

## Assembly integration constraints

The inline syscall macro explicitly writes argument registers but does not declare a full clobber set or a `memory` clobber. The return update also depends on a compiler-specific frame layout using x8. Changes to optimization, compiler version, or function inlining need a fresh inspection of generated assembly and runtime tests; source-level API signatures alone do not establish ABI correctness.
