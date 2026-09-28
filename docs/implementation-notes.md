# Implementation Notes and Current Limitations

[Documentation index](README.md) · [Architecture](architecture.md)

These notes record behavior and risks visible in the supplied source. They distinguish implemented mechanisms from stronger guarantees the code does not establish. They are a static review, not a complete audit or a report of reproduced runtime failures. This documentation update does not change kernel behavior.

## Scope and assignment alignment

| Topic | Behavior in the supplied code |
| --- | --- |
| Memory context | Application and kernel share an address space; privilege transitions do not establish process isolation |
| Platform support | Course runtime libraries supply startup/hardware support; the repository is not a complete independent platform implementation |
| Allocation ABI | `a1` carries bytes; rounding happens in the trap handler, unlike the assignment's block-count ABI |
| Stack size | `DEFAULT_STACK_SIZE = 4096` is multiplied by `sizeof(uint64)`, yielding a 32 KiB stack request |
| Sleep units | Timer ticks; no public millisecond conversion |
| Periodic execution | Callback followed by relative sleep; no fixed absolute release schedule |
| Slot allocation | Linear scans of occupancy flags/chunks; no O(1) allocation guarantee |
| Console output | Dedicated worker busy-polls transmit readiness |
| Fault handling | Unhandled exceptions terminate the whole QEMU guest |
| Extended API | Multi-permit semaphore calls and suspend/resume exist; join and paired-thread operations do not |

The supplied assignment also distinguishes the GitHub repository from a course submission archive: its stated submission format contains only implementation sources and headers in `src/` and `inc/`. The full repository archive and documentation bundle are for GitHub use, not that restricted submission format.

## Thread lifetime and shutdown

**Reclaimed raw handles.** `TCB::dispatch` queues finished threads, and the collector deletes them. Bootstrap `main()` and several experiments poll `isFinished()` through raw handles that can already have been reclaimed. Some tests also explicitly delete those handles. Completion tracking should eventually be separated from reclaimable TCB storage or use a defined join/ownership protocol.

**Wrapper lifetime.** `Thread::~Thread()` is empty. An application object can be destroyed while its virtual `run()` still uses `this`. Completion of a work item and completion of the entire thread body are distinct events; signaling a semaphore just before return is not itself a kernel-level join.

**Empty ready queue.** If no replacement is found, dispatch marks the old thread running even if it was blocked or finished. The usual startup population reduces the chance of this path, but the fallback does not preserve all state invariants.

**Suspend/resume.** These operations do not track a separate explicit-suspension reason. Resume may enqueue a thread still linked to a semaphore queue, or a running/finished thread. Suspend does not remove sleeping threads from the timer list. Restricting legal transitions and coordinating list membership would be necessary before treating this extension as general-purpose cancellation/control.

## Semaphore destruction

`Sem::close` sets `closed` and moves waiters to the ready queue. Syscall `0x22` then immediately deletes the semaphore. A resumed `wait` reads `this->closed`, which may now refer to freed storage.

The intended “closed while waiting” error result therefore has a lifetime gap. A future correction needs to retain the object while waits unwind or store the wakeup result independently in each thread. This issue also affects wrapper destruction and buffer teardown when callers remain blocked.

The C++ `Semaphore` wrapper permits implicit copying of its owning handle and ignores construction failure. Applications should avoid copies and coordinate lifetimes carefully until ownership is made explicit.

## Allocation and alignment

`MemoryAllocator::mem_free` checks a few free-list conflicts but does not authenticate arbitrary pointers, fully validate heap bounds, or guard size arithmetic overflow. Invalid inputs can corrupt allocator metadata.

`SlotAllocator` stores objects in a plain `char` array. For default chunks under the target layout, the storage offset can fail the current types' 8-byte alignment. An explicitly aligned storage area and validated slot boundaries would remove that concern.

Allocation failure paths are incomplete. `TCB::createThread` dereferences the newly allocated pointer without checking it, constructors assume dependent allocations succeed, and the thread syscall wrapper does not comprehensively roll back a stack allocation after failure.

## Console buffering

**Full and empty look identical.** `BoundedBuffer` permits all capacity positions to be filled but tests emptiness with `head == tail`. A full ring also satisfies that equality. This affects shutdown's output-drain loop.

**Software empty is not device complete.** The output thread removes a byte before waiting for hardware readiness, so a truly empty ring can still have one byte pending transmission.

**Blocking from the receive handler.** The external-interrupt path calls the buffer's blocking `put`. If the ring is full, it can suspend the interrupted thread inside interrupt handling, before PLIC completion. A defined drop/defer/nonblocking policy is needed for robust input overflow behavior.

**Index serialization.** The kernel ring uses item/space semaphores but no index mutexes. Kernel-thread operations can interleave with trap-side operations. Availability counting alone does not prove index updates are protected for all callers.

## Interrupt and compiler assumptions

The intrusive queues and allocators have no general internal lock. The collector masks interrupts only around removal from `finishedThreads`, then deletes after restoring status. Direct kernel-thread operations therefore warrant separate preemption analysis even on one CPU.

The syscall assembly macro writes fixed argument registers without a complete clobber list or memory clobber. `Riscv::updateResult` relies on x8 and a specific generated frame layout. Compiler or optimization changes should be evaluated against disassembly and runtime tests before compatibility is claimed.

Unknown syscalls have no defined error response, and most handle arguments are not validated before dereference.

## Test interpretation

- Option 9 prints a message without running paired-thread logic.
- Option 8 computes matrix row sums using a semaphore; it does not exercise a join method.
- Several tests use fixed busy-loop delays or extra sleeps as coordination. Those are timing assumptions, not completion guarantees.
- C/C++ synchronous producer tests evaluate a remainder expression using `10 * id`, while the keyboard producer has id zero. This is a C++ zero-divisor defect, regardless of how a particular instruction sequence behaves on RISC-V.
- The allocation test's successful large request does not alone prove coalescing: another sufficiently large free region could satisfy it.
- Existing printed “success” messages and committed binaries do not substitute for a fresh build and observed test results.

The class pages describe each implemented algorithm in more detail and link to its source so these notes can be revisited as the code evolves.
