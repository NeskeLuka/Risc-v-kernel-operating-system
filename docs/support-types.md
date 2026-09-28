# Supporting Structures

[Documentation index](README.md)

Small nested and test-side structures are documented here so they are not mistaken for additional public service classes.

## Kernel implementation structures

| Structure | Definition | Fields and purpose |
| --- | --- | --- |
| `MemoryAllocator::BlockHeader` | [memoryAllocator.hpp](../os1%20project/project-base/h/memoryAllocator.hpp) | `next`, `size`; free-list linkage and region size in blocks |
| `SlotAllocator<T, CHUNK_SIZE>::Chunk` | [slotAllocator.hpp](../os1%20project/project-base/h/slotAllocator.hpp) | Next chunk, used count, occupancy flags, raw object storage |
| `TCB::Context` | [tcb.hpp](../os1%20project/project-base/h/tcb.hpp) | `ra`, `sp`; two words exchanged by context-switch assembly |

Their full algorithms and constraints appear in [MemoryAllocator](classes/MemoryAllocator.md), [SlotAllocator](classes/SlotAllocator.md), and [TCB](classes/TCB.md). In particular, `Context` is distinct from the complete trap frame stored on the stack.

## Producer/consumer argument records

Each producer/consumer source defines a `thread_data` record. These records borrow resources owned by the test driver and must remain valid while worker bodies access them.

| Source | Fields |
| --- | --- |
| [ConsumerProducer_C_API_test.cpp](../os1%20project/project-base/test/ConsumerProducer_C_API_test.cpp) | `int id`, `Buffer* buffer`, `sem_t wait` |
| [ConsumerProducer_CPP_Sync_API_test.cpp](../os1%20project/project-base/test/ConsumerProducer_CPP_Sync_API_test.cpp) | `int id`, `BufferCPP* buffer`, `Semaphore* wait` |
| [ConsumerProducer_CPP_API_test.cpp](../os1%20project/project-base/test/ConsumerProducer_CPP_API_test.cpp) | `int id`, `BufferCPP* buffer`, `Semaphore* sem` |

They are separate source-level definitions using the same global type name, not one shared header-defined API type. Differing global definitions are an ODR concern when linked together; separate namespaces or a consistently shared declaration would clarify that boundary.

## Allocation test structures

Defined in [myTests/testZaAlokaciju.cpp](../os1%20project/project-base/myTests/testZaAlokaciju.cpp):

- `Employee` contains an id, a 32-character name array, a double salary, and an active flag. The test allocates 50 records and checks selected id/flag values; it does not perform floating-point calculations.
- `ListNode` contains a `uint64` value and a next pointer. The test builds and verifies a 200-node chain, then frees each original allocation.

These are payloads used to exercise the allocator, not kernel-managed object types. The memory test also creates fragmented holes and requests a larger allocation after freeing adjacent blocks. Successful allocation alone does not establish which free region supplied that request.
