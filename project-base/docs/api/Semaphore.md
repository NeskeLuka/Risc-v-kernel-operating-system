# Semaphore

[Documentation index](../README.md) · [Repository README](../../../README.md)

| Property | Details |
|---|---|
| Files | [`h/syscall_cpp.hpp`](../../h/syscall_cpp.hpp) · [`src/syscall_cpp.cpp`](../../src/syscall_cpp.cpp) |
| Layer | C++ API — adapts `sem_*` C API calls |
| Kernel counterpart | [`Sem`](../classes/Sem.md) |

## Responsibility

Application-facing C++ ownership wrapper around `sem_t`. The kernel [Sem](../classes/Sem.md) class performs counting, blocking, and wakeup; this wrapper forwards ordinary one-permit operations through the C API.

## Interface

| Operation | Behavior |
| --- | --- |
| `Semaphore(unsigned init = 1)` | Calls `sem_open` and stores the resulting handle |
| Virtual `~Semaphore()` | Calls `sem_close` for a non-null handle, then clears it |
| `wait()` | Calls `sem_wait`, possibly blocking |
| `signal()` | Calls `sem_signal`, potentially making waiters ready |

The handle is private. The wrapper does not expose `value`, `wait_n`, `signal_n`, or a separate close method. Multi-permit operations are available through raw C-style semaphore handles.

## Typical uses

An initial count of one can protect an application critical section; zero can represent a completion or availability event. A producer can signal after writing shared data, and a consumer can wait before reading it.

```cpp
Semaphore available(0);
// Producer, after publishing data:
available.signal();
// Consumer, before consuming that data:
available.wait();
```

The object must remain alive while either side uses it. This snippet illustrates the operations rather than a complete two-thread program.

## Ownership and failure behavior

The destructor gives the wrapper RAII-style handle cleanup, but **not safe concurrent cancellation**. Closing with blocked waiters inherits the underlying `Sem` use-after-free risk. Destroy only after all users have stopped accessing the semaphore.

The constructor ignores the result of `sem_open`; there is no explicit construction-error state. Initial values are narrowed by the kernel to 16 bits.

Copy construction and assignment are not disabled. Copying duplicates the same raw handle and can cause multiple closes, so treat instances as noncopyable by convention. The code has no transfer-of-ownership mechanism.

`wait` and `signal` return the underlying integer status; valid open handles normally return zero, while an observed closed semaphore returns `-1`. Invalid or reclaimed handles do not have a guaranteed safe error result.
