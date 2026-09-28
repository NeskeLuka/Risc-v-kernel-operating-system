# WorkerC

[Documentation index](../README.md) · [Test guide](../build-and-testing.md)

[Source](../../test/Threads_CPP_API_test.cpp)

**Role:** test/example class. **Entry point:** `Threads_CPP_API_test() — menu option 2`.

## Responsibility and interface

One of four `Thread` subclasses in the C++ threading exercise. Its public default constructor uses the protected base constructor. Public `run()` forwards to private `workerBodyC(void*)` with a null argument.

## Execution

Prints indices 0 through 2, loads 7 into register `t1`, dispatches, prints the observed `t1`, computes recursive Fibonacci(12), prints indices 3 through 5, sets `finishedC`, and dispatches.

The register and recursive-call exercises help investigate saved execution state. The source's final message says “A finished” even though this is worker C.

## Ownership and test interpretation

The wrapper has no additional per-instance data fields. Completion is tracked through a file-local volatile flag, and the enclosing test yields until all four flags are set before deleting its wrapper objects.

A completion flag is application bookkeeping, not a kernel join. Interleaved output is expected, but ordering and timing are not guaranteed. The test helps inspect scheduling and context preservation; it does not prove correctness for every compiler optimization or lifetime edge case.

Related: [Thread](../api/Thread.md), [TCB](../classes/TCB.md), [Riscv](../classes/Riscv.md).
