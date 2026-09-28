# Where to Put These Documentation Files

[Documentation index](README.md) · [Repository README](../README.md)

## Install from the ZIP

Open `riscv-kernel-docs.zip`. Its top level contains **`README.md` and `docs/`**. Copy both into your repository root: the directory that already contains `LICENSE` and `os1 project/`.

Replace the existing root `README.md`. Merge the included `docs/` directory into the existing `docs/` directory, replacing files with the same paths. If this is your first documentation installation, simply copy the whole directory.

Do **not** place `docs/` inside `os1 project/project-base/`; the relative source links expect it at the repository root. Do not rename every class page to `README.md`: names such as `TCB.md` and `Thread.md` are part of the link structure.

## Exact destinations

| File or folder in the ZIP | Destination in your repository | Purpose |
| --- | --- | --- |
| `README.md` | `<repo>/README.md` | GitHub landing page |
| `docs/README.md` | `<repo>/docs/README.md` | Documentation landing page and full class index |
| `docs/architecture.md` | `<repo>/docs/architecture.md` | Boot, trap, scheduling, memory, and I/O walkthrough |
| `docs/build-and-testing.md` | `<repo>/docs/build-and-testing.md` | Build commands, actual test menu, and caveats |
| `docs/implementation-notes.md` | `<repo>/docs/implementation-notes.md` | Current limitations and ownership/concurrency details |
| `docs/support-types.md` | `<repo>/docs/support-types.md` | Nested structures and test data records |
| `docs/INSTALL-DOCS.md` | `<repo>/docs/INSTALL-DOCS.md` | These placement instructions |
| `docs/classes/*.md` | `<repo>/docs/classes/` | Ten kernel-class pages |
| `docs/api/*.md` | `<repo>/docs/api/` | C API/ABI page and four C++ class pages |
| `docs/test-classes/*.md` | `<repo>/docs/test-classes/` | Eighteen test/example class pages |

The package contains **40 Markdown files**, including this guide. It contains documentation only; your source files, Makefile, libraries, binaries, and license files stay in their existing locations.

## If you installed the earlier 39-file documentation edition

That edition placed the C++ wrappers alongside kernel classes and used `docs/syscalls.md`. Their current locations are:

| Earlier location | Current location |
| --- | --- |
| `docs/classes/Thread.md` | `docs/api/Thread.md` |
| `docs/classes/Semaphore.md` | `docs/api/Semaphore.md` |
| `docs/classes/PeriodicThread.md` | `docs/api/PeriodicThread.md` |
| `docs/classes/Console.md` | `docs/api/Console.md` |
| `docs/syscalls.md` | `docs/api/CApi.md` |

After copying the new documentation, remove those five obsolete files if they exist, so readers do not encounter conflicting copies. Preserve any unrelated documentation you added yourself. The later 17-file edition already used `docs/api/`, so it does not require those moves.

## Verify the placement

Open the root README in GitHub, follow **Class reference**, and then open **TCB**. Its source link should reach `os1 project/project-base/h/tcb.hpp`. If your repository uses a different source directory, adjust the documentation's source links to that actual layout.

There is no need to rebuild the kernel to install Markdown documentation. The build/testing guide explicitly distinguishes static documentation checks from kernel runtime validation.
