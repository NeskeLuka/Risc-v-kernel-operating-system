# ProducerKeyboard

[Documentation index](../README.md) · [Test guide](../build-and-testing.md)

[Source](../../test/ConsumerProducer_CPP_Sync_API_test.cpp)

**Role:** test/example class. **Entry point:** `producerConsumer_CPP_Sync_API() — menu option 4`.

## Responsibility and interface

A `Thread` subclass for the synchronous C++ producer/consumer exercise. The constructor stores a borrowed `thread_data*`; `run()` forwards to private `producerKeyboard(void*)`.

## Execution

The helper reads and enqueues keyboard characters until Escape, then sets `threadEnd`, enqueues `!`, and signals the completion semaphore. It also attempts explicit dispatch after every `10 * id` input characters.

## Source caveat

The surrounding test assigns the keyboard producer id zero. The expression `i % (10 * id)` therefore has a zero divisor in C++, so this path must not be described as a reliably validated scheduling example. The same pattern appears in the C-style producer test.

## Ownership

The record, buffer, and completion semaphore belong to the enclosing test. Keep them and the wrapper object alive while the body runs. The synchronous designation refers to explicit yield points; the kernel still has timer-driven preemption enabled.
