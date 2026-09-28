# Consumer

[Documentation index](../README.md) · [Test guide](../build-and-testing.md)

[Source](../../test/ConsumerProducer_CPP_API_test.cpp)

**Role:** test/example class. **Entry point:** `testConsumerProducer() — menu option 6`.

## Responsibility and interface

A `Thread` subclass that consumes integers from `BufferCPP` and prints them as characters. Its constructor stores a borrowed `thread_data*`; public `run()` implements the loop.

## Execution

Until `threadEnd` becomes true, it blocks for an item, prints it through `Console::putc`, and inserts a newline every 80 consumed items. It then drains items while `getCnt()` reports a nonempty buffer and signals the completion semaphore.

## State and constraints

The data record supplies the shared buffer and completion semaphore; the consumer does not own either. Output is character-oriented despite the integer buffer. Buffer and console waits can both suspend execution.

The final `getCnt()` loop observes snapshots. Other producers can be finishing concurrently, so this code is a test-specific shutdown sequence, not a proof that an arbitrary producer pipeline has been completely drained. Preserve shared object lifetimes until every participant has stopped using them.
