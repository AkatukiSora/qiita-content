---
title: Google Cloud のプロジェクトが消せなかった話
tags:
  - GoogleCloud
  - Cloud
  - Google
private: false
updated_at:
id:
organization_url_name: null
slide: false
ignorePublish: false
---

結構しっかり目に運用していた Google Cloud のプロジェクトが不要になったので、課金を停止してプロジェクトを削除（シャットダウン）[^1]しようとしたところ、なぜかエラーになり困った話です。

[^1]: Google Cloud のプロジェクトを削除（シャットダウン）すること。通常、削除後もしばらくは復元可能な猶予があります。

## 結論

App Hub の「アプリケーション管理」が関係しており、`gcloud` 経由でプロジェクトを境界（boundary）から切り離す、という操作をしたら削除できるようになりました。  
おそらく App Hub が比較的新しいサービスであるため、ドキュメントと実際の挙動にズレがある、と推測しています。

```bash
gcloud alpha apphub boundary update --project=PROJECT_ID --location=global --crm-node=""
```

※ `alpha` コマンドのため、将来的に挙動やオプションが変わる可能性もあります。

## 何が起きたか

- プロジェクトを消そうとしたら、Google Cloud Console（コンソール）上では「不明なエラー」扱いで削除できない
- 開発者ツールで APIレスポンスを見ると、次のような文言が出ていた

`Deletion is not allowed because this project is the management project for the folder it resides in.`

意味としては「このプロジェクトは（所属する）フォルダの管理プロジェクトになっているので削除できない」です。  
ただ、フォルダの管理プロジェクトなんて設定した覚えがなく、何を指しているのか分かりませんでした。

## まず公式のトラブルシュートに従う

[プロジェクト削除のトラブルシュート](https://docs.cloud.google.com/resource-manager/docs/troubleshooting-project-deletion?hl=ja)が公式にあったので、まずは素直にチェックしました。

ここで「削除をブロックする可能性のあるリソース[^2]」として挙がっていたものを確認しましたがいずれも使用しておらず、実際プロジェクト上に存在しませんでした。

[^2]: Liens / Cloud Endpoints / 共有 VPC など

## 原因っぽいもの：App Hub

いろいろ検索しているうちに、Cloud Monitoring のアラートを作成したときにおすすめされて触った App Hub が関係していそうだと分かりました。

ただし困ったのが App Hub に属するサービス、ワークロード、アプリケーションを消しても状況が変わらず、プロジェクト削除がブロックされたままだったことです。

## 解決：App Hub のアプリケーション管理を無効化する

App Hub のドキュメントを追っていくと、単一プロジェクト構成でのセットアップ/解除に関するページがあり、そこに[アプリケーション管理を無効化する](https://docs.cloud.google.com/app-hub/docs/set-up-app-hub-single-project?hl=ja#disable)手順が載っていました。

どうやらコンソール上からは無効化できないようで、それらしいことを示す項目もなさそうです。
`gcloud` コマンドを使って無効化する必要があるようでした。

### 実行したコマンド

ドキュメント上では、次のようなコマンドの記載がありました。

ところが手元の環境では `--clear-crm-node` が「存在しないオプション」として弾かれました。

```bash
gcloud alpha apphub boundary update --project=PROJECT_ID --location=global --clear-crm-node
```

### つまずき：`--clear-crm-node` が無い

`gcloud alpha apphub boundary update --help` を見ると `--crm-node` は存在したため、試しに空文字を指定して実行してみました。  
結果としてこれが通り、App Hub のアプリケーション管理が無効化された状態になりました。

```bash
gcloud alpha apphub boundary update --project=PROJECT_ID --location=global --crm-node=""
```

### アプリケーション管理が無効化されたか確認

先述のコマンドが成功し、正常に App Hub から `crmNode`（`--crm-node` で指定する値）がクリアされたかどうかは、次のコマンドで確認できます。
出力に `crmNode` が含まれていなければ、無効化が成功しています。

```bash
gcloud alpha apphub boundary describe --project=PROJECT_ID --location=global
```

### crm-nodeって何ぞや

あまり調べても情報が出てこなかったのですが、少なくとも `--crm-node` にはプロジェクト、フォルダのいずれかのリソース名を指定するとドキュメントに記載があります。
なのでおそらく「どの範囲で App Hub のアプリケーション管理を有効にするか」を指定するオプションなのだと推測しています。

ここに関しては調べ方が悪いのか、ほとんど情報が出てこなかったため、正確なところは不明です。
わかる方いたら教えていただけると嬉しいです。

## その後

App Hub のアプリケーション管理が無効化されたのを確認したうえで、改めてプロジェクト削除すると、今度は問題なく削除できました。

<!-- 元メモ（必要なら後で削除）
- しっかり目に運用していたプロジェクトを消そうとしたら不明なエラーと報告され、消せなかった
- APIのレスポンスに`Deletion is not allowed because this project is the management project for the folder it resides in.`がいた
- 公式トラブルシュートに従う（Liens / Cloud Endpoints / 共有VPCなどは該当なし）
- App Hub が関係していそう。アプリ/ワークロード等を消してもダメ
- App Hub のドキュメントに「アプリケーション管理を無効化」があり、`gcloud` で無効化
- `--clear-crm-node`が無く、代わりに`--crm-node=""`で通った
- その後プロジェクト削除できた
-->
