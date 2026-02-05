---
title: ubuntu22.04のネットワークを管理しているソフトを特定する方法
tags:
  - UbuntuServer
  - Ubuntu22.04
private: false
updated_at: '2025-01-06T16:20:37+09:00'
id: 98ec5c2f4bf877c55ecb
organization_url_name: null
slide: false
ignorePublish: false
---

ネットワーク接続を管理するとき、OSにもよりますが複数の方法がありますよね。
debian系のosならnetplan、systemd-networkdなど、他にも複数の管理ツールがあります。
それぞれ設定方法が異なり、設定ファイルのパスを間違えれば適応されません。
合っていたとしても違う管理ツールの設定であれば、もちろん適応されません。

まずはどのツールで管理されているのか特定したいです。
最近改めてその方法を知ったので書き残しておきます。

# 環境

- OS: ubuntu22.04
- kernel: 5.15.0-130-generic
- systemd: 249.11-0ubuntu3.12

k8sのコントロールプレーンとして稼働させているVM上のubuntuです。

# ネットワークを管理しているツールの特定

といってもそこまで仰々しいものではなく、言われてみればという内容なのですが。

```bash:bash
sudo systemctl is-active "<任意の管理ツール>"
```

これの出力結果がactiveまたはinactiveであることを確認するというだけのものです。
activeであると分かれば、あとは対応する設定方法を調べて適応するだけです。

