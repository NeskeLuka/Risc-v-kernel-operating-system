# Timer

[Documentation index](../README.md) · [Repository README](../../../../README.md)

| Property | Details |
|---|---|
| Files | [`h/timer.hpp`](../../h/timer.hpp) · [`src/timer.cpp`](../../src/timer.cpp) |
| Kind | Static utility with one static: `TCB* sleepHead` |
| Time unit | One **timer tick** (the timer fires 10 times per second → 100 ms) |
| Collaborates with | [`TCB`](TCB.md) (`nextSleep`, `timeSleepCounter`, `dispatch`), [`Scheduler`](Scheduler.md), [`Riscv`](Riscv.md) (calls `tick`) |

## Responsibility

Maintains the sleeping-thread list. The trap handler supplies timer ticks and separately handles time-slice preemption; `Timer` does not program the hardware timer itself.

## Interface

| Operation | Behavior |
| --- | --- |
| `sleep(time_t time)` | Blocks the current thread for a positive relative delay; returns `0` after resumption, `-1` for zero |
| `tick()` | Decrements the head delay and wakes all expired leading entries |

Time is expressed in **timer ticks**, not milliseconds. `time_t` is unsigned. A negative integer converted to it becomes a large delay rather than a rejected negative duration.

## Relative-delay representation

`sleepHead` points to the first sleeping TCB. Each TCB uses `nextSleep` and `timeSleepCounter`; the latter is the delay relative to the preceding entry, not an absolute timestamp.

For wake times 3, 8, and 10 ticks from now, the stored delays are 3, 5, and 2. Inserting a wakeup at tick 6 yields delays 3, 3, 2, and 2. Only the first delay needs to decrease on each tick.

## Sleeping

`sleep` sets the running thread to `SLEEPING`, walks the list while subtracting preceding delays, inserts the thread, and subtracts its remaining delay from the successor. It then resets the time-slice counter and dispatches another thread.

Equal deadlines produce zero-delay followers. Insertion takes O(S) for S sleeping threads.

## Waking

`tick` returns immediately for an empty list. Otherwise it decrements the head and removes all leading zero-delay entries. Entries still marked `SLEEPING` become `READY` and enter the scheduler; entries with another state are discarded from the sleep list without requeueing.

A tick costs O(1 + W), where W is the number of entries removed. Wakeup makes a thread eligible to run; it does not promise immediate CPU access or deadline precision.

## Constraints

There is no separate removal/cancellation API, internal lock, or wraparound policy. Applying the general suspend/resume extension to a sleeping thread does not repair its list membership. Use normal sleep/wakeup transitions when relying on these invariants.

Related: [TCB](TCB.md), [Riscv](Riscv.md), [PeriodicThread](../api/PeriodicThread.md).
