# TCB

[Documentation index](../README.md) · [Repository README](../../../README.md)

| Property | Details |
|---|---|
| Files | [`h/tcb.hpp`](../../h/tcb.hpp) · [`src/tcb.cpp`](../../src/tcb.cpp) · [`src/contextSwitch.S`](../../src/contextSwitch.S) |
| Kind | Instantiable class; objects come from [`SlotAllocator<TCB>`](SlotAllocator.md) |
| Handle type | `thread_t` is `TCB*` (opaque to user code) |
| Collaborates with | [`Scheduler`](Scheduler.md), [`Queue`](Queue.md), [`Sem`](Sem.md), [`Timer`](Timer.md), [`Riscv`](Riscv.md), [`MemoryAllocator`](MemoryAllocator.md) |

## Responsibility

The thread control block owns a thread's execution metadata and stack, coordinates state transitions, and switches execution through `contextSwitch.S`. Application `thread_t` handles are raw pointers to these objects.

## Stored state

| Field or nested type | Purpose |
| --- | --- |
| `body`, `arg` | Entry function and its argument |
| `stack` | Allocated stack base, released by the destructor |
| `Context { ra, sp }` | Return address and stack pointer used by the assembly switch |
| `state` | `READY`, `RUNNING`, `BLOCKED`, `SLEEPING`, or `FINISHED` |
| `timeSlice` | Assigned quantum; defaults to 2 timer ticks |
| `timeSleepCounter`, `nextSleep` | Relative delay and next entry in the timer list |
| `next` | Intrusive ready, semaphore, or finished-queue link |
| `semWaitCnt` | Number of semaphore permits requested by a blocked wait |
| `isKernelThread` | Selects privilege behavior at initial entry |

Static members identify `running`, `garbageCollector`, the `finishedThreads` queue, and `timeSliceCounter`.

## Public interface

| Operation | Behavior |
| --- | --- |
| `createThread(body, arg, stackSpace, isKernelThread)` | Allocates and initializes a TCB; threads with a body are immediately enqueued |
| `yield()` | Executes dispatch syscall `0x13` |
| `getState()`, `setState()`, `isFinished()` | Inspect or directly change state |
| `getTimeSlice()` | Returns the assigned quantum |
| `createGC()` | Creates and records the supervisor-mode collector thread |
| `suspend(TCB*)`, `resume(TCB*)` | Implement the additional suspend/resume facility |
| `operator new`, `operator delete` | Use `SlotAllocator<TCB>` |
| `~TCB()` | Frees a non-null stack |

The constructor, `dispatch`, `thread_exit`, `threadWrapper`, and `contextSwitch` are internal. Friend classes access scheduling and trap state directly.

## Creation and first execution

A body-bearing TCB uses the provided stack or allocates one. The code treats `DEFAULT_STACK_SIZE = 4096` as a count of `uint64` elements: **32 KiB per stack**. The initial context points `ra` to `threadWrapper` and `sp` just beyond the stack array.

A null-body TCB represents the already executing bootstrap context; it is initially running and is not enqueued. On first entry to a normal thread, `threadWrapper` calls `Riscv::popSppSpie`, invokes the body, marks the thread finished when it returns, and yields.

## Dispatch and saved context

1. A running old thread becomes ready and is appended to the scheduler.
2. A finished old thread, except the collector, is appended to `finishedThreads`.
3. The scheduler supplies the next TCB, which becomes running.
4. Assembly saves the old `ra`/`sp` and restores the new pair.

The 16-byte `Context` is not the whole interrupted register state. `supervisorTrap.S` saves that state in a 256-byte frame on the thread's stack. A previously suspended trap handler can therefore resume after the assembly switch and eventually return with `sret`.

## Deferred cleanup

The collector removes finished entries and deletes them while executing on its own stack. This avoids a thread freeing the stack still being used by its exit path. It is a thread reaper, not a tracing garbage collector. If no finished entry exists, it yields.

The collector briefly masks interrupts around queue removal; deletion occurs after restoration. Allocator operations still need careful serialization.

## Lifetime and transition constraints

A completed thread's raw handle can become dangling as soon as the collector deletes it. `isFinished()` polling through that handle is not a safe join mechanism. Creation also lacks complete allocation-failure handling.

`suspend` can remove ready threads or block the running thread. `resume` only excludes null and already-ready targets; it does not restrict use to explicitly suspended threads. Resuming a running, semaphore-blocked, sleeping, finished, or reclaimed TCB can violate state and queue invariants. See [implementation notes](../implementation-notes.md).
