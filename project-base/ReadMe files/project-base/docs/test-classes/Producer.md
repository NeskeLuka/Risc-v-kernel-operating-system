# Producer

[Documentation index](../README.md) · [Test guide](../build-and-testing.md)

[Source](../../test/ConsumerProducer_CPP_API_test.cpp)

**Role:** test/example class. **Entry point:** `testConsumerProducer() — menu option 6`.

## Responsibility and interface

A `Thread` subclass that generates repeated characters derived from its producer id. Construction stores a borrowed `thread_data*`; public `run()` drives the producer loop.

## Execution

While `threadEnd` is false, it writes `id + '0'` into the shared `BufferCPP`, increments a local iteration count, and sleeps for `(iteration + id) % 5` ticks. A zero result calls `sleep(0)`, which returns `-1`; it does not perform a positive sleep.

After termination is requested, the producer signals the shared completion semaphore. It may first need to finish a buffer operation or wake from sleep.

## Dependencies and constraints

The test provides the buffer, borrowed data record, and semaphore. This exercises relative sleep, blocking on full buffers, and multiple producers. Ids are rendered as single character offsets, so ids beyond decimal digits are not formatted as multi-digit numbers.

The shared flag is a test-level volatile flag, not a portable atomic stop token. Completion follows the existing test protocol and should not be mistaken for an implemented `Thread::join`.
