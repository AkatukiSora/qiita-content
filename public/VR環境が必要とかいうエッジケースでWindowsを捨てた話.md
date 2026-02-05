---
title: VR環境が必要とかいうエッジケースでWindowsを捨てた話
tags:
  - Linux
  - steam
  - VR
private: false
updated_at: '2026-02-05T11:28:41+09:00'
id: 22e33bd3d7664a81df4e
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

この記事では、VR環境を必要とする、いわゆる「エッジケース」なユーザーがWindowsからLinuxへと移行した体験について備忘録的に書いていきます。

## なぜWindowsを捨てたのか

長年Windowsをメイン環境として使ってきましたが、以下の理由から移行を決意しました。

- **Windowsがあまり好きではなかった** - 個人的な好みの問題ですが、Windowsのシステム設計や挙動に違和感を感じていました。
- **WSL2での開発からの脱却** - WSL2で開発しており、Linuxネイティブ環境で開発したいという思いがありました。
- **ProtonやWineの技術進歩** - Linux環境でWindowsゲームを動かすProtonやWineなどの技術が大幅に充実しており、十分実用に耐えると判断しました。
- **LinuxでのVR環境のサポート** - VR環境もLinuxで動作するようになってきたことが大きな後押しとなりました。

## メイン機に必要な条件

移行先にほしい条件を考えていたら下記のようにまとまりました。

- **Steamのゲーム（VRChat）が動作する**
- **VR環境が動作する**
- **開発環境が整えられる**
- **普段使いに耐える程度の安定性**

これらすべてを満たせる環境であることが、移行の絶対条件でした。

## 移行先の選定

### 候補

以下のディストリビューションを候補として検討しました。

- Ubuntu
- Fedora
- Alma Linux<!-- textlint-disable -->
- Pop!_OS<!-- textlint-enable -->
- Arch Linux
- Bazzite

### 最終的な選択 Fedora KDE Desktop

最終的に**Fedora KDE Desktop**を選択しました。
理由は以下の通りです。

- **RHEL系のLinuxを使いたかった** - 仕事でも扱う可能性があり、学習効果が高いと判断。
- **友人の実績** - 友人がFedoraでVR環境を構築していたため、情報が得やすい。
- **KDEへの移行** - 最近GNOMEからKDEに乗り換えていたため、KDE環境を使いたかった。

## 前準備

### バックアップ作業

万が一に備えて移行前にバックアップを作成しました。

#### 1. データの退避

移行後も使用するデータについては、事前にクラウドストレージやGitHubに保存していたため、特別な対応は不要でした。ただし、以下の点は確認しておきましょう(n敗)。

- 開発プロジェクトがすべてGitHubにpush済みであること
- 重要なドキュメントがクラウドストレージに保存されていること
- クラウドストレージの同期が完了していること

#### 2. Windowsのフルディスクバックアップ

##### 不要なファイルの削除

バックアップサイズを縮小するため、以下の作業を実施しました。

- 一時ファイルやキャッシュの削除（ディスククリーンアップを使用）
- 使用していないアプリケーションのアンインストール
- ダウンロードフォルダやゴミ箱の整理
- 古い仮想マシンイメージや大容量ファイルの削除

##### Clonezillaによるバックアップ

今回バックアップツールとして、オープンソースの**Clonezilla**を使用しました。

手順は以下の通りです。

1. Clonezillaの起動用USBメモリを作成
2. BitLockerを使用している場合は解除
3. Clonezillaから起動し、ディスク全体をイメージとして外付けHDDに保存
4. バックアップの検証を実施(特に指定しなければ自動)

ついでにBitLockerを解除する理由を書いておきます。

Clonezillaは内部的にpartcloneなどのツールを使用しており、データが存在するブロックのみをバックアップすることで効率化を図っています。  
しかしBitLockerは暗号化によってこれを妨げてしまいます。  
結果、空き領域も含めたすべてのブロックをバックアップする必要があり、バックアップサイズの大幅増加など、非効率なバックアップになってしまいます。  
このため、事前にBitLockerを解除しておくことをおすすめします。

#### 3. インストールイメージの作成

Fedora KDE Desktopの最新版ISOをダウンロードし、お好みの方法でUSBなどに焼き込みます。

だいたいRufusやEtcherなどのツールが使いやすいでしょう。
私はEtcherを使用しました。

## インストール

