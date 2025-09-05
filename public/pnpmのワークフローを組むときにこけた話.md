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
本記事は

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

## actions/node-setupが失敗する

### 発生状況

その時yarnで動いていたワークフローをpnpmに変更しようとしていました。
その時はsetup-nodeの後ろにpnpm/action-setupを追加しました。
そのごGitHub Workflowを動作させたところsetup-nodeが以下のエラーで終了しました
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
[失敗したワークフロー](https://github.com/AkatukiSora/qiita-content/actions/runs/17459539391/job/49580773096)

### 主な原因

[結論](#結論)でお話しているようにcacheの指定をしているのにpnpmのセットアップよりも前でnode-setupをしていることが原因です。
理由はcacheを設定してsetup-nodeを実行する場合、途中でpnpmを実行してキャッシュのpathを取得しているためこれが原因だと思います。

以下でキャッシュパスの取得に使用するコマンドが定義されていて

https://github.com/actions/setup-node/blob/v5.0.0/src/cache-utils.ts#L34-L37

気合でコードをたどっていくと以下で実際に実行されています

https://github.com/actions/setup-node/blob/v5.0.0/src/cache-utils.ts#L69-L87


### 解決方法

cacheを指定する場合はactions/setup-nodeよりも先にパッケージマネージャーのセットアップを済ませておきましょう

## 参考リンク

https://github.com/actions/setup-node/tree/v5.0.0
