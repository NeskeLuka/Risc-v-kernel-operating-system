# SlotAllocator

[Documentation index](../README.md) · [Repository README](../../../README.md)

| Property | Details |
|---|---|
| File | [`h/slotAllocator.hpp`](../../h/slotAllocator.hpp) (header-only template) |
| Kind | Static utility template; one independent pool per instantiated `T` |
| Used by | `TCB::operator new/delete`, `Sem::operator new/delete` |
| Backed by | [`MemoryAllocator::kmalloc`](MemoryAllocator.md) / `mem_free` |

## Responsibility

`SlotAllocator<T, CHUNK_SIZE = 64>` supplies reusable fixed-size storage for kernel objects. `TCB` and `Sem` use separate template instantiations in their class-specific `new` and `delete` operators. This is a header-only allocator, not the global C++ heap allocator.

## Interface

| Operation | Behavior |
| --- | --- |
| `allocateSlot()` | Finds an unused slot or allocates a new chunk; returns null if chunk allocation fails |
| `deallocateSlot(void* ptr)` | Marks a slot free and releases its chunk when no slots remain; null is ignored |

These operations manage storage only. C++ `new` invokes the object's constructor separately; `delete` invokes its destructor before releasing the slot.

## Chunk representation

The nested `Chunk` stores a next pointer, a used-slot count, `CHUNK_SIZE` boolean occupancy flags, and `CHUNK_SIZE * sizeof(T)` bytes of object storage. A static `head` is maintained independently for each template specialization.

Allocation scans chunks from the head and occupancy flags from index zero. When all existing slots are occupied, it obtains a chunk through `MemoryAllocator::kmalloc`, prepends it, and reserves slot zero.

Deallocation searches for the chunk whose data range contains the pointer, derives the slot index, clears its flag, and decrements `usedCnt`. An empty chunk is unlinked and returned through `MemoryAllocator::mem_free`.

## Complexity

Allocation is O(C × S) in the worst case for C chunks and S slots per chunk, plus the underlying heap search when a chunk is added. With the default S = 64, it remains linear in the number of chunks. Deallocation searches O(C), with another heap search when freeing an empty chunk. The implementation is **not constant-time**.

## Constraints

Callers must supply exactly one live slot pointer on release. Interior pointers are accepted by the range/index calculation, and double release is not checked. There is no internal lock.

The storage uses a `char` array without `alignas(T)`. In the default RV64 layout its offset can be 76 bytes, which does not guarantee the 8-byte alignment required by the current pointer-containing object types. The template should not be described as alignment-safe for arbitrary `T`; this is a source-level limitation, not a runtime result established here.

Related: [MemoryAllocator](MemoryAllocator.md), [TCB](TCB.md), [Sem](Sem.md).
