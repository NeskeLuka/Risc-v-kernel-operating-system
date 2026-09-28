# ConsumerSync

[Documentation index](../README.md) · [Test guide](../build-and-testing.md)

[Source](../../test/ConsumerProducer_CPP_Sync_API_test.cpp)

**Role:** test/example class. **Entry point:** `producerConsumer_CPP_Sync_API() — menu option 4`.

## Responsibility and interface

Consumes and prints buffered values in the synchronous C++ producer/consumer test. Its constructor borrows a `thread_data*`; public `run()` delegates to private `consumer(void*)`.

## Execution

The consumer repeatedly removes a value and writes it with `putc`. It dispatches every `5 * id` items and emits a newline every 80 items. The enclosing test assigns its id to the positive producer count, so that dispatch divisor is nonzero for accepted input.

Once `threadEnd` is observed, it drains while the buffer's count snapshot is positive and signals the completion semaphore. It does not own or close the shared buffer.

## What to learn from it

This example combines blocking synchronization with voluntary dispatch. It demonstrates why an application-level queue and a console-output queue can independently suspend the same thread. Shutdown remains test-specific: a count snapshot and a stop flag do not automatically synchronize arbitrary outstanding producers.
