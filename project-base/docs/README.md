# Documentation Index

[Repository README](../../README.md)

This reference describes the supplied implementation. API names and source identifiers retain their original spelling; explanations are in English. Nested structures are covered in their parent pages and the supporting-types reference.

## Guides

| Document | Purpose |
| --- | --- |
| [Architecture](architecture.md) | Boot sequence, trap entry, state transitions, memory, and I/O flows |
| [System calls](api/CApi.md) | Exact operation numbers, register arguments, returns, and C++ mappings |
| [Build and testing](build-and-testing.md) | Build directory, configuration, actual menu behavior, and validation status |
| [Implementation notes](implementation-notes.md) | Current lifetime, concurrency, ABI, and test limitations |
| [Supporting structures](support-types.md) | Nested kernel structures and test argument/payload records |

## Kernel and application API classes

| Class | Responsibility |
| --- | --- |
| [MemoryAllocator](classes/MemoryAllocator.md) | First-fit heap allocation and coalescing |
| [SlotAllocator](classes/SlotAllocator.md) | Chunk-based storage for kernel objects |
| [Queue](classes/Queue.md) | Intrusive FIFO of thread pointers |
| [Scheduler](classes/Scheduler.md) | Shared FIFO ready queue |
| [TCB](classes/TCB.md) | Thread state, stacks, switching, and deferred reclamation |
| [Timer](classes/Timer.md) | Relative-delay sleep list |
| [Sem](classes/Sem.md) | Kernel counting semaphores and blocked waiters |
| [BoundedBuffer](classes/BoundedBuffer.md) | Kernel character ring and availability semaphores |
| [MyConsole](classes/MyConsole.md) | Console buffers and transmit worker |
| [Riscv](classes/Riscv.md) | Trap routing, CSRs, and privilege transitions |
| [Thread](api/Thread.md) | Application thread wrapper |
| [Semaphore](api/Semaphore.md) | Application semaphore wrapper |
| [PeriodicThread](api/PeriodicThread.md) | Repeated activation with relative sleep |
| [Console](api/Console.md) | Character I/O wrapper |

## Test and example classes

These are test fixtures rather than additional kernel services. Each has a separate page describing its source, entry point, execution, and ownership assumptions.

| Class | Source group |
| --- | --- |
| [Buffer](test-classes/Buffer.md) | Application buffer |
| [BufferCPP](test-classes/BufferCPP.md) | Application buffer |
| [Consumer](test-classes/Consumer.md) | Producer/consumer tests |
| [ConsumerSync](test-classes/ConsumerSync.md) | Producer/consumer tests |
| [MatrixRowThread](test-classes/MatrixRowThread.md) | Matrix row computation |
| [MojPeriodicniRadnik](test-classes/MojPeriodicniRadnik.md) | Periodic-thread example |
| [Producer](test-classes/Producer.md) | Producer/consumer tests |
| [ProducerKeyboard](test-classes/ProducerKeyboard.md) | Producer/consumer tests |
| [ProducerKeyborad](test-classes/ProducerKeyborad.md) | Producer/consumer tests |
| [ProducerSync](test-classes/ProducerSync.md) | Producer/consumer tests |
| [ThreadA](test-classes/ThreadA.md) | Suspend/resume experiment |
| [ThreadB](test-classes/ThreadB.md) | Suspend/resume experiment |
| [ThreadC](test-classes/ThreadC.md) | Suspend/resume experiment |
| [ThreadD](test-classes/ThreadD.md) | Suspend/resume experiment |
| [WorkerA](test-classes/WorkerA.md) | C++ thread workers |
| [WorkerB](test-classes/WorkerB.md) | C++ thread workers |
| [WorkerC](test-classes/WorkerC.md) | C++ thread workers |
| [WorkerD](test-classes/WorkerD.md) | C++ thread workers |

## Suggested reading order

Read the architecture guide, then follow `Riscv → TCB → Scheduler/Queue → Sem/Timer`. Continue with the two allocators and console path. Finish with the four C++ wrappers and whichever test demonstrates the mechanism you want to inspect.

The supplied assignment informed terminology and scope; where code differs from its stated interface, the difference is explicit.
