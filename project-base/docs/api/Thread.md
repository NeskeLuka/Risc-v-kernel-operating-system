# Thread

[Documentation index](../README.md) · [Repository README](../../../README.md)

| Property | Details |
|---|---|
| Files | [`h/syscall_cpp.hpp`](../../h/syscall_cpp.hpp) · [`src/syscall_cpp.cpp`](../../src/syscall_cpp.cpp) |
| Layer | C++ API — adapts [`thread_*` C API](CApi.md) calls |
| Kernel counterpart | [`TCB`](../classes/TCB.md) |

## Responsibility

Application-facing C++ wrapper around a kernel `thread_t`. It supports either a function body supplied to its constructor or subclassing with an overridden `run()` method.

## Interface

| Operation | Behavior |
| --- | --- |
| `Thread(void (*body)(void*), void* arg)` | Stores the callback and argument; does not create a TCB yet |
| Protected `Thread()` | Prepares a subclass-based thread |
| `start()` | Creates the kernel thread once; returns `-1` if a handle is already set |
| `dispatch()` | Static voluntary yield |
| `sleep(time_t)` | Static relative sleep in timer ticks |
| `resume(Thread*)`, `suspend(Thread*)` | Operate on the supplied target object's kernel handle |
| Protected `run()` | Virtual body; default implementation is empty |
| Virtual `~Thread()` | Empty; does not stop, join, or destroy the kernel thread |

`myHandle`, `body`, and `arg` are private. If `body` is null, `start` passes a small adapter and `this` to `thread_create`; the adapter invokes virtual `run()`.

## Usage

```cpp
#include "../h/syscall_cpp.hpp"

class Worker : public Thread {
protected:
    void run() override {
        Console::putc('W');
    }
};
```

Create a `Worker` and call `start()` from the user application. Keep the object alive for the entire execution of `run()`. Use explicit application synchronization for completed work; the wrapper has no `join()` operation.

## Lifetime

The wrapper object and its TCB are separate allocations. The kernel collector reclaims finished TCBs and their stacks. Deleting the wrapper does not wait for that event, and deleting it while virtual `run()` still uses `this` is unsafe.

`start()` does not reset `myHandle` after completion, so instances are not restartable. The stored raw handle can outlive the TCB; later suspend/resume calls through a completed wrapper are unsafe.

The suspend/resume methods have an explicit target parameter. For example, `controller.suspend(&worker)` targets `worker`; they are not parameterless self-suspension methods. They also dereference the target without a null check and inherit the kernel extension's state restrictions.

Related: [TCB](../classes/TCB.md), [PeriodicThread](PeriodicThread.md), [C API](CApi.md).
