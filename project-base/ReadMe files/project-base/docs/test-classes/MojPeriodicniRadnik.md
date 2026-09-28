# MojPeriodicniRadnik

[Documentation index](../README.md) · [Test guide](../build-and-testing.md)

[Source](../../myTests/PeriodThreadTest.cpp)

**Role:** test/example class. **Entry point:** `periodicMain() — manually selected`.

## Responsibility and interface

A sample `PeriodicThread` subclass. Its original identifier roughly means “my periodic worker”; documentation is in English while preserving the source name. Construction forwards the period and initializes private counter `brojac` to zero.

## Activation

`periodicActivation()` prints an activation message, the last decimal digit of the counter, and a newline, then increments the counter. It uses `printStringPeriodic` and `Console::putc`.

The example creates a five-tick worker, starts it, sleeps for 20 ticks, calls `terminate`, sleeps ten more ticks, and deletes the wrapper. Actual code uses 20 ticks despite a nearby comment mentioning 22.

## Constraints

The base class executes a callback followed by relative sleep. Output time and scheduling latency therefore affect the number and timing of activations. The additional ten-tick delay is not a formal join guarantee.

The start-failure branch deletes the worker but does not exit the function before later accesses. This example should not be treated as a complete allocation-failure handling pattern. See [PeriodicThread](../api/PeriodicThread.md).
