# Sem

[Documentation index](../README.md) · [Repository README](../../../README.md)

| Property | Details |
|---|---|
| Files | [`h/sem.hpp`](../../h/sem.hpp) · [`src/sem.cpp`](../../src/sem.cpp) |
| Kind | Instantiable; objects come from [`SlotAllocator<Sem>`](SlotAllocator.md) |
| Handle type | `sem_t` is `Sem*` |
| Collaborates with | [`TCB`](TCB.md), [`Scheduler`](Scheduler.md), [`Queue`](Queue.md), [`BoundedBuffer`](BoundedBuffer.md) |

## Responsibility

Implements kernel counting semaphores. Unlike the application-facing `Semaphore` wrapper, this class owns the permit count and queue of blocked TCBs. It supports acquiring or releasing multiple permits in one operation.

## Interface

| Operation | Behavior |
| --- | --- |
| `Sem(unsigned short val = 1)` | Sets the initial count |
| `semOpen(unsigned short val)` | Allocates a new instance |
| `wait(unsigned n = 1)` | Consumes n permits or blocks the current thread |
| `signal(unsigned n = 1)` | Adds n permits and wakes satisfiable queued requests |
| `value() const` | Returns the available count |
| `close()` | Marks closed and makes all queued waiters ready |
| `isClosed() const` | Returns the closed flag |
| `~Sem()` | Closes the instance if needed |
| `operator new/delete` | Obtain/release slots through `SlotAllocator<Sem>` |

`wait` and `signal` return `-1` when they observe a closed semaphore, otherwise `0`. Initial values pass through a 16-bit unsigned parameter; the public API's wider `unsigned` value is narrowed in the trap handler.

## Waiting and signaling

If `val >= n`, `wait` subtracts n immediately. Otherwise it leaves the available count unchanged, stores n in `TCB::semWaitCnt`, marks the current thread blocked, queues it, and dispatches.

`signal` adds n, then examines the queue head. If enough permits exist for that request, it deducts them, removes the waiter, clears its request count, marks it ready, and appends it to the scheduler. It repeats until the head request cannot be satisfied.

## Ordering semantics

Queued wakeups preserve FIFO order: a large request at the head can delay smaller requests behind it. However, **new wait calls may bypass existing queued requests** if the currently available count satisfies them, because `wait` does not first check whether the queue is empty. This is not a strict fairness guarantee across all arrivals.

Waiting is O(1) excluding scheduling. Signaling is O(W) for W awakened waiters; closing is O(B) for B blocked threads. Zero-permit operations are accepted on an open semaphore.

## Closing and ownership

`close` masks interrupts while marking the object closed and releasing its wait queue. A resumed `wait` checks `closed` and intends to report failure.

There is a lifetime defect in the syscall path: `sem_close` calls `close()` and immediately deletes the object, while resumed waiters later read `this->closed`. Thus closing with blocked waiters can access freed storage. Do not describe this as safe cancellation; see [implementation notes](../implementation-notes.md).

Raw handles receive little validation, and large permit values can overflow the signed internal count or narrow during conversion. Direct kernel use requires appropriate serialization.

Related: [Semaphore](../api/Semaphore.md), [Queue](Queue.md), [BoundedBuffer](BoundedBuffer.md).
