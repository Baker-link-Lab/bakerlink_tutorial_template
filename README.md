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
rules. On Linux, the Compose configuration supplies
`host.docker.internal:host-gateway`.
