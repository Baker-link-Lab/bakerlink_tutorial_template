---
name: embedded-test-workflow
description: "Design and verify no_std embedded Rust tests with Baker Link Env Agent tools: execute host tests, build embedded-test ELF files, and distinguish compilation/debug launch from on-device test results. Use when writing firmware tests, selecting host targets, or diagnosing test failures."
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

- Use `bakerInspectProject` to discover packages, target names, and `hostTriple`. Read source/manifest before choosing features and test kind. Do not assume this no_std firmware package can run host tests unchanged.
- **Host tests:** use `bakerRunTests` with `mode: "host"`, `kind: "lib"`, `"bin"`, or `"test"`, the appropriate package/name, and `targetTriple` equal to the discovered host triple. `name` is required except for `lib`. Optional `testFilter` selects a test-name filter, not arbitrary runner options. Read `success`, `exitCode`, `stdout`, and `stderr`; a host failure is not an MCU test failure.
- **Embedded compilation:** use `bakerRunTests` with `mode: "embedded-build"`, `kind: "test"`, `name: "example_test"` for this template (or a discovered target), package/features/profile as needed. `kind: "lib"` is also supported for library test harnesses; no `testFilter` is accepted. Omit `targetTriple` to respect the embedded Cargo default. This uses `--no-run` and returns diagnostics and `artifact.artifactId`, not test pass/fail.
- **Source-level hardware investigation:** pass that artifact ID to `bakerStartDebugging` using `launchWithArtifact` and an existing configuration, or `direct` with explicit hardware conditions. Follow the [Baker Link debug workflow](../baker-link-debug-workflow/SKILL.md). Build or debug launch of a test ELF does not prove its assertions ran or passed; harness-specific result collection is not implemented in extension 0.2.0.
- Keep the project's `embedded-test` harness. Do not switch to `defmt-test` just to accommodate tool limitations. The template's probe-rs runner line is commented out; do not assume a configured runner or container USB access exists.
- Baker Link reaches host probe-rs without container probe-rs. Only use a separate terminal-based on-device test runner if the user explicitly chooses that route and its runner, USB access, target, and safety have been verified. Do not use it as an automatic fallback for disconnected/unavailable Agent tools.
- On-device execution can flash/reset hardware. Wait for approval, and report actual test names/assertion output only if available from a supported runner or user-provided logs. Otherwise finish compile checks and mark hardware test execution/results blocked or unverified, rather than substituting host tests.

## 4. Diagnose failures with evidence

- Preserve the failing test name, full error, target/features, probe/board, wiring, and relevant RTT output before changing code.
- Separate compile/link failures, runner/probe failures, target startup failures, assertion failures, and external-device communication failures.
- Change one likely cause at a time and rerun the narrowest relevant test. Do not weaken or remove an assertion merely to make the suite pass.
- Use the Baker Link debug workflow when source-level hardware investigation is needed. Remember that breakpoints perturb timing; use logs or external signal measurement for time-sensitive failures.

## Completion report

List each command and whether it compiled, ran on host, or ran on physical hardware. Include passing/failing test names, target/feature settings, hardware assumptions, and remaining unverified behavior.
