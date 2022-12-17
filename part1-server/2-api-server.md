# 第2回 / APIサーバーの構築

### この章のゴール

以下の事項について理解する。

* APIサーバー
* Replit
* Go
  * 基本的な文法
  * 実行方法
  * echo

また、アクセスすると引数`id`に渡した値を文字列として返すAPIサーバーを構築する。

### APIサーバーとは

　まず、API（Application Programming Interface）という単語について説明します。APIとは、ソフトウェアの機能を公開し、他のソフトウェアがその機能を利用できるようにするための取り決めです。具体例を挙げると、次のようなソフトウェアでAPIが使われています。

* Google Chromeの拡張機能
  * 拡張機能のアイコンをGoogle Chromeのツールバーに表示する機能
  * 右クリックメニューに拡張機能の項目を追加する機能
  * 拡張機能自体を実行する機能
* MinecraftのMod
  * ブロック・アイテムを追加する機能
  * エンティティを追加する機能
  * レシピを追加する機能

　そして、APIで取り決められた機能をネットワークを介して提供するようなサーバーをAPIサーバーと呼びます。具体的には、以下のようなAPIサーバーがあります。

* Slack API
  * Slackにメッセージを投稿する機能
  * Slackのメッセージにリアクションをつける機能
* LINE Messaging API
  * LINEの指定したユーザーにメッセージを投稿する機能
  * LINE botを作る機能

