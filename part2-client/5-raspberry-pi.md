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

　Raspberry Piは小型のコンピューターです。もともと教育用に開発されましたが、現在は個人開発や製品開発など幅広い分野で使われています。

　GUIとCUIの両方で操作できますが、今回は時間の関係からCUIで操作することにします。

### 準備

　以下の装置を準備してください。

* Raspberry Pi Model B+ v1.2（以下、Raspberry Piと呼称）
* Sony PaSoRi RC-S380（以下、ICカードリーダーと呼称）
* LANケーブル
* 電源ケーブル

　自分のPCをSYSLANに接続してください。

### ログイン

1. Windowsターミナルを開きます
2. `ssh pi@192.168.111.134`を実行します
3. パスワードを入力します。
4. `pi@raspberrypi:~ $`と表示されたらログイン成功です

{% hint style="info" %}
手順2についての説明：

`sshはリモートのコンピューターに接続して操作するためのコマンドです。`

`192.168.111.134はSYSLANにおけるRaspberry piのIPアドレス、piはRaspberry piでのユーザー名を意味しています。`
{% endhint %}

### Linuxの操作の復習

* ディレクトリ（＝フォルダ）への移動: `cd <移動先のディレクトリ名>`
* ディレクトリの作成: `mkdir <ディレクトリ名>`
* 現在のディレクトリにあるファイルの確認: `ls`
* ファイルの編集: `vi <編集したいファイル名>` or `nano <編集したいファイル名>`

#### ファイルの編集について（vi）

　ようこそ。viはすばらしいエディターです。

* `i`を押すと編集モードになり、文字が打てます
* `ESC`を押したのち`:wq`を入力するとファイルを保存して閉じます。

#### ファイルの編集について（nano）

* メモ帳と同じように文字が打てます
* `CTRL+X`->`Y`->`ENTER`でファイルを保存して閉じます。

### ICカードリーダーを使う

　まずディレクトリ`infra-lecture`へ移動してください。そして自分のSlack IDと同じ名前のディレクトリを作成してください。次に、`card_reader.py`という名前のファイルを編集し、次のプログラムを入力してください。`#`から始まる行は入力しなくてよいです。

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
      sid = int(data[2:9])
      
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

　終わったら、ターミナルから次のコマンドを実行してください。

```bash
$ python3 card_reader.py
```

　ICカードリーダーに学生証を近づけると、ターミナルに学籍番号が表示され、同時にSlackへメッセージが送信されることを確認してください。

{% hint style="warning" %}
実行時にno such deviceと表示されたら：

他の人のプログラムがICカードリーダーを使用している可能性が高いです。ICカードリーダーは同時に1つのプログラムからしか使用できません。時間をおいて再度試してみてください。
{% endhint %}

### 時間が余ったら

* [https://qiita.com/pf\_packet/items/9a50d9f3b1f478930b02](https://qiita.com/pf\_packet/items/9a50d9f3b1f478930b02) を読んでNFCのしくみを理解する
  * `SYSTEM_CODE`、`SERVICE_CODE`に関連
* GPIOを使ってLEDを光らせる
