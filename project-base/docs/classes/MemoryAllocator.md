# MemoryAllocator

[Documentation index](../README.md) · [Repository README](../../../README.md)

| Property | Details |
|---|---|
| Files | [`h/memoryAllocator.hpp`](../../h/memoryAllocator.hpp) · [`src/memoryAllocator.cpp`](../../src/memoryAllocator.cpp) |
| Kind | Static utility (constructors deleted) |
| Heap | `[HEAP_START_ADDR, HEAP_END_ADDR)` — provided by `hw.lib` |
| Block size | `MEM_BLOCK_SIZE` = **64 bytes** (assignment allows 64…1024) |
| Collaborates with | [`Riscv`](Riscv.md) (friend: syscalls `0x01`/`0x02`), [`SlotAllocator`](SlotAllocator.md), [`TCB`](TCB.md) (stacks), [`BoundedBuffer`](BoundedBuffer.md) |

## Responsibility

Manages the raw heap exposed by `HEAP_START_ADDR` and `HEAP_END_ADDR`. It provides variable-size allocations for user data, thread stacks, console storage, and slot-allocator chunks. It is a static service: construction and copying are disabled.

## Interface and units

| Operation | Visibility | Behavior |
| --- | --- | --- |
| `init()` | Public | Initializes one free region; repeated calls do nothing |
| `kmalloc(size_t size)` | Public, kernel-facing | Accepts **bytes**, adds header overhead, rounds to allocation blocks |
| `mem_alloc(size_t size)` | Private; accessible to `Riscv` | Accepts a **block count**, including header space |
| `mem_free(void* ptr)` | Public | Reinserts and coalesces a previously allocated region |

The internal `mem_alloc` source comment says bytes, but the arithmetic operates in blocks. The C API accepts bytes; the trap handler performs its conversion. `MEM_BLOCK_SIZE` is 64 bytes.

## Representation

`freeMemHead` points to an address-ordered singly linked list. Each region starts with a nested `BlockHeader` containing `next` and `size`. `size` counts the entire region in blocks, including its header. An allocated region keeps its size header immediately before the returned payload.

`initialized` guards initialization. Heap boundaries are supplied externally; this class does not acquire memory from another OS.

## Allocation algorithm

1. Scan from `freeMemHead` for the first region large enough.
2. If it is larger than requested, place a new free header at the remainder's start.
3. Replace or remove the selected list entry.
4. Return the address just after its header.

For a 100-byte request, a 16-byte RV64 header produces `ceil((100 + 16) / 64) = 2` blocks: 128 bytes reserved, with 112 bytes after the header. Allocation takes O(F) in the number of free regions.

## Release algorithm and results

`mem_free` subtracts the header size, finds the address-ordered insertion point, and merges with the following and preceding regions when adjacent. Search is O(F); the local merge operations are O(1).

| Result | Meaning in this implementation |
| --- | --- |
| `0` | Region inserted successfully |
| `-1` | Null pointer |
| `-2` | Header equals an existing free-list entry or overlaps the previous free region |

Allocation returns null if the internal block request is zero or no suitable region exists. `kmalloc(0)` returns null; the user syscall path adds header space before allocation, so `mem_alloc(0)` through that API can allocate a header-sized region.

## Ownership and constraints

Return only original payload pointers to `mem_free`. It does not fully validate heap bounds, arbitrary pointers, or all double-free cases. Overflow checks and explicit allocator locking are absent. Kernel callers must provide appropriate serialization; trap entry alone does not cover every direct kernel-thread call.

Global C++ allocation uses the memory syscalls. Only [TCB](TCB.md) and [Sem](Sem.md) override allocation to use [SlotAllocator](SlotAllocator.md).
