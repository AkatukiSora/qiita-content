---
title: ArcブラウザでTLS証明書の詳細を確認しようとしたけど、うまくいかなかった話
tags:
  - ARC
  - certificate
  - TLS証明書
private: false
updated_at: '2024-12-15T16:56:27+09:00'
id: f3915b69ee51c7a8e36a
organization_url_name: null
slide: false
ignorePublish: false
---
最近Arcブラウザっていう比較的新しいブラウザを教えてもいました
しかしTLS証明書の詳細の開き方がわからず、海外記事を気合で読み漁り、模索しているところです
All Englishなドキュメントはできるだけ読みたくないので、未来の自分のためにここにまとめようと思います

# 公式ヘルプをチェック

まずはArcのヘルプセンターから情報を探してみます
開くと大きく

- Getting Started
- Features
- Troubleshooting

とありました

が検索フォームがあったのでこの3つは無視して検索します
"cert"とかで検索すれば出てくるだろうと踏んでいたのですが、一致するものはありませんでした
"TLS"、"SSL"等関連するワードでもめぼしいものは無く
細かく見ても、近しいものはありませんでした

# 仕方ないので英語で検索
海外発の情報を漁っていくとReddit[^1]とYoutube[^2]から有力情報を得ました
[^1]: https://www.reddit.com/r/ArcBrowser/comments/16rul9p/website_httpsssl_status_missing_in_arc/?rdt=47169
[^2]: https://www.youtube.com/watch?v=UL554VF_3c4

どうやらURLが表示されている部分の右側すぐのところにある、設定アイコンのようなものに隠れているようです
押すと左下の方に見慣れた鍵マークと"secure"という文字が
更にここを押すと証明書の詳細が見れるようです

# 解決？

redditにもあったように設定アイコンみたいなものを押すと、左下に見慣れた鍵マークと"secure"という文字がありました
なんですがなぜか押しても反応せず......

他に情報を探ってみるもそれらしいものを見つけられませんでした
解決策が見つからなさそうな雰囲気がするので、一旦公式に問い合せて結果を待つことにします

続きは問い合わせの結果がまとまり次第、更新するはずです
