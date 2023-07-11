---
title: "ChatGPTのCode Interpreterはどこまでできるのか"
emoji: "📝"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["chatgpt", "openai"]
published: true
---

:::message
この記事は2023/07/09時点での内容になります。今後のChatGPTのアップデートによってこの記事での検証結果は変化する可能性があります。
:::

先日（2023/07/07）、OpenAIの公式Twitterアカウントから以下のアナウンスがあった。

https://twitter.com/OpenAI/status/1677015057316872192

そこで自分のアカウントの設定画面を見てみると、どうもすでにCode Interpreterがすでに利用できるようだったので、何ができて何が出来ないのか遊んでみた。

https://twitter.com/Yuhei_FUJITA/status/1677222860975394817

## ChatGPTのCode Interpreterとは

そもそもこのCode Interpreterは何ができるのか、[さきほどのツイート](https://twitter.com/OpenAI/status/1677015057316872192)には以下のように書かれている。

> It lets ChatGPT run code, optionally with access to files you've uploaded. You can ask ChatGPT to analyze data, create charts, edit files, perform math, etc.

僕は英語が読めないのでGhatGPTに翻訳してもらった。

> それにより、ChatGPTはコードを実行し、オプションであなたがアップロードしたファイルにアクセスすることができます。あなたはChatGPTにデータを分析させたり、チャートを作成したり、ファイルを編集したり、数学的な計算を行わせることができます等。
> https://chat.openai.com/share/0e130d2e-2cfc-4f61-935b-26ea6a48b3d6

ようは、ChatGPTにテキストではなく入力としてファイルをアップロードできるようになるということだ。他にも任意のソースコードを実行することができる。

実はこれ、機能としてはChatGPT Pluginと一緒にアナウンスされていたので「やっと来たか」という感じだった。

https://openai.com/blog/chatgpt-plugins#code-interpreter

## 有効にする方法

このCode Interpreterを利用するには、以下の上限が必要となる。

- ChatGPT Plusに課金していること
- 設定で有効にしていること
- waitlistに登録していること（？）

まず大前提として、この機能はChatGPT Pluginと同様、ChatGPT Plus（20USD/month）に課金している必要がある。また、これはまだBeta機能ということもあってか設定画面から手動で有効にしないと利用できない。

![Code Interpreterを有効にするための設定画面のスクリーンショット](https://storage.googleapis.com/zenn-user-upload/6e35078df491-20230709.png)

さらに、[さきほどのツイート](https://twitter.com/OpenAI/status/1677015057316872192)にもある通り、Code Interpreterが使えるようになるのは来週中（2023/07/10〜2023/07/16）とのことなので、ChatGPT Plusに課金している人全員がまだ使えるわけではないらしい。おそらく以前あったwaitlistに登録していたため、先行して利用できているような気がする。

## 実際にやってみた

というわけで、せっかく先行利用できるので何ができるのか試してみた。

:::message
この先、実際にCode Interpreterを使ってみた結果はすべて共有リンクを掲載していますが、実行結果の一部は非表示になってしまうようです。プロンプトの参考程度に参照してください。

![Code Interpreterのチャットを共有するダイアログ](https://storage.googleapis.com/zenn-user-upload/003527621e02-20230709.png)
:::

### QRコードの生成

まずは[さきほどのツイート](https://twitter.com/OpenAI/status/1677015057316872192)などにもあったPythonのソースコードの実行を試してみた。

なるほど、動作としてはこういう流れになるようだ。

1. Pythonのソースコードを出力
2. Code InterpreterがPythonのソースコードを実行
3. 実行結果（画像）を出力

![ChatGPTにQRコードを出力させたチャット](https://storage.googleapis.com/zenn-user-upload/20b331c603af-20230709.png)

https://chat.openai.com/share/0dc61efe-b581-41cc-bf55-70644f29a1a7

文字列以外でも出力可能なのはありがたい。

### Pythonによる計算

次に試してみたのは[OpenAIの記事](https://openai.com/blog/chatgpt-plugins#code-interpreter)にあったプロンプト。

![Code Interpreterによる計算](https://storage.googleapis.com/zenn-user-upload/08b37ae8f639-20230709.png)

https://chat.openai.com/share/6f407e0b-7a8a-447c-a41f-9740d33c305f

以前からChatGPTは計算が苦手と言われてきたが、これで計算もできるようになったみたいだ。

### グラフの出力

他にも[OpenAIの記事](https://openai.com/blog/chatgpt-plugins#code-interpreter)の動画内で登場していたグラフの出力も試してみた。

![y = 1/sin(x)のグラフを出力](https://storage.googleapis.com/zenn-user-upload/bce2b49c478d-20230709.png)

https://chat.openai.com/share/c5a8f90b-2722-4296-8ba8-03e18ac4cce9

なるほど、numpyのような有名なライブラリは利用できるらしい。

### Pythonファイルの実行

試しに、ChatGPTが出力したソースコードをそのままアップロードしてみた。

![アップロードしたPythonのソースコードを実行](https://storage.googleapis.com/zenn-user-upload/6315f8a15ca4-20230709.png)

https://chat.openai.com/share/4803fcc2-f3de-4515-b118-2a2b7d1e94c7

### CSVファイルからグラフ描画

次に試してみたのはCSVファイル。今回は[e-Statで公開されている人口データ](https://www.e-stat.go.jp/stat-search/files?stat_infid=000031524010)を利用させてもらった。

![CSVファイルをアップロードし、グラフを出力](https://storage.googleapis.com/zenn-user-upload/f98c8aefa055-20230709.png)

https://chat.openai.com/share/7a248e54-fb17-43c2-bd69-27b5b0161c0a

スクリーンショットを見ると手こずっていることに気づくかもしれないが、なんとかグラフを出力することができた（グラフの正確性までは未検証）。

ちなみに[今回利用したCSVファイル](https://www.e-stat.go.jp/stat-search/files?stat_infid=000031524010)、[このチャット](https://chat.openai.com/share/07a42c6e-54c6-4ce7-be80-33de6e36bc9f)でもChatGPTに指摘されているとおりCSVとして正しくないのだ（そういうところだぞ）。それにもかかわらず、内容を理解して読み込めているのは素直にすばらしい。

なお、画像（グラフ）の描画日本語を利用することはサポートしているフォントの都合上、現状ではできないようだ。

### PDFファイルの読み込み

もう少し複雑なファイルとして、PDFファイルを読み込んでみた。

今回は[厚生労働省が公開しているPDFファイル](https://www.mhlw.go.jp/toukei/list/dl/20-21-h29.pdf)を読み込ませた。

![PDFの要約](https://storage.googleapis.com/zenn-user-upload/2d79588d74e5-20230709.png)

https://chat.openai.com/share/d3b46ec1-2c29-432b-804d-a0fd1c3ca9b7

テキスト部分であれば読み込むことができていそうだ。ただし、画像やグラフ部分まで読み込めているわけではなさそうだ。仕様書や取り扱い説明書を読み込ませて回答させるみたいなことはある程度はできそうだが、レイアウトに依存する。

### ファイル変換

CSVの読み込みができることはわかったので、今度はそれを別のファイル形式に変換してみた。

今回はさきほど利用した[e-Statで公開されている人口データ](https://www.e-stat.go.jp/stat-search/files?stat_infid=000031524010)から特定のデータのみをExcelファイルとして出力してもらった。

![CSVファイルからExcelファイルを出力](https://storage.googleapis.com/zenn-user-upload/a905018f7313-20230709.png)

https://chat.openai.com/share/aa599ba0-ce7d-492b-995d-ddf8e4d5e919

細かい検証はしていないが、Excelファイルとして出力できた。先ほども述べた通り、このCSVファイルは**注意書きをCSVファイル内に記述するという正気の沙汰とは思えないこと**をしている。それでも欠損値として理解し、処理できているのはやはりすごい。

![CSVファイルから出力させたExcelファイル](https://storage.googleapis.com/zenn-user-upload/39b38c6d808e-20230709.png)

### スライドの作成

ExcelがいけるならPowerPointもいけるだろうということで、OpenAIの紹介スライドを作成してもらった。

![](https://storage.googleapis.com/zenn-user-upload/9334903a04f1-20230709.png)

https://chat.openai.com/share/5c065cb9-c17a-4648-a6da-105bc7613bb3

問題なくできた。完成度はそこまで高くないが、これはどちらかというとライブラリに依存するのでChatGPTの能力というわけではない。

画像出力ができていないが、これはプロンプトを工夫すればいけるような気がする。

![](https://storage.googleapis.com/zenn-user-upload/cd9780e28259-20230709.png)

### 画像ファイルの処理

次は[OpenAIの記事](https://openai.com/blog/chatgpt-plugins#code-interpreter)の動画にもあった画像処理を試してみた。

![画像に対してフィルター処理を実施](https://storage.googleapis.com/zenn-user-upload/75bf72652109-20230709.png)

https://chat.openai.com/share/7d8968a2-277d-4b52-9cf6-79d5c2ba02ff

読み込んだ画像に対してPythonを実行することでフィルター処理なども可能だ（最後触れてはいけなさそう画像が出力されているがそっとしておこう）。

今回興味深かったのが、処理に失敗しても自らトライ＆エラーで解決を試みているところだ。

### 動画ファイルの処理

画像がいけるなら動画もいけるだろうということで、適当な動画をアップロードして写真同様に処理を試してみた。

![動画の変換](https://storage.googleapis.com/zenn-user-upload/96d85edc3fb3-20230709.png)

https://chat.openai.com/share/84df0b6a-2127-4104-9879-ce278e536c22

結果から言うと、メモリ不足になりがちだが工夫次第でそれなりのことはできた。

また、今回試行錯誤していてわかったのが、条件は不明だが実行環境が途中で破棄されることがあるらしい。

参考までに、元動画と出力した動画をYouTubeにアップロードしておいた。

https://youtu.be/nRUo8awOQlQ

https://youtu.be/JRo2gj1aSVU

https://youtu.be/1Btej1d3CfU

## Code Interpreterの限界

この記事を書く上で紹介し切れていないいろいろなプロンプトを試していったところ、現段階でのCode Interpreterの限界も見えてきた。

### 実行できるのはPythonのみ

Code Interpreterで実行できるのはあくまでPythonのみ。

また、numpyのような有名なライブラリは利用できるが、すべてのライブラリを利用できるというわけではない。Code Interpreterは外部のネットワークから隔離されたサンドボックス環境下でソースコードを実行する。そのため、以下のような処理が必要なPythonのソースコードは実行出来ない。

- `pip install`など外部ライブラリのインストールが必要なソースコード
- Web APIを呼び出すソースコード

これは外部ライブラリを利用してOCR処理を試みてみた例。

![](https://storage.googleapis.com/zenn-user-upload/9ae8ff5efe40-20230709.png)

https://chat.openai.com/share/c4ef918e-d30a-487c-9f1b-29963a037c7a

### 内容を理解できるのはあくまでテキストのみ

今回のファイルアップロードで「ついにマルチモーダルにも対応したのか」と思っていたが、どうやらそういうわけではなさそうだった。

試しにアップロードした画像の説明やテキスト抽出（OCR）を試してみたが、できなかった。前述した通り画像や動画を処理することはできるが、あくまでPythonを介しての処理であり、画像や動画の内容を理解しているわけではない。

![](https://storage.googleapis.com/zenn-user-upload/328b89619cad-20230709.png)

https://chat.openai.com/share/323b1df8-d4a0-44d6-aecc-06db4bcf543d

また、PDFファイルの読み込みは可能だが「テキストベースのPDF」に限定されるようだ。画像だらけだったり、スライドをPDF出力したようなPDFファイルは読み込みに失敗する傾向にある。

### 大量のファイルやサイズの大きなでファイルは扱えない

当たり前といえば当たり前だが、Pythonの実行環境に与えられるリソースには限界がある。そのため、大量のファイルやサイズの大きなファイルを処理しようとするとエラーになってしまう。

また、実行環境は条件は不明だがある程度で破棄されるため、あまり長いやり取りをすると途中で破棄されてしまうこともある。

### 実行環境は維持されない

以下のツイートで指摘されたので追記。

Pythonの実行環境は一定時間経過したりセッションが切れると破棄されてしまう。Code Interpreterで出力したファイルが必要であれば、ダウンロードしておく必要がある。

https://twitter.com/msmt_kny/status/1678240508752302080

## まとめ

まだBeta版ということで発展途上ではあるが、Code InterpreterによってChatGPTがさらに進化した。

今までもChatGPTにソースコードを出力してもらってそれを手元で実行して…みたいなことでできていたが、これがすべてChatGPT上で完結できるのは思考との速度感的にはかなり嬉しい。素Python+αでできることはこれでほぼできるということなので、利用の幅もかなり広がったように思う。

余談だが、僕が個人的に開発していたQRコードを出力できるChatGPT Pluginはこれにて役目を完全に終えた。