インストール自体は標準的な手順に従って実施しました。特別な設定はせず、btrfsのsubvolを使用したデフォルト構成で構築しています。

（詳細は割愛します）

## 環境構築

### 開発環境のセットアップ

従来の開発環境で使用していたツールを順次導入しました。

- VSCode
- Docker
- mise
- 1Password
- その他開発ツール

### GPUドライバの導入

---

NVIDIAのGPUドライバは、RPM FusionとAKMODを使用して導入しました。

参考：[RPM Fusion - NVIDIA Howto](https://rpmfusion.org/Howto/NVIDIA#Current_GeForce.2FQuadro.2FTesla)

必要に応じてセキュアブートの設定も実施：
[RPM Fusion - Secure Boot](https://rpmfusion.org/Howto/Secure%20Boot?highlight=%28%5CbCategoryHowto%5Cb%29)

### Steamの導入

---

SteamをインストールしてVRChatをプレイできる環境を整えました。

### Proton-GE-rtspの導入

VRChatで動画を視聴したり、RTSPなどのストリーミング配信を正しく動作させるために、**Proton-GE-rtsp**を導入しました。

#### Proton-GE-rtspとは

Proton-GE-rtspは、GloriousEggrollが開発するProton-GEのフォーク版で、RTSPストリーミングプロトコルのサポートが追加されています。VRChat内で動画プレイヤーやRTSP配信を視聴する際に必要となります。

#### 導入手順

[Proton-GE-rtsp GitHub Releases](https://github.com/SpookySkeletons/proton-ge-rtsp/releases)から最新版をダウンロードします。
Steamが認識できるディレクトリに展開します。

```bash
mkdir -p ~/.steam/root/compatibilitytools.d/

tar -xzf proton-ge-rtsp-*.tar.gz -C ~/.steam/root/compatibilitytools.d/
```

Steamを完全に終了してから再起動します。  

その後Steamライブラリから以下の手順でVRChatにProton-GE-rtspを適用します。

- VRChatを右クリック → プロパティ
- 互換性タブを開く
- 「特定の Steam Play 互換性ツールの使用を強制する」にチェック
- ドロップダウンから「proton-ge-rtsp」を選択

これでVRChatを起動すると、Proton-GE-rtsp経由で実行され、動画プレイヤーやRTSP配信が正常に動作するようになります。

## Flatpak周りでのトラブル

### 問題発生

Flatpak版のSteamでVRChatを起動したところ、動作が異常に重たくなりました。

`nvtop`で確認すると、GPUがほとんど使われていないことが判明しました。

### 原因と解決

Flatpakのサンドボックスにより、VRChatがGPUに適切にアクセスできていなかったことでした。

今回はネイティブ版のSteamを導入することで問題を解消しました。

## VR環境の構築

### WiVRnの選択

今回はSteamVRを使わず、**WiVRn**を選択しました。

### WiVRnの導入

1. **Flatpakで導入** - WiVRnはFlatpak版で問題なく動作。
2. **VRChatの起動オプション設定** - WiVRnのガイドに従って設定。
3. **動作確認** - WiVRnを起動し、HMD（Quest 3）から接続してVRChatを起動。問題なく動作した。

### SteamVRオーバーレイの代替としてWayVRを導入

SteamVRのオーバーレイなどの資産が活かせないのは残念でしたが、代替として**WayVR**（旧WlxOverlay-s）を導入しました。

### 謎のトラブル（未解決）

WayVRの使用中に以下のような謎の問題に遷遇しています。

- WayVR起動中にQuest 3をパススルーモードにしたり、一時的にWiVRnの接続を終了すると、WayVRが「正常終了」するが、WiVRn側へアタッチされたままになる。
- すでにアタッチされていることになっているため、WayVRが再度起動できない。
- 時間を置くと解決することもあり、非常に謎。
- WiVRn、monado（OpenXRランタイム）、WayVRすべてのログで異常は見られず、LOSS_PENDINGへ遷移し、正常終了後もアタッチされたままになる模様。

この問題が解決できたら、また別の記事で詳細を共有します。

## まとめ

今回の移行で得られたものは以下の通りです。

- **Linuxネイティブ環境での開発** - WSL2を経由せず、直接Linux上で開発できるようになりました。
- **Windowsに依存しないVR環境** - Linuxでも快適にVRChatをプレイできることが確認できました。
- **ProtonやWineの技術進歩を実感** - ゲーム環境の充実度が想像以上でした。

## あとがき

VR環境が必要という、いわゆるエッジケースなユーザーでもLinuxへの移行が可能であることを実証できました。

Linuxネイティブで開発できるようになったことで、GUIとCLI間の連携（`xdg-open`など）が非常にやりやすくなりました。

今後も定期的にシステムの盆栽（調整・最適化）を楽しみつつ、何か新しい発見があればまた記事にします。

<!--
メモ

- まえがき
  - なぜWindowsを捨てたのか
    - Windowsがあんまり好きではなかった
    - WSL2で開発をしていたので、Linuxネイティブで開発をしたい
    - Linux周りでwindowsのゲームを動かすProtonやWineなどの技術が充実してきた
    - VR環境もLinuxで動くようになってきた
- 前提
  - メイン機に必要とする条件
    - Steamのゲーム(VRChat)が動く
    - VR環境が動く
    - 開発環境が整う
    - 普段使いに耐える程度の安定性
- 移行先選定
  - 候補 Ubuntu, Fedora, Alma, Pop!_OS, Arch, Bazzite
  - 今回はFedora KDE Desktopを選択
  - 理由
    - RHLE系のLinuxを使いたかった
    - 友人がFedoraでVR環境を構築していたので情報が得やすそう
    - 最近GNOMEからKDEに乗り換えていたのでKDEを使いたかった
- 前準備
  - バックアップを縮小するために不要なファイルを全削除
  - Windowsのフルディスクバックアップ
  - 今回はclonezillaを使用(BitLockerの解除をお忘れなく)
  - 移行後も使いたいデータはクラウドかGitHubにあるので割愛
  - インストールイメージの作成
- インストール(割愛)
  - 割愛
  - 今回はbtrfs subvolで特別なことはせずに構築
- 環境構築
  - 開発環境で使っていたものを導入
    - VSCode
    - Docker
    - Mise
    - 1password
    - その他開発ツール
  - rpmfusion + akmodでGPUドライバ導入(https://rpmfusion.org/Howto/NVIDIA#Current_GeForce.2FQuadro.2FTesla)
    - 必要に応じてセキュアブートを設定(https://rpmfusion.org/Howto/Secure%20Boot?highlight=%28%5CbCategoryHowto%5Cb%29)
  - Steamの導入
  - VRChatの導入
    - VRChatで動画を見たり、RTSPなどの配信を動作させるためにProton-GE-rtspを導入
    - Proton-GE-rtspの導入方法
- Flatpak周りでトラブル
  - Flatpak版SteamでVRChatの動作が異常に重たかった
  - nvtopで確認するとGPUをほとんど使っていなかった
  - 原因はFlatpakのサンドボックスによってVRChatがGPUにアクセスできていなかった模様
  - ネイティブ版Steamを導入して解決
- VR環境構築
  - 今回はSteamVRを使わない方法、WiVRnを選択
  - WiVRnの導入方法
    - Flatpakで導入
    - こちらはFlatpak版で問題ないよう
    - WiVRnのガイドに従ってVRChatの起動オプションを設定
  - 動作確認
    - WiVRnを起動し、HMDから接続してVRChatを起動
    - 問題なく動作
  - SteamVRのオーバーレイなどの資産が活かせず悲しい
    - 代替としてWayVR(旧WlxOverlay-s)
    - 謎のトラブル
      - WayVR起動中にQuest3をパススルーモードにしたり、一時的にWiVRnの接続を終了するとWayVRが"正常終了"し、WiVRn側にアタッチされたまま終了する
      - すでにアタッチされていることになっているせいでWayVRが再度立ち上がらあい
      - 時間を置くと解決することもあるので非常に謎
      - WiVRn, monado(OpenXRランタイム), WayVRすべてのログに異常は見られず、LOSS_PENDINGへ遷移し、正常終了後にアタッチされたままになる模様
      - 解決できたらまた記事を描く
- まとめ
  - 今回の移行で得られたもの
    - Linuxネイティブ環境での開発
    - Windowsに依存しないVR環境
    - ProtonやWineの技術進歩によるゲーム環境の充実
- あとがき
  - VR環境が必要とか言うエッジケースでもLinuxへ移行できた
  - Linuxネイティブで開発ができるようになったのでGUIとCLI間の連携がやりやすくなった(xdg-openとか)
  - 今後も定期的に盆栽をしつつ、何かあればまた記事にしたい

-->
