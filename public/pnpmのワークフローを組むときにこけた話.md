---
title: pnpmのワークフローを組むときにこけた話
tags:
  - 'Node.js'
  - 'pnpm'
  - 'GitHubActions'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

本記事は、GitHub Actionsで`cache: pnpm`を指定したsetup-nodeを使う際にハマったので備忘録がてら残しておこうと思います。

## 環境

- Node.js: `v22`
- pnpm: `v10.13.1`（Corepack経由）
- CI: GitHub Actions（`ubuntu-latest`）

## 結論

> \# Used to specify a package manager for caching in the default directory. Supported values: npm, yarn, pnpm.
> \# Package manager should be pre-installed
> \# Default: ''
>
> [GitHub actions/setup-node README.md](https://github.com/actions/setup-node/blob/a0853c24544627f65ddf259abe73b1d18a591444/README.md) より引用

`Package manager should be pre-installed`と書かれています。
cacheパラメーターを指定する場合、事前にパッケージマネージャーがセットアップされている必要があります。

## actions/setup-nodeが失敗する

### 発生状況

その時yarnで動いていたワークフローをpnpmに変更しようとしていました。
その時はsetup-nodeの後ろにpnpm/action-setupを追加しました。
その後GitHub Actionsのワークフローを動作させたところ、setup-nodeが以下のエラーで終了しました。
```bash

Run actions/setup-node@v4
  with:
    node-version: 20.18.0
    cache: pnpm
    cache-dependency-path: pnpm-lock.yaml
    always-auth: false
    check-latest: false
    token: ***
Attempting to download 20.18.0...
Acquiring 20.18.0 - x64 from https://github.com/actions/node-versions/releases/download/20.18.0-11182621166/node-20.18.0-linux-x64.tar.gz
Extracting ...
/usr/bin/tar xz --strip 1 --warning=no-unknown-keyword --overwrite -C /home/runner/work/_temp/4e584c97-3c0c-4bc5-a92e-4c93e8b2411d -f /home/runner/work/_temp/0ae0a89d-9c47-46f0-a2da-ad5825c8aa60
Adding to the cache ...
Environment details
  node: v20.18.0
  npm: 10.8.2
  yarn: 1.22.22
Error: Unable to locate executable file: pnpm. Please verify either the file path exists or the file can be found within a directory specified by the PATH environment variable. Also check the file mode to verify the file is executable.

```

[変更の差分](https://github.com/AkatukiSora/qiita-content/commit/6b7d99fe303fcb5b0fcd25c049948d55fe284c2b#diff-32824c984905bb02bc7ffcef96a77addd1f1602cff71a11fbbfdd7f53ee026bb)

### 主な原因

[結論](#結論)で述べたとおり、`cache`を指定しているのにpnpmのセットアップよりも前で`setup-node`を実行していることが原因です。
`cache`を設定して`setup-node`を実行する場合、内部でpnpmを呼び出してキャッシュのパスを解決します。そのため、pnpmが未インストールだと失敗します。

- 参照: [キャッシュパスの解決ロジック](https://github.com/actions/setup-node/blob/v5.0.0/src/cache-utils.ts#L34-L37)
- 参照: [実行箇所](https://github.com/actions/setup-node/blob/v5.0.0/src/cache-utils.ts#L69-L87)


### 解決方法

`cache`を指定する場合は、`actions/setup-node`よりも先にパッケージマネージャー（pnpm）のセットアップを済ませておきましょう。

正しい順序（例）:

```yaml
- name: Set up pnpm
  uses: pnpm/action-setup@v4
  with:
    run_install: false

- name: Set up Node.js
  uses: actions/setup-node@v4
  with:
    node-version: 22
    cache: pnpm
    cache-dependency-path: pnpm-lock.yaml
```

誤った順序（落とし穴の例）:

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 22
    cache: pnpm
# ↑ この時点で setup-node が pnpm を探しに行き失敗する
- uses: pnpm/action-setup@v4
```

## 参考リンク

- actions/setup-node: https://github.com/actions/setup-node/tree/v5.0.0
- pnpm/action-setup: https://github.com/pnpm/action-setup
