---
name: embedded-io-workflow
description: "Plan, implement, bring up, and integrate embedded Rust peripheral I/O on RP2040, including GPIO, ADC, I2C, SPI, UART, DMA, PIO, interrupts, and external devices. Use when adding or debugging hardware I/O, sensors, buses, or peripheral-driven features."
---

# Embedded I/O Workflow

Use this workflow for hardware-facing changes. The goal is to prove a small I/O path on the actual hardware before integrating it into application behavior.

## 1. Establish facts before editing

- Inspect the workspace, `Cargo.toml`, `.cargo/config.toml`, `build.rs`, existing launch/tasks, and the relevant source. Record the exact board/chip and crate versions in use.
- Ask for missing hardware facts when they affect correctness: board revision, peripheral part number, pin wiring, voltage levels, pull-ups, clock rate, power/reset requirements, and expected behavior. Do not invent a pinout or assume an example's wiring matches the user's board.
- Prefer the peripheral datasheet, board schematic/pinout, and documentation for the exact crate version. Check electrical limits and bus-specific requirements before proposing wiring or enabling outputs.
- Choose the implementation model already used by the project. Do not mix `rp2040-hal` blocking/interrupt code with Embassy async peripheral ownership unless the user asks for a deliberate migration.

## 2. Find and adapt a relevant reference

- For `rp2040-hal`, search the [rp-hal RP2040 examples](https://github.com/rp-rs/rp-hal/tree/main/rp2040-hal-examples/src/bin) for the same peripheral and mode.
- For Embassy, search the [Embassy RP examples](https://github.com/embassy-rs/embassy/tree/main/examples/rp/src/bin) for the same peripheral, async/blocking mode, and task/resource pattern.
- Treat upstream `main` examples as design references, not drop-in code. Compare their dependency versions, feature flags, target, pin types, and APIs with this project's lockfile/manifest before adopting code.
- Check existing examples in this template for project-specific setup (boot2, clocks, logging, memory layout, and Baker Link workflows), but do not assume they cover every peripheral or board.
- Summarize which reference and assumptions apply before a broad change. Link to the upstream file rather than copying large code excerpts.

## 3. Prove the smallest useful I/O path

- Build a minimal path first: initialize only the required peripheral and pins, perform one bounded operation, and report its result or error. Keep application state machines and unrelated features out of this bring-up step.
- Make failures observable. Use concise `defmt`/RTT logs for initialization, state transitions, raw readings, and error categories. Avoid logging secrets or flooding time-critical paths.
- Handle errors explicitly. Do not turn bus failures, invalid readings, timeouts, or missing devices into plausible success values.
- For buses, verify address/mode/bit order/clock and required pull-ups or chip-select behavior. For GPIO, verify active level, pull mode, and debounce needs. For DMA/interrupt/PIO, verify buffer lifetime, ownership, completion/error paths, and interrupt clearing.
- If timing or signal integrity is in question, consider a GPIO timing marker and suitable external measurement. A debugger breakpoint halts execution and can change interrupt, DMA, and protocol timing.

## 4. Verify, then integrate

- Run formatting and the narrowest useful compile/check for the selected target and feature set. Do not report a build as successful unless its exit status is successful.
- Test hardware-independent conversion/state logic separately when the project structure allows it. Use the embedded-test workflow for target-dependent behavior; compilation of a test ELF is not an on-device test.
- Before starting hardware debug or running target tests, tell the user if the operation may connect to, program, reset, or otherwise affect a physical board. Confirm the intended target is safe to use.
- Verify the minimal I/O path on hardware when available. Record the observed log/signal and distinguish it from assumptions or unverified behavior.
- Only after bring-up succeeds, connect the peripheral to the main feature. Keep hardware access behind a clear boundary where practical; preserve HAL/Embassy ownership rules and avoid blocking an async executor.
- Re-run relevant checks and hardware observations after integration. Remove temporary instrumentation or explain why it remains.

## Completion report

Report the hardware assumptions, source/reference and version match, files changed, commands actually run, test/build outcomes, hardware observations, and anything not verified. Never describe a compile-only check as proof that a peripheral works.
