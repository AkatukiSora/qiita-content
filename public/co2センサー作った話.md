---
title: co2センサー作った話
tags:
  - 'RaspberryPi'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# 前書き
皆さんは部屋でコーディングしているときや作業しているときに「気だるいな」や「なんか頭痛がする」と感じたことはありませんか？
そういうときはだいたい換気をすると解決するので二酸化炭素めぇ...と思っていたんです
最近RaspberryPi+CO2センサーで二酸化炭素濃度を可視化している記事[^1]を見つけました
真似するしかない、ということでセンサー買って作って視覚化までしてみました

:::note warn
この記事に書いてある内容で起こった出来事には責任を取れません
自己責任で行ってください
:::

[^1]:https://qiita.com/take314/items/a5ab8ef6e8773699cb25

# 使ったものもろもろ
### 使用機材

- RaspberryPi 3B+
- MH-Z19C[^2]
- Amazonで手に入れた野生のジャンパ線(メス-メス)

なぜこの時期に3B+かというとそこに使っていない3B+が埃被っていたからです
新しく買うならZero 2 Wとかを使うべきでしょう

それとMH-Z19Cは私が買ったときは2,500円程度だったんですが倍近く値上げしていました....
世知辛い

[^2]:https://akizukidenshi.com/catalog/g/g116142/

### 使用ソフト

- RaspberryPi OS Lite
- InfluxDB
- Grafana

ラズパイは基本SSHで操作するのでGUIは不要
Raspberry Pi Imagerで焼き込むときにWiFiの設定(有線の場合は不要)、ユーザーの設定とSSH公開鍵を焼き込んでおくと、最初からSSH接続できるので楽

InfluxDBはラズパイからネットワーク経由でアクセスできる場所に
GrafanaはInfluxDBにアクセスできる場所にデプロイしましょう
我が家では外出中でもアクセスできるようにKubernetesにデプロイしてCloudFlare Tunnel経由で公開しています


# InfluxDBとGrafanaをデプロイ

### InfluxDB

