# 第5回 / Raspberry Piの操作

{% hint style="warning" %}
この回は鯖室でRaspberry Piを使って行うため、事前の予約が必要です。詳しくは#22-infra-lectureの指示に従ってください。
{% endhint %}

### この章のゴール

次の事項について確認する。

* Raspberry Piの起動
* nfcpy

また、Raspberry Piで学生証から学籍番号を読み取り、PART 1のAPIサーバーと連携させる。

### Raspberry Piとは

　小型のコンピューターです。もともと教育用に開発されましたが、現在は個人開発や製品開発など幅広い分野で使われています。

　GUIとCUIの両方で操作できますが、今回は時間の関係からCUIで操作することにします。

### 準備

以下の装置を準備してください。

* Raspberry Pi Model B+ v1.2（以下、Raspberry Piと呼称）
* Sony PaSoRi RC-S380（以下、ICカードリーダーと呼称）
* LANケーブル
* 電源ケーブル

　自分のPCにVisual Studio Codeをインストールし、さらに拡張機能「Remote Development」をインストールしてください。

### ログイン

　自分のPCでVisual Studio Codeを開きます。左下の緑ボタンから、「Remote-SSH: Connect to Host」を選択します。

　接続先を聞かれたら`raspberry@<IP>`を入力します。そして「フォルダーを開く」から、`/home/raspberry`を開きます。

### ICカードリーダーを使う

　左の「ファイルエクスプローラー」から`card_reader.py`というファイルを作り、次のプログラムを入力する。\`#\`から始まる行は入力しなくてよいです。

```python
import nfc
import requests

# 諸々の設定情報
API_URL = "<自分のAPIサーバーのURL>?id="
PROXY = "http://172.20.20.104:8080"
PROXIES = {"http": PROXY,"https": PROXY}
SYSTEM_CODE=0x809e
SERVICE_CODE = 0x100b

# ICカードを読み取ったときの処理
def connected(tag):
  global prev, p
  # ICカード自体についての情報を受け取る
  idm, pmm = tag.polling(system_code=SYSTEM_CODE)
  tag.idm, tag.pmm, tag.sys = idm, pmm, SYSTEM_CODE

  if isinstance(tag, nfc.tag.tt3.Type3Tag):
      # ICカード内に保存されている情報を受け取る
      sc = nfc.tag.tt3.ServiceCode(SERVICE_CODE >> 6, SERVICE_CODE & 0x3f)
      bc = nfc.tag.tt3.BlockCode(0, service=0)
      data = tag.read_without_encryption([sc], [bc])
      # 受け取った情報から学籍番号を抽出する
      sid = data[2:9]
      
      # 学籍番号をコンソールに表示する
      print(sid)
      # APIサーバーへ学籍番号を投げる
      requests.post(API_URL + str(sid), proxies=PROXIES, verify = False)
  else:
    print("tag isn't Type3Tag")

# USBで接続したICカードリーダーを初期化する
clf = nfc.ContactlessFrontend('usb')
clf.connect(rdwr={'on-connect': connected})
```

　コンソールから次のコマンドを実行する。

```bash
$ python3 card_reader.py
```

　ICカードリーダーに学生証を近づけると、コンソールに学籍番号が表示され、同時にSlackへメッセージが送信されることを確認する。
