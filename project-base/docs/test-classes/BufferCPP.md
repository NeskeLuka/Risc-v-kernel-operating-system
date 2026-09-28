# BufferCPP

[Documentation index](../README.md) · [Test guide](../build-and-testing.md)

[Source](../../test/buffer_CPP_API.cpp)

**Role:** test/example class. **Entry point:** `producer/consumer tests`.

## Responsibility

An integer ring buffer used by application tests, synchronized through C++ `Semaphore` objects. This is distinct from the kernel's character-only `BoundedBuffer`.

[Declaration](../../test/buffer_CPP_API.hpp)

## Interface and representation

`BufferCPP(int capacity)` allocates an integer array of `capacity + 1` entries but grants only `capacity` space permits. `put(int)` waits for space, locks the tail, writes and advances it, then signals an item. `get()` performs the complementary operation under the head lock and returns an integer.

`getCnt()` acquires the head mutex followed by the tail mutex, calculates the wrapped index distance, and releases them in reverse order. Its count is a snapshot, not a promise that a later `get` cannot block.

The four synchronization objects represent available spaces, available items, exclusive head access, and exclusive tail access. Keeping one unused array position distinguishes empty from full, unlike the kernel ring's equality-only check with all slots usable.

## Ownership and teardown

The buffer owns its array and synchronization objects. Its destructor prints a deletion message and remaining buffered characters, frees storage, and closes/deletes the synchronization objects. That output is test instrumentation, not a silent container contract.

Destroy only after producers and consumers have stopped. Construction does not robustly recover from allocation failure or invalid capacity, and semaphore teardown inherits the kernel's lifetime restrictions. Implicit copying would duplicate owning pointers/handles and is unsafe.

## What it exercises

Each nonblocking operation uses O(1) ring arithmetic, with additional syscall/scheduling costs and potentially unbounded blocking. The tests exercise memory allocation, ordinary semaphore operations, concurrent producers, and console output. They do not establish safety under arbitrary destruction or handle misuse.