まずはInfluxDBをデプロイします
公式から出ている[helmチャート](https://github.com/influxdata/helm-charts)があるみたいなのでそれを使います
私はingressを使用するのでvalueFileをちょっとだけ書きます
また使用しているIngressClassとStorageClassはデフォルトに設定しているので割愛します

```yaml:value.yaml
ingress:
  enabled: true
  hostname: influxdb.example.com
```

デプロイできたことを確認したら公式ドキュメントに従って初期設定を行います
WebUIにアクセスできてバケットやapiトークンの作成ができればOKです

https://docs.influxdata.com/influxdb/v2/get-started/setup/

### Grafana

こちらも同じように公式ドキュメントにしたがってデプロイしていきます
必要に応じてリソース制限やStorageClassやサービスを変更します
私はドキュメントとは別にIngressを設定しました

https://grafana.com/docs/grafana/v11.4/setup-grafana/installation/kubernetes/

セットアップを済ませWebUIが使えたらOKです


# 組み立て

GNDと5VはGNDと5Vに接続するだけです
RXはGPIO14に、TXはGPIO15に接続します
ラズパイのGPIOヘッダーは順番にGPIO番号が振られているわけではないので`pinout`コマンドを使用して目的の番号を探してください
GNDと5Vの場所も`pinout`でわかります

# ラズパイの設定

ここに手こずりました
Bluetoothを無効化しないと使えないみたいでこれにたどり着くまでとても時間がかかりました

/boot/firmware/config.txtに以下の設定があることを確認します
なければ[all]のところに追記してください
```ini:/boot/firmware/config.txt
[all]
enable_uart=1
dtoverlay=pi3-disable-bt
```

この状態で再起動します
起動して`/dev/ttyAMA0`が存在していればOKです

# 簡単にコーディング

偉大な先人がデータを読み出すpythonのライブラリを作っていただいていたのでそれを使います
pythonはなんかvenvとか言うものを使わないと使えなくなたみたいなので先に設定しておいてください
このあとはvenv環境で動かしてる前提で書いていきます

```bash:bash
pythom -m pip install mh_z19
```

pythonコマンド経由でpipを呼び出すのは多分venvのpipを使うためだと思います
pipでグローバルにパッケージをインストールできないようになってるようなので

```bash:bash
sudo python -m mh_z19
```
を実行して`{co2: 450}`とかが表示されればokです
ちなみにシリアル通信を使用するのでsudoが必要です
私の環境だとvenvを有効化したうえで`venv/bin/python3 -m mh_z19`としないと呼び出せませんでした
なにか間違えたかも知れません

実際にデータをinfluxDBにpushするコードはこちらです
```python:python
import influxdb_client, time
from influxdb_client import InfluxDBClient, Point, WritePrecision
from influxdb_client.client.write_api import SYNCHRONOUS

import mh_z19

#influxdb client init
token = ""
org = "influxdata"
url = "https://influxdb.example.com"
bucket="ExampleBucket"

write_client = influxdb_client.InfluxDBClient(url=url, token=token, org=org)
write_api = write_client.write_api(write_options=SYNCHRONOUS)

ExpectExecCycle = 5 #s

def getCo2():
    co2DataValue = mh_z19.read()
    return co2DataValue

def getPushData():
    Data = getCo2()
    record = (
        Point(bucket)
        .tag("functionality", "Sensor")
        .field("co2_level", Data["co2"])
    )
    write_api.write(bucket=bucket, org=org, record=record)

while True:
    biginTime = time.time()

    getPushData()

    endTime = time.time()

    execTime = endTime - biginTime
    diffTime = ExpectExecCycle - execTime
    time.sleep(diffTime)
```

ほとんどinfluxDBのチュートリアルそのままのコードを使っていますがapiキーだけ目的のバケットへのwrite権限だけのものにしています
最小権限の原則ってやつです
環境に合わせてうまく調整してください

コード最後の方はCO2データの読み出しに結構時間がかかっていたので厳密に5秒感覚でデータを取るようにちょっとした細工しています

あとはsystemdデーモンにするなりして完成です

# Grafanaで可視化

まずinfluxDBのwebUIを開きます

Data Explorer > CO2のデータがあるバケット > タグ等
で目的のCO2のデータだけが見えるようにしてください
目的のデータだけが見えるようになったら`SCRIPT EDITER`を押してfluxクエリを表示します
それをすべてコピーしてGrafanaに移動します

先にGrafanaでinfluxDBの情報が見えるようにデータソースとして登録します
ここはおそらく案内があるのでその通りに作成してください
ここでもapiキーはすべてのバケットにread権限だけにしています
最小権限の原則ってやつです(二回目)

データソースとして登録できたらダッシュボードとパネルを作成します
パネルの設定画面でデータソースをinfluxDBにして、queryの欄に先ほどコピーしたものを貼り付けます
あとは細かい見え方や表示の仕方などを調整して完成です

# 後書き
特に理由はないですが自室のデータを公開しているのでどんなものか気になった人は見てみてください

https://grafana.akatuki-host.com/public-dashboards/c6e2a86e2a4648178232ca5aac18c4bf

部屋を閉じきって2~3時間もすれば平気で1000ppm[^3]は超えるので定期的な換気は重要と再認識しました
数分窓を開けておくだけできれいにCO2濃度が落ちていくのを見えるのは面白かったです

自分のいる環境がどんな状態か可視化されるのは新鮮でしたし、換気のタイミングもはっきりわかるのでおすすめです
個人的にはいい感じに可視化出来ていて、デスクトップのあいてる端の方においておくと幸せになれるので満足しています
それとドアを開ける、窓を開ける等の動作にかなり敏感に反応していて驚きました
今後もセンサー類は追加していこうと思っています


[^3]:CO2濃度は400ppm前後が外気、500~700ppmが正常に換気された室内、1000ppmを超える眠気などが起き始め換気推奨と言われている
