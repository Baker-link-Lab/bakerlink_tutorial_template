# bakerlink_tutorial_template

<a href="https://www.buymeacoffee.com/Bakerlink.Lab" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>

Baker link.シリーズ用の組込みRustプロジェクトテンプレートです。

## Debug

1. Baker Link Envからプロジェクトを開きます。
2. VS Codeで **Reopen in Container** を実行します。
3. Baker Link EnvのDAP Serverを **Run** にします。
4. `F5`を押します。

pre-launch taskがコンテナ内でELFをビルドし、Baker Link Envへ転送してからデバッグを開始します。ホスト側パスの設定は不要です。コンテナは`vscode`非rootユーザーで動作します。
