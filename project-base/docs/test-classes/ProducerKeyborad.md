# ProducerKeyborad

[Documentation index](../README.md) · [Test guide](../build-and-testing.md)

[Source](../../test/ConsumerProducer_CPP_API_test.cpp)

**Role:** test/example class. **Entry point:** `testConsumerProducer() — menu option 6`.

## Responsibility and interface

A `Thread` subclass that supplies keyboard input to `BufferCPP`. The spelling `ProducerKeyborad` is the actual source identifier. Its constructor stores a borrowed `thread_data*`; public `run()` performs the work.

## Execution

`run` reads characters with `getc`, enqueues them until Escape (`0x1b`), sets the shared `threadEnd` flag, places `!` into the buffer as a final marker, and signals the completion semaphore. It does not explicitly yield or sleep; input and full-buffer waits can block it.

## State and lifetime

The borrowed record contains an id, buffer pointer, and completion semaphore pointer. The surrounding test keeps records in its stack frame and waits for completion signals before cleanup. This class owns none of those shared resources.

Its instance must survive its body, as with every `Thread` subclass. The marker supports test termination; it is not a general stream-closing protocol. See [Thread](../api/Thread.md) for the distinction between a completion signal and a formal join.
