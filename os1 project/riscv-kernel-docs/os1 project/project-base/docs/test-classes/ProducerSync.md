# ProducerSync

[Documentation index](../README.md) · [Test guide](../build-and-testing.md)

[Source](../../test/ConsumerProducer_CPP_Sync_API_test.cpp)

**Role:** test/example class. **Entry point:** `producerConsumer_CPP_Sync_API() — menu option 4`.

## Responsibility and interface

Generates id-based characters for the synchronous C++ producer/consumer test. The constructor borrows a `thread_data*`; `run()` delegates to private `producer(void*)`.

## Execution

While the shared stop flag is clear, it enqueues `id + '0'`. Every `10 * id` iterations it explicitly dispatches. Numeric producers are created for ids starting at one; id zero is reserved for `ProducerKeyboard`. After the stop flag is observed, it signals the shared completion semaphore.

## Dependencies and constraints

`BufferCPP` supplies item/space blocking and index synchronization. The `Thread` base supplies scheduling entry, and the data record supplies a borrowed completion semaphore.

No resource ownership is transferred to the producer. The enclosing test must keep the record and buffer alive. Its explicit yields are useful for exercising synchronous context switches, but it may also be preempted or blocked by other kernel mechanisms. A test completion signal is not a general thread-join API.
