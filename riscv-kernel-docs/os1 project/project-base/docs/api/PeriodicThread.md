# PeriodicThread

[Documentation index](../README.md) · [Repository README](../../../../README.md)

| Property | Details |
|---|---|
| Files | [`h/syscall_cpp.hpp`](../../h/syscall_cpp.hpp) · [`src/syscall_cpp.cpp`](../../src/syscall_cpp.cpp) |
| Base class | [`Thread`](Thread.md) |
| Built on | `time_sleep` → [`Timer`](../classes/Timer.md) |

## Responsibility

Extends `Thread` with a repeated activation callback separated by a relative sleep interval. Derived application classes implement `periodicActivation()` instead of a complete scheduling loop.

## Interface

| Operation | Behavior |
| --- | --- |
| Protected `PeriodicThread(time_t period)` | Stores the interval in timer ticks |
| Protected virtual `periodicActivation()` | One activation; default body is empty |
| Protected `run()` override | Repeatedly activates and sleeps while period is positive |
| `terminate()` | Sets period to zero |
| Inherited `start()` | Creates the underlying thread |

## Execution model

The loop calls `periodicActivation()` immediately on its first run, then calls `Thread::sleep(period)`. After resumption it checks the loop condition and repeats.

This is **fixed-delay execution**: the separation between activation starts includes callback execution time, the sleep interval, and scheduling delay. It does not compensate for work duration or maintain an absolute release deadline, and it is not a hard real-time scheduler.

```cpp
class Heartbeat : public PeriodicThread {
public:
    Heartbeat() : PeriodicThread(10) {}
protected:
    void periodicActivation() override {
        Console::putc('.');
    }
};
```

The example requests ten ticks of sleep after each activation. It does not mean ten milliseconds.

## Termination and lifetime

`terminate` is cooperative: it changes a field and neither removes the thread from the sleep list nor waits for the running callback to return. An already sleeping thread must wake normally before observing the zero period. If termination occurs during an activation, the following sleep may receive zero and return `-1`; the loop then ends.

A zero initial period executes no activations. An instance cannot be restarted through inherited `start()` after its handle has been created.

Keep the wrapper alive until execution has finished; arbitrary extra sleep is not a formal join. The period field is not atomic, so the class should not be advertised as providing a portable C++ cross-thread memory-ordering contract.

Related: [Thread](Thread.md), [Timer](../classes/Timer.md), [MojPeriodicniRadnik](../test-classes/MojPeriodicniRadnik.md).