　実際に、Cat as a serviceというAPIサーバーを使ってAPIの呼び出しを体験してみましょう。次のURLにアクセスしてみてください：[https://cataas.com/cat](https://cataas.com/cat) 。アクセスするたびにランダムな猫の画像がWebブラウザに表示されるはずです。これは、次のような取り決めがなされているAPIです。

* URL: /cat
  * 補足：ここでは https://cataas.com の後ろにつける文字列という認識で問題ありません
* 入力: なし
* 出力: ステータスコード200でランダムな猫の画像を返す

　もう１つ例を挙げましょう。次のURLにアクセスしてみてください：[https://cataas.com/cat/says/hoge?color=red](https://cataas.com/cat/says/hoge?color=red) 。赤い文字で"hoge"と描かれた画像がWebブラウザで表示されるはずです。これは、次のような取り決めがなされているAPIです。

* URL: /cat/says/:text?color=:color
  * 補足：":text"や":color"は、ユーザーが自由に値を指定できることを表しています
* 入力
  * ":text"：文字
  * ":color"：文字色
* 出力
  * ステータスコード200で、ランダムな猫の画像に:color色で:textを描画した画像を返す

{% hint style="info" %}
なお、ここではわかりやすさのため画像を返すAPIサーバーを取り上げましたが、何も返さないAPI、テキスト・音声・バイナリデータを返すAPIサーバーなども実在します。[https://github.com/n0shake/Public-APIs](https://github.com/n0shake/Public-APIs) には公開されている有名なAPIサーバーがまとめられています。
{% endhint %}

#### 演習1

　[https://cataas.com/](https://cataas.com/#/) や [https://cataas.com/doc.html](https://cataas.com/doc.html) を参考にして、例で触れていないCat as a serviceのAPIを呼び出してみましょう。

### 環境構築

　今回の講座では、Webブラウザでコードを編集・実行できるサービスであるReplitを用います。以下の手順に従って環境構築をしてください。

1. [https://replit.com/signup](https://replit.com/signup) から自分のアカウントを作成する。
2.  "Skip"ないし"×"を押して画面を進め、左上の"Create"を押す（図1）。

    <figure><img src="../.gitbook/assets/Screenshot 2022-12-12 at 12.52.01.png" alt=""><figcaption><p>図1: ダッシュボード画面</p></figcaption></figure>

3\. モーダルが表示されたら、"Template"の欄に"Go"を指定して”Create Repl"を押す（図2）。

<figure><img src="../.gitbook/assets/Screenshot 2022-12-12 at 12.52.20.png" alt=""><figcaption><p>図2: モーダルが表示された画面</p></figcaption></figure>

　4. 次のような画面が表示されたら、中央上の"Run"を押す（図3）。

<figure><img src="../.gitbook/assets/Screenshot 2022-12-12 at 12.56.14.png" alt=""><figcaption><p>図3: IDE画面</p></figcaption></figure>

5\. 画面右のパネルに"Hello, World!"と表示されることを確認する（図4）。

<figure><img src="../.gitbook/assets/Screenshot 2022-12-12 at 12.58.10.png" alt=""><figcaption><p>図4: プログラムの実行</p></figcaption></figure>

　おめでとうございます！Goで書かれたプログラムを実行できました。ここまで来れば環境構築は終了です。最後に、画面に表示されているパネルの役割を確認しておきます（図5）。

<figure><img src="../.gitbook/assets/Screenshot 2022-12-12 at 12.58.10 copy.png" alt=""><figcaption><p>図5: パネルの説明</p></figcaption></figure>

| 図中の番号 | 名前     | 概要                                                    |
| ----- | ------ | ----------------------------------------------------- |
| 1     | ファイル一覧 | プログラムを構成するファイルの一覧を表示します。今回の講座では`main.go`のみを使用します。     |
| 2     | エディター  | ファイル一覧で選択したファイルを編集します。このパネルをもっとも使用します。                |
| 3     | コンソール  | プログラムの実行やコマンドの実行のために使用します。このパネルを使用する際には文章中で明示的に指示します。 |
| 4     | 実行ボタン  | プログラムを実行します。                                          |

### Go入門

{% hint style="info" %}
以下では、「プログラムを入力して実行してください」という旨の指示が出されます。コピペでも問題ありませんが、もし暇ならば自分で入力したほうが学習効果が高いです。

cf. [https://qiita.com/yuya\_takeyama/items/b8a8c548a4a1c6a05531](https://qiita.com/yuya\_takeyama/items/b8a8c548a4a1c6a05531)
{% endhint %}

　ここからは、実際にプログラムを書いてAPIサーバーを構築します。APIサーバーの構築にはGo（ [https://go.dev/](https://go.dev/) ）を使用します。GoはGoogleが開発したプログラミング言語で、文法がシンプルなこと、エコシステムが充実していることなどが特徴です。

　さきほど実行したGoのプログラムをもう一度見てみましょう。

```go
package main

import (
	"fmt"
)

func main() { // [1]
	fmt.Println("Hello, World!") // [2]
}

```

　このプログラムで重要な点は2つです。

* \[1] プログラムを実行すると、最初に`main`関数の中（`func main() {`〜`}`）が実行される。
* \[2] `fmt.Println("Hello, World!")`を実行すると"Hello, World!"が画面に表示される。

これで"Hello, World!"を画面に出力するGoのプログラムが書けるようになりました。めでたい。

#### 演習2

`fmt.Println("Hello, World!")`の""の中を変更して、別の文字列を画面に表示させてみましょう。

### APIサーバーを構築する

　続けて、GoのWebフレームワークの1つであるEcho（ [https://echo.labstack.com/](https://echo.labstack.com/) ）を導入します。Webフレームワークとは、プログラミング言語においてWebサービスの開発を容易にするものです。Goは汎用的な言語であるためWebサービスのほかCLIツール、オペレーションツールなど幅広い開発に用いることができます。したがってWebフレームワークを使用せずともGo単体でAPIサーバーを構築できますが、煩雑かつバグを埋め込みやすいです。そこで、今回はWebフレームワークを導入することにします。

　コンソールで次のコマンドを実行してください（図6）。

```bash
$ go get github.com/labstack/echo/v4
```

<figure><img src="../.gitbook/assets/Screenshot 2022-12-12 at 17.35.00.png" alt=""><figcaption><p>図6: コマンド実行後の画面</p></figcaption></figure>

次に、`main.go`を次のように変更してください。

```go
package main

import (
	"net/http"
	
	"github.com/labstack/echo/v4"
)

func main() {
	e := echo.New()
  
	e.GET("/", func(c echo.Context) error {
		return c.String(http.StatusOK, "Hello, World!")
	})
  
	e.Start(":1323")
}

```

最後に、実行ボタンを押してください（図7）。

<figure><img src="../.gitbook/assets/Screenshot 2022-12-12 at 17.40.11.png" alt=""><figcaption><p>図7: 実行中の画面</p></figcaption></figure>

はい、これでAPIサーバーの構築が終わりました。一瞬でしたね。実はReplitの機能によりデプロイまでが自動で行われています。さきほどの作業で一体なにが作られたのか確認してみましょう。

　まず、画面を見てわかるとおり、右上に"Webview"パネルが追加されました。これはAPIサーバーにアクセスした結果を表示しています。この結果が正しいかどうかを確かめるために、パネルのURLバーからURLをコピーして自分のブラウザからアクセスしてみましょう。

{% hint style="info" %}
画像とはURLが異なるので注意してください。
{% endhint %}

{% hint style="info" %}
このURLにアクセスできるようになるまで数十秒程度かかるようです。もし表示されない場合は少し待ってから再度アクセスしてみてください。
{% endhint %}

<figure><img src="../.gitbook/assets/Screenshot 2022-12-12 at 17.50.39.png" alt=""><figcaption><p>図8: APIサーバーにアクセスした結果</p></figcaption></figure>

すると、"Hello, World!"と表示されました（図8）。つまり、アクセスすると"Hello, World!"という文字列を返すAPIサーバーを作ったことがわかります。

　`main.go`に入力したプログラムを読んで、なぜこうなったのかを見てみましょう。一番重要なのは次に示す3行です。

<pre class="language-go"><code class="lang-go">(略)
e.GET("/", func(c echo.Context) error { // [1]
	return c.String(200, "Hello, World!") // [2]
})
<strong>(略)
</strong></code></pre>

まず\[1]で、`GET`メソッドで`/`へリクエストが送られたときに処理を行うことを指示しています。そして\[2]で、ステータスコード`200`で文字列`"Hello, World!"`をレスポンスとして返すことを指示しています。

#### 演習3

\[2]の行を変更して、レスポンスとして返すステータスコードや文字列を変更してみましょう。変更が反映されているかの確認には、以前紹介したDevToolsのNetworkタブを使うとよいでしょう。

### 引数を受け取る

　URLにはクエリパラメータと呼ばれる仕様があり、これを使ってURLに任意の値を含めることができます。Google検索を例に挙げて説明します。[https://www.google.com/?q=hoge](https://www.google.com/?q=hoge) にアクセスすると、検索窓に"hoge"という文字列が入力された状態で表示されます。これは、URLの末尾にある`?q=hoge`でクエリパラメータを指定したためです。`q`という名前の引数に`hoge`という値を渡すことを意味します。

　さきほどのプログラムを次のように変更して引数を受け取るようにしてみましょう。

```go
(略)
e.GET("/", func(c echo.Context) error {
	id := c.QueryParam("id") // [3]
	return c.String(200, "id: "+id) // [4]
})
(略)
```

まず\[1]で、`id`という名前の引数から値を受け取ります。そして\[2]で、文字列”id: ”のあとに受け取った`id`の値を付け加えた文字列を返します。

　プログラムを実行して、URLの末尾に`?id=hoge`をつけてアクセスすると次のような文字列が得られるはずです（図9）。`id`という名前の引数に`hoge`という値を渡したので、"id: hoge"という文字列が返されています。

<figure><img src="../.gitbook/assets/Screenshot 2022-12-12 at 18.22.37.png" alt=""><figcaption><p>図9: 引数を受け取るようにした結果</p></figcaption></figure>

{% hint style="info" %}
プログラムの変更はプログラムを一度停止しないと反映されません。APIサーバーの様子がおかしいときは再実行してみてください。
{% endhint %}
