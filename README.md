# bakerlink_tutorial_template

<a href="https://www.buymeacoffee.com/Bakerlink.Lab" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>

Baker link.シリーズ用の組込みRustプロジェクトテンプレートです。

## Baker Link Env VS Code extension

This template uses the development version of
[Baker Link Env](https://github.com/Baker-link-Lab/baker-link-env-vscode).
The extension is not yet published to the Marketplace. Build its VSIX with
`npm ci && npm run package` in that repository, reopen this project in its
Dev Container, and use **Extensions: Install from VSIX...** to install the VSIX
in the container (not just on the host).

Start Baker Link Env on the host with **Run**, then press **F5**. The
`cargo: build` pre-launch task builds the ELF; the extension uploads it with
SHA-256 verification before connecting to the host DAP server.

- `.bakerlink/connection.json` is the source of truth for DAP and upload endpoints.
- `.vscode/launch.json` uses `type: "baker-link-debug"`,
  `bakerLink.elf` for the container-side file, and
  `programBinary: "bakerlink://<project>/debug"` for the host artifact.
- Run **Baker Link: Check Host Connection** to check TCP reachability.
  It does not check USB probe readiness.
- The shell upload tasks remain available for manual use or the official
  probe-rs extension. To use that extension instead, change `type` to
  `"probe-rs-debug"`, remove `bakerLink`, set `server` to the DAP endpoint,
  keep `remoteServerMode: false`, and use
  `"preLaunchTask": "baker-link: prepare debug"`.

For release/example builds, update both the build task and `bakerLink.elf`,
and use a matching artifact profile/identifier. The MVP uploads one artifact
per launch session. There is no host start/stop API or automatic host startup.
Both host services are unauthenticated; use only trusted networks and firewall
rules. On Linux, the Compose configuration supplies
`host.docker.internal:host-gateway`.
