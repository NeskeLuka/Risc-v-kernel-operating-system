# Queue

[Documentation index](../README.md) · [Repository README](../../../../README.md)

| Property | Details |
|---|---|
| Files | [`h/queue.hpp`](../../h/queue.hpp) · [`src/queue.cpp`](../../src/queue.cpp) |
| Kind | Value class (two pointers: `head`, `tail`) |
| Used by | [`Scheduler`](Scheduler.md) (ready queue), [`Sem`](Sem.md) (`blockedQueue`), [`TCB`](TCB.md) (`finishedThreads`) |

## Responsibility

An intrusive FIFO of `TCB*` used for ready threads, semaphore waiters, and finished threads awaiting reclamation. “Intrusive” means each element supplies its own link: `TCB::next`. Enqueueing allocates no separate node.

## Interface

| Operation | Result | Complexity |
| --- | --- | --- |
| `Queue()` | Initializes null head and tail | O(1) |
| `put(TCB*)` | Appends a non-null thread; ignores null | O(1) |
| `get()` | Removes the first thread or returns null | O(1) |
| `peek()` | Returns the first thread without removal | O(1) |
| `remove(TCB*)` | Removes the matching thread if present | O(N) |
| `isEmpty() const` | Tests whether head is null | O(1) |

## State and invariants

`head` identifies the oldest item; `tail` identifies the newest. Removing the final item clears both. `get()` and successful `remove()` clear the removed thread's `next` pointer. `put()` clears the new tail's link before attaching it.

A TCB must belong to **at most one queue using `next` at a time**. Inserting a duplicate or adding a thread still linked elsewhere can break both queues. There is no membership validation or locking in this class.

The timer uses the separate `TCB::nextSleep` link. That allows a distinct sleep-list representation, but does not make arbitrary transitions between sleeping, blocked, and ready states safe.

## Ownership

Queue membership does not transfer memory ownership. The queue neither destroys threads nor changes their states. Callers such as `TCB::dispatch`, `Sem::signal`, and the collector implement those policies.

Related: [Scheduler](Scheduler.md), [TCB](TCB.md), [Sem](Sem.md), [Timer](Timer.md).
