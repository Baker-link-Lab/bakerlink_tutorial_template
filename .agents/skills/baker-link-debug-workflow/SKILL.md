---
name: baker-link-debug-workflow
description: "Build and debug RP2040 firmware on physical hardware using Baker Link Env VS Code Agent tools, probe-rs, RTT, and source breakpoints. Use when launching a Baker Link session, investigating MCU behavior, setting breakpoints, or diagnosing probe/debug issues."
---

# Baker Link Hardware Debug Workflow

Use the existing Baker Link launch configuration and Agent tools to investigate one observable hardware question at a time. Debug launch can affect a physical device.

## Available tools and limits

The Baker Link Env extension currently exposes these VS Code Copilot Agent tools:

- `baker-link-env_build`: run only the configured debug build, release build, or `cargo check` task.
- `baker-link-env_start_debugging`: start an existing `baker-link-debug` launch configuration. Its pre-launch task and ELF upload run as configured.
- `baker-link-env_debug_status` and `baker-link-env_stop_debugging`: inspect or stop this workspace's Baker Link debug session.
- `baker-link-env_set_breakpoint`, `baker-link-env_list_breakpoints`, and `baker-link-env_remove_breakpoint`: manage source-line breakpoints inside the workspace.

These are VS Code Language Model Tools, not a general-purpose MCP interface. They do not run arbitrary shell commands or `cargo test`, return target variable/register values, or report whether a breakpoint was actually hit. Use the VS Code Debug view/Debug Console, RTT output, and terminal tasks as needed. Do not claim an unavailable observation or action occurred.

## Procedure

1. **Frame the observation.** Ask what board and configuration are active, how to reproduce the issue, what should happen, and what actually happens. Identify the relevant source path and observable state/transition. If a physical target could be driven unsafely, stop and clarify its wiring/load first.
2. **Check project setup.** Inspect `.vscode/launch.json`, `.vscode/tasks.json`, `.cargo/config.toml`, target/features, ELF path/artifact profile, and host connection settings. Use `debug_status` before starting another session. Do not invent a launch configuration or start a duplicate session.
3. **Build narrowly.** Use the `build` tool for one of its supported configured modes. For examples, alternate features, or tests, use a configured task or terminal command because the Agent build tool only supports the named standard build/check tasks. Report the exact command/task and outcome.
4. **Place purposeful breakpoints.** Read the source and select the smallest number of lines that distinguish the current hypotheses. Use `set_breakpoint` with a workspace-relative path and one-based line; inspect with `list_breakpoints`. Do not add speculative breakpoint sets or assume the target will hit a line.
5. **Confirm before hardware effects.** Starting debug may connect to the probe, upload firmware, program or reset the MCU, and run a pre-launch build. Explain this and confirm the target is the intended board and can be interrupted/programmed. Then start the existing configuration with `start_debugging`.
6. **Observe in VS Code.** Use the Debug view/Debug Console for pause state and variables, and RTT for logs. The Agent tool's `debug_status` only indicates session status; it does not expose target execution state or values. Capture the actual observation and compare it with the hypothesis.
7. **Account for perturbation.** A breakpoint halts the processor and may change interrupt, DMA, Embassy scheduling, watchdog, and bus timing. If the bug disappears or changes, remove the breakpoint and investigate with RTT logs, GPIO timing markers, or an external logic analyzer/oscilloscope instead.
8. **Iterate and clean up.** Change one cause at a time, rebuild, and reproduce. When done, remove temporary breakpoints with `remove_breakpoint`, confirm session status, and stop the session when appropriate. Do not stop unrelated debug sessions.

## Common failure categories

- **Build/link failure:** inspect the actual task output, target, features, linker script, and ELF path before attempting hardware operations.
- **No host connection:** use Baker Link's connection check and verify the container-to-host endpoints/firewall. A TCP check does not prove USB probe readiness.
- **Probe or target not found:** verify host DAP service, USB/probe visibility on the host, chip selection (`RP2040`), SWD wiring, power, and reset state. Do not infer probe readiness from a successful network check.
- **ELF/profile mismatch:** make the build task, `bakerLink.elf`, and `bakerlink://<project>/<profile>` artifact identifier agree.
- **No breakpoint hit:** verify the source line is executable in the loaded ELF and that execution reaches it; the tool only sets a source breakpoint.

## Completion report

Report the selected launch configuration, build outcome, breakpoint locations, observed VS Code/RTT evidence, changes made, and whether the physical target was actually programmed and exercised. Separate observations from hypotheses and list any remaining uncertainty.
