# Scheduler

[Documentation index](../README.md) · [Repository README](../../../README.md)

| Property | Details |
|---|---|
| Files | [`h/scheduler.hpp`](../../h/scheduler.hpp) · [`src/scheduler.cpp`](../../src/scheduler.cpp) |
| Kind | Static utility (singleton by construction: private constructor, one static `Queue`) |
| Algorithm | **FIFO / round-robin** |
| Collaborates with | [`TCB`](TCB.md) (`dispatch`, `resume`, `suspend`), [`Sem`](Sem.md) (`signal`, `close`), [`Timer`](Timer.md) (`tick`), [`Queue`](Queue.md) |

## Responsibility

Provides the kernel's shared FIFO ready queue. It chooses the next queued thread; `TCB` performs the actual state changes and assembly context switch. The scheduler itself has no timer, priority model, or independent scheduling loop.

## Interface

| Operation | Behavior |
| --- | --- |
| `put(TCB*)` | Appends through `readyQueue.put` |
| `get()` | Removes and returns the oldest ready entry, or null |
| `remove(TCB*)` | Removes a specified entry if present |

The only data member is static `Queue readyQueue`. Construction is private. `put` and `get` are O(1); removal is O(N).

## Scheduling policy

New threads with bodies are enqueued during TCB construction. On a normal dispatch, the running thread becomes ready and goes to the tail. The next thread comes from the head. Timer-driven dispatch after the configured slice therefore gives round-robin behavior among runnable threads.

Blocked and sleeping threads are not requeued by dispatch. Semaphore signaling and timer expiration explicitly make them ready and append them. A waking thread does not immediately preempt the current one merely because it was enqueued.

## Caller contract

The caller must maintain valid thread states and unique queue membership. This wrapper does not reject blocked, finished, running, or duplicate entries. It neither owns nor frees TCBs and contains no lock.

When `get()` returns null, `TCB::dispatch()` falls back to the old thread and marks it running. That fallback is not a complete idle-thread policy and can be unsafe if the old thread was blocked or finished. Startup normally leaves bootstrap/collector work available, but the empty-queue case still matters when reasoning about correctness.

Related: [Queue](Queue.md), [TCB](TCB.md), [Timer](Timer.md).
