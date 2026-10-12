# bakerlink_tutorial_template

<a href="https://www.buymeacoffee.com/Bakerlink.Lab" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>

Baker link.シリーズ用の組込みRustプロジェクトテンプレートです。

## Baker Link Env VS Code extension

This template automatically installs
[Baker Link Env](https://marketplace.visualstudio.com/items?itemName=baker-link-lab.baker-link-env)
from the VS Code Marketplace inside the Dev Container via
`.devcontainer/devcontainer.json`. No manual VSIX installation is required.
Wait for the extension installation to finish before debugging.

For an existing container, run **Dev Containers: Rebuild Container** after
updating the configuration, or install `baker-link-lab.baker-link-env` manually
from the Extensions view in the Dev Container window. Verify it is installed
in the container, not only on the host. Requires VS Code >= 1.116.0.

Start Baker Link Env on the host with **Run**, then press **F5**. The
`cargo: build` pre-launch task builds the ELF; the extension uploads it with
SHA-256 verification before connecting to the host DAP server.

- `.vscode/launch.json` uses `type: "baker-link-debug"`,
  `bakerLink.dapServer` and `bakerLink.uploadUrl` for host endpoints,
  `bakerLink.elf` for the container-side file, and
  `programBinary: "bakerlink://<project>/debug"` for the host artifact.
- Run **Baker Link: Check Host Connection** to check TCP reachability.
  Select the debug configuration if the project has multiple configurations.
  It does not check USB probe readiness.
- **Baker Link: Open Connection Settings** opens `.vscode/launch.json`.
- Requires extension version 0.1.1 or newer. `.bakerlink` is no longer needed;
  ELF upload is handled entirely by the extension, without `jq`, `curl`, or a
  project-local script. DAP and upload endpoints default to
  `host.docker.internal:50001` and `http://host.docker.internal:50002` when omitted.

For an existing project, copy non-default endpoints from
`.bakerlink/connection.json` into the `bakerLink` object in `.vscode/launch.json`,
switch `preLaunchTask` to a build-only task, remove shell-upload tasks, then
delete `.bakerlink`. The new extension does not read the old connection file.

For release/example builds, update both the build task and `bakerLink.elf`,
and use a matching artifact profile/identifier. The MVP uploads one artifact
per launch session. There is no host start/stop API or automatic host startup.
Both host services are unauthenticated; use only trusted networks and firewall
rules.

## Cross-platform host connection

The debug endpoints stay the same on Windows, macOS, and Debian:
`host.docker.internal:50001` and `http://host.docker.internal:50002`.
The base `.docker/compose.yaml` must not override this host name.
`.devcontainer/devcontainer.json` also loads `.docker/compose.host.yaml`
(JSON is valid YAML).

Opening through Baker Link Env configures this override for the active Docker
engine. On Rancher Desktop / Docker Desktop it leaves `extra_hosts` empty so
the runtime's native DNS reaches the Windows/macOS/Linux host. On local,
rootful Docker Engine on Debian/Linux it adds the `host-gateway` mapping.
Rootless, remote native Docker engines and Windows containers are not supported
by automatic setup.

If opening directly in VS Code without Baker Link Env, use the shipped empty
override for Desktop runtimes. For local, rootful Docker Engine on Linux, set:

```json
{
    "services": {
        "{{project-name}}": {
            "extra_hosts": {
                "host.docker.internal": "host-gateway"
            }
        }
    }
}
```

After changing hosts or Docker runtimes, reopen through Baker Link Env (or
update the override manually), then run **Dev Containers: Rebuild Container**.
An existing container retains its old hosts mapping until recreated.
Start both host services with **Run** in Baker Link Env, using Bind IP
`0.0.0.0`, then run **Baker Link: Check Host Connection** in the container.
Allow the host ports through the firewall only for trusted networks.

## Running embedded tests

The template uses
[embedded-test](https://github.com/probe-rs/embedded-test) with `probe-rs run`
as the Cargo target runner. Connect an RP2040 and run the tests from an
environment where `probe-rs` can access the debug probe:

```sh
cargo test --test example_test
```

The `.cargo/config.toml` runner and `build.rs` linker-script setup are already
configured. `defmt-test` is not needed; `defmt` and `defmt-rtt` remain available
for logging.
