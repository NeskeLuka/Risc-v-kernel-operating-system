# WorkerD

[Documentation index](../README.md) · [Test guide](../build-and-testing.md)

[Source](../../test/Threads_CPP_API_test.cpp)

**Role:** test/example class. **Entry point:** `Threads_CPP_API_test() — menu option 2`.

## Responsibility and interface

One of four `Thread` subclasses in the C++ threading exercise. Its public default constructor uses the protected base constructor. Public `run()` forwards to private `workerBodyD(void*)` with a null argument.

## Execution

Prints indices 10 through 12, loads 5 into `t1`, dispatches, computes recursive Fibonacci(16), prints indices 13 through 15, sets `finishedD`, and dispatches.

The Fibonacci helper itself yields when its argument is divisible by ten, adding context switches within recursion.

## Ownership and test interpretation

The wrapper has no additional per-instance data fields. Completion is tracked through a file-local volatile flag, and the enclosing test yields until all four flags are set before deleting its wrapper objects.

A completion flag is application bookkeeping, not a kernel join. Interleaved output is expected, but ordering and timing are not guaranteed. The test helps inspect scheduling and context preservation; it does not prove correctness for every compiler optimization or lifetime edge case.

Related: [Thread](../api/Thread.md), [TCB](../classes/TCB.md), [Riscv](../classes/Riscv.md).
