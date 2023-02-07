# 第6回 / Monbanの移植

### この章のゴール

　MonbanをReplitに移植し、動作させる。

{% hint style="info" %}
この文書には、Monbanを移植するためにどのような方針で作業すればよいかが書かれており、ググりながら自分で進めてもらうことを想定しています。
{% endhint %}

### まずは動かす

1. Replitの`Import from GitHub`を使って[https://github.com/arata-nvm/monban-server](https://github.com/arata-nvm/monban-server)をReplitにインポートする
2. ToolsのSecretsから環境変数を設定する
3. 実行する
4. `curl -X POST <URL> -H "Content-Type: application/json" -d '{"student_id": 0}'`をターミナルで叩いてSlackにメッセージが投稿されるか確認する
5. 鯖室のRaspberry Piのプログラム`~/nfc/update.py` のURLを更新する。

上記の手順を踏んでうまく動作したらこの節は終わり。動作しなかったら頑張って原因を探し、修正する。

### 改良のアイデア

　もしやる気があるならば以下の改良をするとMonbanがより扱いやすくなる。

* DBをFirebaseに移行する
  * わざわざGoogle SheetsをDBとして使うメリットは現在ないので、扱いが楽な別のプロダクトを利用したほうがMonbanをメンテナンスしやすい
* DBを使わないように変更する
  * 現在DBは入退室の統計をとって遊ぶぐらいの用途しかないので、SlackへのメッセージだけがほしいならDBをわざわざ使う必要はない
