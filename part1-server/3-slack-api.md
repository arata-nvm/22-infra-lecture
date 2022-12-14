# 第3回 / Slack botの作成

{% hint style="warning" %}
🚧 工事中。
{% endhint %}

### この章のゴール

次の事項について理解する。

* net/http
* Slack API

また、Replitのシェルで以下のコマンドを実行すると"Hello, hoge!"のメッセージがSlackに送信されるAPIを実装する。

```bash
$ curl -X POST https://FoolhardyGiganticProcedure.aratasub.repl.co?id=hoge
```

### Slack APIを使う準備

　Slackにメッセージを送信するためにIncoming WebhookというSlackの機能を使用します。これは、HTTPリクエストをSlackから指定されたURLへ送信すると、指定したチャンネルへ指定したメッセージを送信してくれる機能です。

　これから使用するのでSlackで#22-infra-sandboxチャンネルに参加しておいてください。

　まずはSlackからWebhook URLを取得しましょう。Webブラウザで [https://sysken.slack.com/apps/A0F7XDUAZ--incoming-webhook-](https://sysken.slack.com/apps/A0F7XDUAZ--incoming-webhook-) を開いてください。Slackへのログインを求められた場合はSYSKENのSlackのアカウントでログインしてください。すると次のような画面が開くので、緑色の「Slackに追加」ボタンを押してください（図1）。

<figure><img src="../.gitbook/assets/Screenshot 2022-12-14 at 17.21.22.png" alt=""><figcaption><p>図1: Incoming Webhookの画面(1/3)</p></figcaption></figure>

　次の画面に進んだら、チャンネルを選択する画面が開きます。リストから#22-infra-sandboxを選択して、緑色の「Incoming Webhookインテグレーションの追加」ボタンを押してください（図2）。

<figure><img src="../.gitbook/assets/Screenshot 2022-12-14 at 17.21.33.png" alt=""><figcaption><p>図2: Incoming Webhookの画面(2/3)</p></figcaption></figure>

　Incoming Webhookの設定画面に移ります。画面を下にスクロールして、図の項目を探してください（図3）。見つけたら、次のことをしてください。

* 投稿するチャンネルが#22-infra-sandboxになっていることを確認してください
* 名前を「\[Slack ID]-bot」にしてください
  * 例: Slack IDがArataの場合、「Arata-bot」にする
* （任意）説明ラベル、アイコンを好きなものにしてください

　最後に、緑色の「設定を保存する」ボタンを押してください。後でWebhook URLをコピペするので、このページは開いたままにしておいてください。

<figure><img src="../.gitbook/assets/Screenshot 2022-12-14 at 17.21.58.png" alt=""><figcaption><p>図3: Incoming Webhookの画面(3/3)</p></figcaption></figure>

### cURLでリクエストを送る

　さて、もしWebブラウザでWebhook URLを開いた好奇心旺盛な人がいればお気づきかもしれませんが、Incoming WebhookへのリクエストはWebブラウザでは送信することができません（正確には頑張れば送ることはできる）。これは、通常のリクエストと以下の点が異なるためです。

* HTTPリクエストメソッドがGETではなくPOSTである
* HTTPリクエストボディが必要である

　そこで、リクエストを送信するためにcURLというツールを利用します。これは、HTTPを始めとしたプロトコルを使ってデータの送受信ができるツールです。Windowsにも標準でインストールされていますが、OSによって細かい動作が異なるため、ここではLinux上で動くcURLに統一することにします。

　第2回で使用したReplitのプロジェクトを開くと、右側にシェル（"Console"パネルの横にある"Shell"パネルのこと）というパネルが表示されるはずです。実はReplitでは1つのプロジェクトごとに1つのLinuxが起動しており、ここでLinuxのコマンドを実行できます。Linuxについては第4回で触れるので、詳しい説明は省略します。

　ひとまず説明は置いておき、以下のコマンドをシェルに入力して実行してみてください。

{% hint style="warning" %}
注:

* 先頭の$は入力しなくてよいです。これはLinuxのコマンドであることを示す記号です。
* `<自分のWebhook URL>` をIncoming Webhookの設定画面のWebhook URLに変更してください。
* コマンドを入力したら、エンターキーを押して実行してください。
{% endhint %}

```bash
$ curl -X POST -d '{"text": "Hello, world!"}' <自分のWebhook URL>
```

　するとSlackの#22-infra-sandboxチャンネルに「Hello, world!」というメッセージが投稿されているはずです。

　このコマンドの詳細は次のとおりです。

| コマンド                             | 意味                                                       |
| -------------------------------- | -------------------------------------------------------- |
| `curl`                           | cURLコマンドを使うことを示す。                                        |
| `-X POST`                        | HTTPリクエストメソッドにPOSTを指定する。                                 |
| `-d '{"text": "Hello, world!"}'` | HTTPリクエストボディに`'payload={"text": "Hello, world!"}'`を指定する。 |
| `<自分のWebhook URL>`               | HTTPリクエストを送信するURLを指定する。                                  |

　ちなみに、いままでWebブラウザで行っていた単純なGETリクエストは以下のコマンドで同じことができます。

```
$ curl <URL>
```

#### 演習1

　第2回で構築したAPIに、cURLコマンドを使ってリクエストを送信してみましょう。

### Goでリクエストを送る

&#x20;　さきほどcURLを使って送信したリクエストを今度はGoを使って送信してみます。Replitで新しいプロジェクトを作成して（[第2回](2-api-server.md#huan-jing-gou-zhu)#環境構築の2\~3）、`main.go`に以下のプログラムを入力してください。そして、実行してください。

```go
package main

import (
	"fmt"
	"net/http"
	"strings"
)

const WEBHOOK_URL = "<自分のWebhook URL>"

func main() {
	body := `{"text": "Hello, world!"}`
	http.Post(WEBHOOK_URL, "application/json", strings.NewReader(body))
}

```

　cURLのときと同じくSlackの#22-infra-sandboxチャンネルに「Hello, world!」というメッセージが投稿されているはずです。

　このプログラムでも重要なのは次の2行です。

```go
	body := `{"text": "Hello, world!"}` // [1]
	http.Post(WEBHOOK_URL, "application/json", strings.NewReader(body)) // [2]
```

まず\[1]ではHTTPリクエストボディとして送信する文字列を定義しています。ここでは、\[2]の`body`にこの文字列がそのまま入ると考えて問題ありません。次に\[2]では実際にリクエストを送信しています。()内のカンマで区切られた単位を引数と呼びますが、それぞれの引数は次のことを指定しています。

* 第1引数 `WEBHOOK_URL`: リクエストを送信するURL
* 第2引数 `"application/json"`: ボディーに含めるデータの種類
  * MIMEタイプとよばれる仕様を使って、画像・音声・テキストデータなどの種類を指定します
  * cf. [https://developer.mozilla.org/ja/docs/Web/HTTP/Basics\_of\_HTTP/MIME\_types/Common\_types](https://developer.mozilla.org/ja/docs/Web/HTTP/Basics\_of\_HTTP/MIME\_types/Common\_types)
* 第3引数 `strings.NewReader(body)`: リクエストで送信するボディーのデータ
  * `strings.NewReader`の説明は省略します

### APIサーバーからリクエストを送る

　ここまでの回で、GoでAPIサーバーを構築する方法、そしてGoでリクエストを送信する方法を学びました。次は、これらを組み合わせてSlackにメッセージを送信する（＝SlackのWebhook URLへリクエストを送信する）APIサーバーを構築しましょう。

　第2回で使用したReplitのプロジェクトを開いて`main.go`を次のように変更してください。第2回の最後のプログラムと比較して、変更した行の色を変えているので参考にしてください。

<pre class="language-go" data-line-numbers><code class="lang-go">package main

import (
	"github.com/labstack/echo/v4"
<strong>	"net/http"
</strong><strong>	"strings"
</strong>)

<strong>const WEBHOOK_URL = "https://hooks.slack.com/services/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"
</strong>
func main() {
	e := echo.New()

	e.GET("/", func(c echo.Context) error {
		id := c.QueryParam("id")
<strong>		body := `{"text": "Hello, World!"}`
</strong><strong>		http.Post(WEBHOOK_URL, "application/json", strings.NewReader(body))
</strong>		return c.String(200, "id: "+id)
	})

	e.Start(":1323")
}

</code></pre>

　そして、プログラムを実行して次のコマンドでAPIサーバーにリクエストを送るとSlackにメッセージが送信されるはずです（ReplitのWebviewパネルの機能で、実行直後に送信されているかもしれません）。おめでとうございます！

```
$ curl <自分のAPIサーバーのURL>
```

　さて、最後に2つほどプログラムを変更して終わりにします。

　まず、せっかくパラメータとしてidの値を受け取っているのにこのプログラムではほとんど活用されていません。そこで、Slackへ送信するメッセージにidの値を含めるようにしましょう。上で示したプログラムの16行目を次のように変更してください。

```go
		body := `{"text": "Hello, ` + id + `!"}`
```

　すると次のコマンドのように、APIにアクセスする際にidをクエリパラメータとして付与しておくとその値がSlackのメッセージに使用されるようになりました。

```
$ curl <自分のAPIサーバーのURL>?id=hoge
```

　次に、プログラムを実行するたびにSlackへメッセージが送信されてしまうのがすこし面倒です。そこで、APIサーバーが受け取るリクエストのHTTPメソッドをGETからPOSTへ変更してみましょう。この変更は、上で示したプログラムの14行目を次のように変更することで行なえます。

```go
	e.POST("/", func(c echo.Context) error {
```

　すると、ブラウザでAPIにアクセスしてもメッセージが送信されなくなりました。この動作確認は、次のコマンドで行うことができます。

```
$ curl -X POST <自分のAPIサーバーのurl>?id=hoge
```

　🎉おめでとうございます！これでPART 1の内容は終わりです。PART 1ではAPIサーバーとは何か、そしてGoとechoでAPIサーバーをどうやって構築するのかについて学びました。これはつまり簡易版Monbanの作成が半分終わったということでもあります。PART 2ではクライアント側、つまり我々が物理的に触ることのできるRaspberry Piを使った内容に進みます。お疲れさまでした
