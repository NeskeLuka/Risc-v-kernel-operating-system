# BoundedBuffer

[Documentation index](../README.md) · [Repository README](../../../../README.md)

| Property | Details |
|---|---|
| Files | [`h/boundedBuffer.hpp`](../../h/boundedBuffer.hpp) · [`src/boundedBuffer.cpp`](../../src/boundedBuffer.cpp) |
| Kind | Instantiable class |
| Used by | [`MyConsole`](MyConsole.md) — input buffer and output buffer |
| Built on | [`Sem`](Sem.md), [`MemoryAllocator::kmalloc`](MemoryAllocator.md) |

## Responsibility

A kernel-side ring buffer of characters, used for console receive and transmit queues. It allocates its byte array directly from `MemoryAllocator` and uses kernel `Sem` objects for availability.

## Interface

| Operation | Behavior |
| --- | --- |
| `BoundedBuffer(int capacity)` | Allocates capacity bytes; initializes zero items and capacity free spaces |
| `put(char)` | Waits for a free space, writes at tail, advances tail, signals an item |
| `get()` | Waits for an item, reads at head, advances head, signals a space |
| `isEmpty()` | Returns `head == tail`; see the ambiguity below |
| `~BoundedBuffer()` | Frees the array and deletes both semaphores |

## Internal state

`buffer`, `cap`, `head`, and `tail` describe the circular array. `itemAvailable` counts readable characters and `spaceAvailable` counts writable positions. Index advancement uses modulo capacity.

The successful nonblocking data movement is O(1), plus any semaphore wakeup and scheduling work. A caller can block indefinitely if no counterpart produces or consumes data.

## Important invariants and limits

Capacity must be positive, and all allocations must succeed; the constructor does not validate these conditions. Semaphore initialization narrows capacity to `unsigned short`.

Because every one of the `cap` positions can be occupied, both a completely full ring and an empty ring have `head == tail`. Consequently `isEmpty()` cannot distinguish full from empty. Console shutdown uses this method, so it is not a reliable drain-completion test.

There are no head/tail mutexes. Availability semaphores do not by themselves serialize multiple concurrent writers or readers around index updates. The class depends on its calling context's serialization, which requires care because syscall, interrupt, and kernel-thread paths all use it.

Destroy only after all users have stopped; blocked calls and ignored semaphore failure returns make concurrent teardown unsafe. Calling `put` from the console interrupt handler when the ring is full can dispatch from an interrupt context, which is another current limitation.

Do not confuse this type with the test-side [Buffer](../test-classes/Buffer.md) and [BufferCPP](../test-classes/BufferCPP.md): those store integers, reserve an extra ring slot, and use separate index mutexes.
