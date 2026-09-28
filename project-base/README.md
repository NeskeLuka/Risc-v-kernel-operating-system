# Kernel Source and Documentation

[Project overview](../README.md) · [Documentation index](docs/README.md) · [Architecture](docs/architecture.md) · [System calls](docs/api/CApi.md)

This directory contains the RISC-V kernel sources, supplied platform libraries, tests, and component documentation.

## Source map

| Directory | Contents |
| --- | --- |
| [h/](h/) | Kernel declarations and C/C++ API headers |
| [src/](src/) | Kernel implementation, syscall wrappers, and assembly |
| [lib/](lib/) | Supplied platform headers and libraries |
| [test/](test/) | Interactive test entry point and test fixtures |
| [myTests/](myTests/) | Additional development experiments |
| [docs/](docs/README.md) | Class references, architecture, testing, and implementation notes |

## Build and run

From this directory:

```bash
make
make qemu
```

See the [build and testing guide](docs/build-and-testing.md) for prerequisites, toolchain selection, the test menu, and debugger setup limitations.

## Core components

[Traps and CSRs](docs/classes/Riscv.md) · [Threads](docs/classes/TCB.md) · [Scheduling](docs/classes/Scheduler.md) · [Semaphores](docs/classes/Sem.md) · [Memory allocation](docs/classes/MemoryAllocator.md) · [Console](docs/classes/MyConsole.md)

Each class page links directly to its corresponding header and implementation files. Known implementation constraints are described in the [implementation notes](docs/implementation-notes.md).
