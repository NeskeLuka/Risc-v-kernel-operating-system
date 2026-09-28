# ThreadD

[Documentation index](../README.md) · [Test guide](../build-and-testing.md)

[Source](../../test/ThreadSuspended.cpp)

**Role:** test/example class. **Entry point:** `threadSuspendChain() — manually selected`.

## Responsibility and interface

A `Thread` subclass participating in the explicit suspend/resume experiment. Constructor: `ThreadD(Thread* a, Semaphore* sem)`. The work runs in a protected `run()` override.

## State and execution

Borrowed target `a` and completion semaphore.

Runs a longer busy loop, resumes A, and signals completion.

The enclosing function constructs C, A, B, and D, starts them in A/B/C/D order, then waits for four completion signals. It is compiled with the source tree but is not selected by the current menu.

## Constraints

The intended ordering relies on relative busy-loop durations and scheduler timing. It is not enforced by a complete handshake, so the experiment can expose races rather than provide a deterministic suspension protocol.

The kernel uses one `BLOCKED` state for both explicit suspension and semaphore waiting, and resume accepts states that can violate queue membership. Do not generalize this example to sleeping, semaphore-blocked, or finished targets. The function also does not delete its allocated wrappers and semaphore.

Related: [Thread](../api/Thread.md), [TCB](../classes/TCB.md), [implementation notes](../implementation-notes.md).
