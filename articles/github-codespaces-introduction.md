---
title: "いつでもどこでもVS Codeが利用できるGitHub Codespaces"
emoji: "💨"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["vscode", "githubcodespaces"]
published: true
---


:::message
これは[Visual Studio Code Advent Calendar 2023](https://qiita.com/advent-calendar/2023/vscode)の25日目の記事です。
:::

## はじめに

皆さんは外出先などで突然VS Codeを使いたくなったことはないでしょうか？僕はあります。

それは職場かもしれませんし、カフェかもしれませんし、オフトゥンの中かもしれません。そんな時に、ブラウザとネットワーク環境さえあればいつものVS Codeが使えるのが[GitHub Codespaces](https://github.com/features/codespaces)です。

## GitHub Codespacesとは

GitHub Codespacesは、GitHubが提供しているクラウドベースの開発環境です。

機能的にもUI的にも普段利用しているVS Codeとほぼ同じため、日常的にVS Codeを利用している人であれば、いつも通りの開発体験を得ることができます。

![GitHub Codespaces](https://storage.googleapis.com/zenn-user-upload/0682c840acaf-20231225.png)

GitHub上の任意のRepositoryをcloneすることができるので、GitHubで管理しているソースコードであればすぐに開発を開始することができます。

https://github.com/features/codespaces

### ブラウザ上で利用できる

GitHub Codespacesはネットワーク環境さえあれば、ブラウザ上でいつでも利用可能です。

そのため、たとえばiPadなどのタブレット端末でも利用することができます。

![iPadで開いたGit減価償却](https://storage.googleapis.com/zenn-user-upload/99f3ed28004a-20231225.png)
*iPadで開いたGitHub Codespaces*

:::message
後述しますが、ローカルのVS Codeをクライアントとして利用し、実行環境としてGitHub Codespacesに接続して利用することもできます。
:::

### 必要なマシンスペックを選択できる

GitHub Codespacesでは、開発に必要なマシンスペックを選択することができます。

たとえばMySQLなどのDBを利用したい場合などは、必要に応じてマシンスペックを上げることができます。

![GitHub Codespacesのオプション](https://storage.googleapis.com/zenn-user-upload/f72d199ba4b4-20231225.png)
*RegionやMachine Typeを選択できる*

最大で16-core / 64GB RAM / 128GB SSDを選択選択できるので、よっぽど重たい処理をしない限りは十分でしょう。

:::message alert
後述しますが、マシンスペックを上げれば上げるだけ金額も高くなるので、必要十分なマシンスペックを選択してください。基本的には `2-core` で十分です。MySQLなどを利用する場合でも、 `4-core` あれば十分動作します。
:::

### 開発環境を統一できる

これはDockerなどを利用する場合と同じですが、開発環境を統一することができます。

利用しているPCのOSが開発チーム内で異なっている場合であっても、GitHub Codespacesを利用すれば開発環境を統一することができます。

また、READMEなどのMarkdownに `[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/<owner>/<repo>)` のように記述することで、GitHub Codespacesで開くボタンを設置することができます。

![Open in GitHub Codespaces](https://storage.googleapis.com/zenn-user-upload/cdcf41639c6f-20231225.png)
*READMEなどにGitHub Codespacesで開くボタンを設置できる*

:::message
もしDev Containerを利用氏ている場合、 `devcontainer.json` を利用してGitHub Codespacesの環境構築を行うこともできます。VS Codeの設定や拡張機能も起動時に読み込ませることができるので、セットアップも非常に簡単です。
:::

https://docs.github.com/en/codespaces/setting-up-your-project-for-codespaces/setting-up-your-repository/facilitating-quick-creation-and-resumption-of-codespaces#creating-a-link-to-resume-a-codespace

## 使い方

それでは早速GitHub Codespacesを利用してみましょう。

### 必要なもの

まずGitHub Codespaces利用するために最低限必要なものを説明します。

- GitHubアカウント
- ブラウザ
- ネットワーク環境
- 支払い方法（カード決済 or PayPal）

GitHub CodespacesはGitHubのサービスですので、GitHubアカウントが必要です。

それともう1つ必要なのが、支払い方法です。詳しい料金は後述しますが、GitHub Codespacesは有料のサービスです。毎月の無料枠はありますが、無料枠を超えた場合は有料になります。そのため、無料枠内でしか利用しないとしても、支払い方法の登録は必要です。

支払い方法は以下のページから設定可能で、カード決済（クレジットカード or デビットカード）かPayPalを利用することができます。

https://github.com/settings/billing/payment_information

:::message
学生の場合など、特定の条件を満たすユーザーは無料で利用できる場合があります。
:::

### GitHub Codespacesの起動

GitHub Codespacesを起動するには、GitHub Codespacesを利用したいRepositoryのページにアクセスします。

1. **Code** をクリックする
2. **Codespaces** タブを開く
3. **+** をクリックする

これだけで、Repositoryがcloneされた状態でGitHub Codespacesが起動します。

![GitHub Codespacesの起動](https://storage.googleapis.com/zenn-user-upload/1acd666de113-20231225.png)

:::message
もしブラウザ上ではなくローカルにインストールしているVS Codeをクライアントとして利用したい場合、コマンドパレット（ `Cmd` / `Ctrl` +  `Shift` + `P` ）から以下のコマンドを実行すると、ローカルのVS Codeをクライアントとして利用することができます。

```text:コマンドパレット
>Codespaces: Open in VS Code Desktop
```

:::

### 設定の共通化

もしGitHub Codespacesの環境をカスタマイズしたり、セットアップ時に任意の処理を実行したい場合、 `.devcontainer/devcontainer.json` に記述することになります。

ここでは、以下のような設定が可能です。

- VS Codeの設定
- インストールしたい拡張機能の定義
- インストールしたい環境の定義（Node.jsなど）
- 起動時などに実行したいコマンド（ `npm install` など）

詳しくは以下のドキュメントを参照してください。

https://code.visualstudio.com/docs/devcontainers/create-dev-container

:::message
Docker Container内でVS Codeを利用できるDev Containerを利用している場合、すでにある `.devcontainer/devcontainer.json` や `docker-compose.yml` などをそのまま利用可能です。
:::

## 料金

GitHub Codespacesは有料のサービスですが、金額は「コンピューティング使用料」と「ストレージ使用料」の2つの料金で構成されています。

### コンピューティング使用料

コンピューティング使用料は、Machine Typeと利用時間によって決まります。

当然ですが、Machine Typeが高いほど料金も高くなります。

| マシンの種類 | Unit of Measure | 含まれる使用量の乗数 | Price |
| --- | --- | --- | --- |
| 2 コア | 1 時間 | 2 | $0.18 |
| 4 コア | 1 時間 | 4 | $0.36 |
| 8 コア | 1 時間 | 8 | $0.72 |
| 16 コア | 1 時間 | 16 | $1.44 |
| 32 コア | 1 時間 | 32 | $2.88 |

https://docs.github.com/en/billing/managing-billing-for-github-codespaces/about-billing-for-github-codespaces#about-billing-for-compute-usage

:::message
デフォルトでは、30分間無操作だと自動的にGitHub Codespacesはidle状態になります。idle中はコンピューティング使用料は発生しません。
:::

### ストレージ使用料

ストレージ使用料は、GitHub Codespacesで利用しているストレージの容量によって決まります。

これは、利用の有無に関わらず、環境を破棄しない限り発生し続けます。

https://docs.github.com/en/billing/managing-billing-for-github-codespaces/about-billing-for-github-codespaces#about-billing-for-storage-usage

:::message
デフォルトでは、idle状態が30日継続した環境は自動的に破棄されます。破棄されるとGitHub Codespaces上のリソース（ `git push` されていないソースコードも含む）は完全に削除されます。
:::

### 無料枠

また、毎月以下の無料枠があります。さすがに毎日すべての開発をGitHub Codespacesで行うと無料枠を使い切ってしまいますが、たまに利用する程度であれば無料枠内で利用することができます。

| アカウント プラン | 1か月あたりのストレージ | 1か月あたりのコア時間 |
| --- | --- | --- |
| GitHub Free | 15 GB/月 | 120 |
| GitHub Pro | 20 GB/月 | 180 |

詳しくは以下のドキュメントを参照してください。

https://docs.github.com/en/billing/managing-billing-for-github-codespaces/about-billing-for-github-codespaces

## 注意点

最後に、GitHub Codespacesを利用する上で注意すべき点をいくつか紹介します。

### 環境を削除するとソースコードも削除される

GitHub CodespacesはGitHubが用意したContainer環境を利用しています。これによってどこからでも利用できますが、環境を削除するとContainer内のリソースはすべて完全に削除されてしまいます。

「せっかく書いたコードが消えてしまった」ということがないように、必ず編集したソースコードはGitHubなどにpushしておきましょう。

:::message alert
GitHub Codespacesの環境は、idle状態が30日経過すると自動的に削除されます。事前にメールで通知が来ますが、消えると困るソースコードは必ずpushするようにしましょう。
:::

### Machine Typeの上げすぎや長時間の起動は控える

GitHub CodespacesはMachine Typeを上げるとよっぽど重たい処理でもない限り、スペック面で困ることはありません。しかし、必要以上にMachine Typeを上げると、料金が高くなってしまいます。

また、GitHub Codespacesはデフォルトで30分間無操作だと自動的にidle状態になります。idle状態になるとコンピューティング使用料は発生しなくなりますが、再起動するのに若干時間がかかります。だからといって起動しっぱなしにしてしまうとずっと課金され続けるので、使わないときはidle状態にしておきましょう。

## まとめ

実はこのZennの記事も、GitHub Codespacesを利用して書いています。むしろ最近はほとんどのコーディングをGitHub Codespacesで行っています。

GitHub Codespacesをデフォルトで利用しておくとマシンスペックを気にしたり、別のパソコンなどからでも瞬時にコーディングの続きを始められるので、非常に便利です。

日常的にVS CodeとGitHubを利用している人は、ぜひGitHub Codespacesを利用してみてください。
