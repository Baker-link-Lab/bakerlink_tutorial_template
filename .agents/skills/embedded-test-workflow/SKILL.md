---
name: embedded-test-workflow
description: "Design, add, build, and run tests for no_std embedded Rust firmware, including host-testable logic and on-device embedded-test/probe-rs tests. Use when writing firmware tests, investigating test failures, or deciding what can be verified without hardware."
---

# Embedded Test Workflow

Separate deterministic logic checks from behavior that requires the MCU or physical peripherals. State clearly which level was actually verified.

## 1. Classify the behavior

- Inspect the package, target configuration, features, and existing tests before choosing a test command. This template defaults Cargo to `thumbv6m-none-eabi`; a host test may need a separate host-compatible crate/package or an intentional target override.
- Keep calculations, parsing, protocol/state transitions, and other hardware-independent logic testable without taking PAC peripherals where practical.
- Use `embedded-test` for target-side tests that need MCU behavior or peripherals. Do not replace it with `defmt-test`, or mix the two harnesses.
- Do not pretend physical bus/device behavior is covered by a pure unit test. State any fake, mock, or hardware assumptions.

## 2. Write focused tests

- Give each test one observable behavior and a meaningful failure condition. Cover boundary values, invalid input, error/timeout paths, and state transitions when relevant.
- Use embedded-test's per-test initialization to acquire/configure hardware needed by each test. Tests run on the real target and the runner resets between cases; keep initialization safe and repeatable.
- Make pin assignments and external wiring explicit. A test that drives outputs or accesses a bus can affect connected hardware; do not assume the board is disconnected or safe.
- Avoid unbounded waits. Use a bounded timeout or a clear failure path for events that may never occur.
- Keep tests independent where possible. Do not rely on another test's peripheral state or execution order.

## 3. Build and execute in distinct stages

- Run formatting and compile/check the relevant target and feature configuration first. For example, `cargo test --test example_test --no-run` checks that this template's embedded test target compiles; it does not run tests.
- To execute target tests, run the appropriate `cargo test` command from a terminal in the Dev Container where `probe-rs` can access the RP2040 probe. The configured `.cargo/config.toml` runner is `probe-rs run --chip RP2040 --protocol swd`.
- On-device test execution flashes and resets the target. Tell the user before doing this, confirm the intended board is available, and never claim execution if only compilation occurred.
- The Baker Link Env Agent tools currently expose build tasks, debug-session control/status, and source breakpoints; they do not expose a `cargo test` tool. Do not claim a test was run through those tools. Use an available VS Code task or terminal command, and report if neither is configured.
- If the probe, board, wiring, or runner is unavailable, finish compile-only checks where possible and mark hardware execution as blocked/unverified rather than silently substituting a host test.

## 4. Diagnose failures with evidence

- Preserve the failing test name, full error, target/features, probe/board, wiring, and relevant RTT output before changing code.
- Separate compile/link failures, runner/probe failures, target startup failures, assertion failures, and external-device communication failures.
- Change one likely cause at a time and rerun the narrowest relevant test. Do not weaken or remove an assertion merely to make the suite pass.
- Use the Baker Link debug workflow when source-level hardware investigation is needed. Remember that breakpoints perturb timing; use logs or external signal measurement for time-sensitive failures.

## Completion report

List each command and whether it compiled, ran on host, or ran on physical hardware. Include passing/failing test names, target/feature settings, hardware assumptions, and remaining unverified behavior.
