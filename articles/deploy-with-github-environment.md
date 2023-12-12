---
title: "Environmentを使ってGitHub ActionsのDeploy先を切り替える"
emoji: "⛳"
type: "tech"
topics: ["github", "githubactions"]
published: true
published_at: 2023-12-13
---

:::message
これは[GitHub dockyardコミュニティ Advent Calendar 2023](https://qiita.com/advent-calendar/2023/github-dockyard)の13日目の記事です。
:::

## はじめに

みなさんはCI/CD基盤として何を使っていますか？GitHubを利用していると、GitHub Actionsを利用している人も多いかと思います。

今回は、GitHub ActionsにあるEnvironmentsを利用して、Deploy先を切り替える方法を紹介します。

## Deploy先の違いによるGitHub Actionsの処理の違い

通常、システム開発を行う場合はユーザーに提供する本番環境と開発者が利用する開発環境を用意するのが一般的です。場合によってはさらにステージング環境を用意することもあるでしょう。

| 環境 | 用途 |
| --- | --- |
| 本番環境（prod） | 実際にユーザーに提供される環境。 |
| ステージング環境（stg） | 本番環境に近い環境。本番環境にDeployする前に動作確認を行う。 |
| 開発環境（dev） | 開発者が利用する環境。開発者が自由にDeployできる。 |

これらはDeploy頻度も違えば、Deployされるソースコードにも違いがあります。そのため、CI/CDの処理も環境によって微妙に異なってきます。具体的には以下のようなものがあるでしょう。

- 環境変数
- Buildコマンド
- Deploy先
- 権限

しかし、基本的な処理が大きく変わるということはあまりないです。

## GitHub ActionsのSecrets/Variables

GitHub Actionsには、ハードコーディングしたくない値を保存する機能として、SecretsとVariablesがあります。これは環境変数のようなもので、GitHub Actionsを構成するYAMLファイルから参照できます。

### Secrets

Secretsは名前からもわかるとおり、機密情報を保存するための環境変数のようなものです。Secretsに保存された値は後から確認することができず、GitHub Actionsのlogに出力される場合も自動的にマスク処理されます。

登録は `https://github.com/<owner>/<repository>/settings/secrets/actions/new` から行うか、 `gh` コマンドを利用して行います。


```bash:ghコマンドによるSecretsの登録
gh secret set NAME --body VALUE
```

GitHub ActionsのYAMLファイルからは以下のように参照できます。

```yaml:Secretsの参照
- name: Echo Secrets
  run: echo ${{ secrets.NAME }} # secrets.NAME で参照できる
```

https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions

:::message
このあとにも何度か出てきますが、 `gh` コマンドはGitHubをCLIから操作できる便利なツールです。本記事では詳しく触れませんが、便利なので詳しくは[About GitHub CLI \- GitHub Docs](https://docs.github.com/en/github-cli/github-cli/about-github-cli)を参照してください。
:::

### Variables

VariablesはSecretsと基本的には同じですが、後から値の確認ができる点や、GitHub Actionsのlogに値がそのまま出力される点が異なります。

登録は `https://github.com/<owner>/<repository>/settings/variables/actions/new` から行うか、 `gh` コマンドを利用して行います。


```bash:ghコマンドによるSecretsの登録
gh variable set NAME --body VALUE
```

GitHub ActionsのYAMLファイルからは以下のように参照できます。

```yaml:Variablesの参照
- name: Echo Variables
  run: echo ${{ vars.NAME }} # vars.NAME で参照できる
```

https://docs.github.com/en/actions/learn-github-actions/variables

### Secrets/Variablesの比較

SecretsとVariablesを比較すると、以下のようになります。

|| Secrets | Variables |
| --- | --- | --- |
| 利用方法 | `${{ secrets.SECRET_NAME }}` で利用する | `${{ vars.VARIABLE_NAME }}` で利用する |
| 値の確認 | 後から確認できない | 後から確認できる |
| logに出力 | 自動的にマスクされる | そのまま出力される |
| 用途 | 機密情報を保存する | ハードコーディングしたくない値を保存する |

## Environmentsで環境ごとにSecrets/Variablesを切り替える

ここまででGitHub Actionsからハードコーディングしたくない値をYAMLファイルか排除することはできました。しかし、これだけでは環境ごとに異なる値を利用することはできません。

そんなときに便利なのがEnvironmentsです。これは環境ごとに異なるSecrets/Variablesを定義できる機能です。

設定は `https://github.com/<owner>/<repository>/settings/environments` から行います。

`gh` コマンドから行う場合は `--env` オプションでEnvironment名を指定するだけです。

```bash:ghコマンドによるEnvironmentsに対するSecrets/Variablesの登録
gh secret set NAME --body VALUE --env ENVIRONMENT_NAME
gh variable set NAME --body VALUE --env ENVIRONMENT_NAME
```

そして、GitHub ActionsのYAMLファイルからは以下のように参照できます。

```yaml:Environmentsの使用例
jobs:
  Echo:
    runs-on: ubuntu-latest
    environment: dev # dev環境のEnvironmentsを利用する
    steps:
      - name: Echo Secrets
        run: echo ${{ secrets.NAME }} # devのEnvironmentsで保存したSecretのNAMEが取得できる
      - name: Echo Variables
        run: echo ${{ vars.NAME }} # devのEnvironmentsで保存したVariableのNAMEが取得できる
```

違いは `jobs.<job_id>.environment` に環境名を指定する点だけです。

:::message
`jobs.<job_id>.environment` を指定できるのは各jobの実行時です。jobの途中で切り替えることはできません。
:::

https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment

## GitHub Actions実行時にEnvironmentsを切り替える

ここまででGitHub Actionsの環境変数であるSecrets/VariablesとEnvironmentsによる環境ごとの切り替え方を説明しました。あとは、このEnvironmentsをDeploy先に合わせて切りあえてあげれば、BuildやDeployの処理は共通化しつつ、環境ごとに異なる値をGitHub Actionsに渡すことが可能になります。

今回は、各環境にDeployするBranchの紐づけを以下のようにするものとします。

| 環境 | Branch | 説明 |
| --- | --- | --- |
| 本番環境（prod） | `main` | `main` 以外のBranchは許可しない |
| ステージング環境（stg） | `release*` | `release` から始まるBranchを許可する |
| 開発環境（dev） | `*` | すべてのBranchを許可する |

また、Deployは任意のタイミングで実行できるよう、 `on.workflow_dispatch` を利用します。

```yaml:Deploy先の指定を伴う手動実行
on:
  workflow_dispatch:
    inputs:
      environment:
        required: true
        options:
          - 'dev'
          - 'stg'
          - 'prod'
```

このようにすると、GitHub Actionsを手動で実行することができ、かつDeploy先を選択できるようになります。

https://docs.github.com/ja/actions/using-workflows/events-that-trigger-workflows#workflow_dispatch

また、各Environmentsには以下のような値が保存されているとします。

```bash:devの状態
$ gh variable list --env dev
NAME            VALUE            UPDATED               
AWS_REGION      us-east-1        less than a minute ago
BUILD_MODE      develop          less than a minute ago
S3_BUCKET_NAME  web-hosting-dev  less than a minute ago

$ gh secret list --env dev
NAME          UPDATED               
AWS_ROLE_ARN  less than a minute ago
```

```bash:stgの状態
$ gh variable list --env stg
NAME            VALUE            UPDATED               
AWS_REGION      ap-northeast-1   less than a minute ago
BUILD_MODE      stating          less than a minute ago
S3_BUCKET_NAME  web-hosting-stg  less than a minute ago

$ gh secret list --env stg
NAME          UPDATED               
AWS_ROLE_ARN  less than a minute ago
```

```bash:prodの状態
$ gh variable list --env prod
NAME            VALUE             UPDATED               
AWS_REGION      ap-northeast-1    less than a minute ago
BUILD_MODE      production        less than a minute ago
S3_BUCKET_NAME  web-hosting-prod  less than a minute ago

$ gh secret list --env prod
NAME          UPDATED               
AWS_ROLE_ARN  less than a minute ago

```

## 最終的なGitHub Actions

そうしてできたGitHub Actionsの全体が以下になります。

今回はnpmでBuildしたWebアプリをS3にDeployすることを想定しています。

```yaml:Deploy先を切り替えるGitHub Actions
name: Deploy
on:
  workflow_dispatch: # GitHub Actionsを手動で実行できるようにする
    inputs:
      environment: # Deploy先を選択できるようにする
        required: true
        options:
          - 'dev'
          - 'stg'
          - 'prod'

jobs:
  Setup: # on.workflow_dispatch で選択されたDeploy先とBranchからEnvironmentsの設定を行う
    runs-on: ubuntu-latest
    steps:
      - name: Dev
        if: ${{ inputs.environment == "dev" }}
        run: echo "branch=${{ github.ref }}"" >> "$GITHUB_ENV"

      - name: Stg
        if: ${{ inputs.environment == "stg" && startsWith(github.ref, "refs/heads/release") }}
        run: echo "branch=${{ github.ref }}"" >> "$GITHUB_ENV"

      - name: Prod
        if: ${{ inputs.environment == "prod" && github.ref == "refs/heads/main" }}
        run: echo "branch=main" >> "$GITHUB_ENV"

      - name: Error
        if: ${{ env.branch == null }} # どの条件にも当てはまらない場合はエラーにする
        run: echo "Invalid environment and Branch" && exit 1

      - name: Output branch
        run: echo "branch=${{ env.branch }}" >> "$GITHUB_OUTPUT" # 他のjobから参照できるように $GITHUB_OUTPUT に出力する

  Build:
    needs: Setup
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }} # Deploy先に合わせてEnvironmentsを切り替える
    steps:
       - name: Checkout
        uses: actions/checkout@v4
        with:
          ref: ${{ needs.Setup.outputs.branch }} # Setupで出力したbranchを利用する

      - uses: actions/setup-node@v4
        with:
          node-version: "lts/*"

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build --mode ${{ vars.BUILD_MODE }}

      - name: Upload to Artifact
        uses: actions/upload-artifact@v3
        with:
          name: artifact
          path: dist

  Deploy:
    needs: [Setup, Build]
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }} # Deploy先に合わせてEnvironmentsを切り替える
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Download build file from artifact
        uses: actions/download-artifact@v3
        with:
          name: artifact
          path: dist

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v3
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ vars.AWS_REGION }}

      - name: Deploy to S3
        run: aws s3 cp dist s3://${{ vars.S3_BUCKET_NAME }} --recursive
```

ポイントは、Deploy先を決定するjobである `Setup` と、実際にBuildやDeployを行うjobである `Build` と `Deploy` を分けている点です。

これはなぜかというと、 `jobs.<job_id>.environment` はjobを実行するタイミングで指定する必要があり、途中で変更することはできないからです。そのため、先ほど実装した `Setup` とはjobをわけてDeployの処理を実装します。

注目すべきは `Build` と `Deploy` のjobで、Deploy先ごとに微妙にコマンドが異なっているにもかかわらず、その差をEnvironmentsが吸収してくれたおかげでまったく同じjob定義となっている点です。

もしこれを愚直に実装すると、Deploy先ごとに似通ったjobを複数定義する必要があり、メンテナンス性が低くなってしまいます。

:::message
実際はもう少し複雑になり、特定のDeploy先のみ必要なstepがあったりするでしょう。そういったときは、以下のように `if` でstepごとの実行条件を指定すればよいです。

```yaml:ifによるstepの実行条件
- name: Run only on dev
  if: ${{ inputs.environment == "dev" }}
  run: echo "This step is executed only on dev"
```

:::

まとめ

今回は、GitHub ActionsのEnvironmentsを使ったDeploy処理の共通化を紹介しました。

EnvironmentsとSecrets/Variablesを利用することで、GitHub Actions上でも `.env` を切り替えるように環境変数の切り替えが簡単に実装できます。もしGitHub ActionsでCI/CDを実装している場合は、ぜひ試してみてください。
