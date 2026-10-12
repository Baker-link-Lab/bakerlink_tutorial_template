---
name: baker-link-debug-workflow
description: "Build and debug embedded Rust binaries, examples, and test ELF files on RP2040 using Baker Link Env Agent tools. Use when selecting launch configurations, substituting artifacts, setting breakpoints, investigating MCU behavior, or diagnosing build/probe/debug failures."
---

# Baker Link Hardware Debug Workflow

Use Baker Link Env extension 0.2.0 or later to investigate one observable hardware question at a time. Debug launch can affect a physical device. Do not move example code into the main binary or rewrite project settings merely to make a tool run.

## Available tools and limits

Use the actual available tool names; the chat reference names below correspond to the extension's `baker-link-env_*` tools.

| Chat tool                                                             | Purpose                                                                                                        |
| --------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| `bakerInspectProject`                                                 | Discover Cargo packages, target names/source paths, host triple, and saved launch configurations.              |
| `bakerBuild`                                                          | Run a configured `cargo: build`, `cargo: build (release)`, or `cargo: check` task.                             |
| `bakerBuildArtifact`                                                  | Build a selected binary/example or compile a test/lib target without running it; return `artifact.artifactId`. |
| `bakerRunTests`                                                       | Execute host tests or compile embedded test ELF files; see the embedded-test workflow.                         |
| `bakerStartDebugging`                                                 | Start using `launch`, `launchWithArtifact`, or `direct` mode.                                                  |
| `bakerDebugStatus`, `bakerStopDebugging`                              | Inspect session presence or stop Baker Link sessions in the selected workspace.                                |
| `bakerSetBreakpoint`, `bakerListBreakpoints`, `bakerRemoveBreakpoint` | Add, list, or remove source breakpoints at a workspace file and one-based line.                                |

These are VS Code Language Model Tools, not a standalone MCP server. The extension runs inside the Dev Container and connects to host probe-rs; container-side probe-rs or direct USB access is not required for this route. A connected VS Code client and trusted workspace are required.

Results are JSON text. Read the actual `success`, diagnostics, artifact IDs, and session results; do not treat a completed tool call as proof of success. Tools do not expose RTT/Debug Console logs, target variables/registers, breakpoint verification/hits, pause state, stack traces, Continue, or Step. Session presence does not prove successful flashing, breakpoint arrival, or peripheral correctness.

## Select the launch mode

- **`launch` (default):** select an existing configuration by name. Preserve its hardware settings, ELF, platform overrides, and `preLaunchTask`; use this for the normal configured program.
- **`launchWithArtifact`:** build the desired target first, then pass its `artifact.artifactId` and an existing configuration name. Inherit hardware settings, substitute the ELF and artifact URI, and skip `preLaunchTask`. Use this for an example/test on the existing board without rewriting launch/tasks. Requires exactly one core and does not support `remoteServerMode: true`.
- **`direct`:** pass an artifact ID and explicitly known `chip`, `dapServer`, and `uploadUrl`. Optional `flash`, `haltAfterReset`, and `rttEnabled` default to true. Do not guess endpoints/chip or silently fall back to this mode when a named configuration is missing.

Saved launch settings are read as JSONC; save intended changes before use. Invalid settings must be corrected, not bypassed. If multiple configurations or Cargo targets are possible, select explicitly rather than assuming.

For example, build a discovered Embassy example with `bakerBuildArtifact`:

```json
{
  "kind": "example",
  "name": "embassy_blinky",
  "features": ["embassy"],
  "noDefaultFeatures": true,
  "profile": "debug"
}
```

Then use `bakerStartDebugging`:

```json
{
  "mode": "launchWithArtifact",
  "configuration": "Baker Link: Debug",
  "artifactId": "<artifact.artifactId returned by the successful build>"
}
```

Supply the discovered package name where needed and `workspaceFolder` explicitly in multi-root workspaces. Cargo runs from that folder's manifest. Omit `targetTriple` to respect `.cargo/config.toml`, including the template's embedded default, linker flags, and environment settings. Obtain executable paths from tool results, never path guesses. Artifact IDs are session-local (last 64 builds); rebuild after reload, eviction, or changed executable contents.

## Procedure

1. **Frame the observation.** Ask what board and configuration are active, how to reproduce the issue, what should happen, and what actually happens. Identify the relevant source path and observable state/transition. If a physical target could be driven unsafely, stop and clarify its wiring/load first.
2. **Check project setup.** Call `bakerInspectProject`, inspect the relevant source and Cargo configuration, and call `bakerDebugStatus`. Select package/target/features and a launch mode. Do not start a duplicate session or replace an unrelated running session.
3. **Build narrowly.** For normal `launch`, let its `preLaunchTask` build unless a separate check is needed. For artifact modes, use `bakerBuildArtifact` (or embedded-build tests), require `success: true`, and retain the actual executable and artifact ID. Do not add a temporary task or change the source layout to build examples.
4. **Place purposeful breakpoints.** Read the discovered target source; an example can have a custom source path. List existing breakpoints first, add only useful executable lines with `bakerSetBreakpoint`, and confirm with `bakerListBreakpoints`. Track which ones this operation added and preserve pre-existing breakpoints. Addition/enabled state does not establish a hardware breakpoint hit.
5. **Confirm before hardware effects.** Explain the selected board, artifact, endpoints, flashing/reset behavior, and whether `preLaunchTask` runs. Wait for the tool confirmation before starting. Do not bypass approval with a terminal probe command.
6. **Observe without inventing evidence.** Read the launch result and call `bakerDebugStatus`. For pause state, variables, Debug Console, or RTT values, use a separately available observation tool or ask the user for the relevant UI output. If neither is available, mark those observations unverified. Never claim that `running: true` proves ADC readings or breakpoint arrival.
7. **Account for perturbation.** A breakpoint halts the processor and may change interrupt, DMA, Embassy scheduling, watchdog, and bus timing. If the bug disappears or changes, remove the breakpoint and investigate with RTT logs, GPIO timing markers, or an external logic analyzer/oscilloscope instead.
8. **Iterate and clean up.** Change one cause at a time, rebuild, and reproduce. For a bounded verification session, stop it with `bakerStopDebugging`, verify status, remove only newly added temporary breakpoints, and verify the list. Do not stop unrelated sessions or clear user breakpoints.

## Common failure categories

- **Build/link failure:** inspect the actual task output, target, features, linker script, and ELF path before attempting hardware operations.
- **`No client was connected`:** reconnect the VS Code client to the Dev Container. This is Agent tool dispatch failure, not evidence of DAP/USB failure; stop hardware actions rather than falling back to terminal probe-rs.
- **Attachment-only results:** verify extension 0.2.0 is installed in the container and reload. Do not infer results from unreadable attachments; rebuild artifacts after reload.
- **No host connection:** use Baker Link's connection check and verify the container-to-host endpoints/firewall. A TCP check does not prove USB probe readiness.
- **Probe or target not found:** verify host DAP service, USB/probe visibility on the host, chip selection (`RP2040`), SWD wiring, power, and reset state. Do not infer probe readiness from a successful network check.
- **ELF/profile mismatch:** in normal launch, align the configured task, `bakerLink.elf`, and artifact URI. In artifact modes, use the latest successful build's ID and let the extension substitute paths/URIs; do not manually guess them.
- **No breakpoint hit:** verify the source line is executable in the loaded ELF and that execution reaches it; the tool only sets a source breakpoint.

## Completion report

Report the selected launch configuration, build outcome, breakpoint locations, observed VS Code/RTT evidence, changes made, and whether the physical target was actually programmed and exercised. Separate observations from hypotheses and list any remaining uncertainty.
