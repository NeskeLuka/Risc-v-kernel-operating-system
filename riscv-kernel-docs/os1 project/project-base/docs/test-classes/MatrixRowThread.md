# MatrixRowThread

[Documentation index](../README.md) · [Test guide](../build-and-testing.md)

[Source](../../test/MakeMatrix.cpp)

**Role:** test/example class. **Entry point:** `makeMatrix() — menu option 8`.

## Responsibility and interface

A `Thread` subclass that computes the sum of one matrix row. Its constructor accepts `int* row`, an element count, `int* rowSum`, and `Semaphore* doneSem`; protected `run()` performs the calculation.

## State and algorithm

All pointers are borrowed. `run()` accumulates the first `cols` integers, stores the result through `rowSum`, and signals the shared semaphore. Computation is O(cols) and needs one local accumulator.

The enclosing example starts five workers for a 5 × 4 matrix, waits five times on a semaphore initialized to zero, and adds the row results. The rows sum to 4, 4, 8, 8, and 4, giving **28**.

## Lifetime and synchronization

Each worker writes a distinct output slot, so the row computations do not require a shared accumulator lock. The semaphore coordinates availability of results. The matrix, output array, wrappers, and semaphore must remain alive while workers use them.

The menu calls this a join example, but it uses completion signaling and has no `Thread::join` method. A signal immediately before returning still does not prove the entire thread wrapper has exited.
