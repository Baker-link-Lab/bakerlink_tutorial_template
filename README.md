# bakerlink_tutorial_template

<a href="https://www.buymeacoffee.com/Bakerlink.Lab" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>

Baker link.シリーズ用の組込みRustプロジェクトテンプレートです。

## Debug

1. Baker Link Envからプロジェクトを開きます。
2. VS Codeで **Reopen in Container** を実行します。
3. Baker Link EnvのDAP Serverを **Run** にします。
4. `F5`を押します。

pre-launch taskがコンテナ内でELFをビルドし、Baker Link Envへ転送してからデバッグを開始します。ホスト側パスの設定は不要です。

## Dev Container

[公式 Rust Dev Container イメージ](https://github.com/devcontainers/images/tree/main/src/rust)の
`mcr.microsoft.com/devcontainers/rust:1-bookworm` をベースに、Rust 1.94.1、
`thumbv6m-none-eabi`、`flip-link`、`cargo-binutils`、`elf2uf2-rs` を追加しています。

- Compose の `user: vscode` と Dev Container の `remoteUser: vscode` により、
  コンテナ・VS Code・ビルドタスクは通常 `vscode` ユーザーで動作します。
- Linux のバインドマウントの権限は `updateRemoteUserUID: true` により、
  コンテナ作成時にホストユーザーの UID/GID に合わせます。ワークスペースを
  root で `chown` したり、`chmod 777` にしたりする必要はありません。
- Cargo と Rustup は公式イメージ既定の `/usr/local/cargo` と
  `/usr/local/rustup` を使います。公式イメージの `rustlang` グループの権限を
  利用し、追加ツールも `vscode` としてインストールします。追加分にも同じ
  グループの書込み権限を適用し、UID/GID の変更後も利用できるようにしています。
- 公式イメージには開発用の `sudo` 権限が含まれます。通常のビルドに
  `sudo` は不要です。この構成は開発用であり、本番用イメージではありません。

Dockerfile やユーザー設定を変更した場合は **Dev Containers: Rebuild Container**
を実行してください。既に生成したプロジェクトにはテンプレートの変更が自動反映
されないため、該当設定を更新してから再ビルドしてください。
