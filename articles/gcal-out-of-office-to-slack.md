---
title: "Googleカレンダーの不在からSlackに自動で勤怠連絡する"
emoji: "⛳"
type: "tech"
topics: ["gas", "slack"]
published: true
published_at: 2023-12-13
---

:::message
これは[Google Apps Script Advent Calendar 2023](https://qiita.com/advent-calendar/2023/google-apps-script)の13日目の記事です。
:::

## はじめに

みなさんの会社では勤怠連絡はどのようにしていますか？

カレンダーに記載しておく、チャットで連絡するなどいろいろと方法はあると思いますが、これは結構面倒だったり忘れたりすることがあります。現在僕が所属している会社では、カレンダーに休暇や早退などの予定を入れておいて、前日までにSlackで勤怠連絡をするという方法を取っています。

しかし、我々プログラマーにとってこのような「決まりきったことを手動で行う」という業務は精神的苦痛が非常に大きく、避けるべきです。

そこで今回、Google Apps Script（GAS）とSlack APIを使って自動化しました。

## システムの概要

最初に、今回作成したシステムの概要を説明します。

シーケンス図にまとめるとこんな感じです。

```mermaid
sequenceDiagram
  participant trigger as Time Base Trigger
  participant gas as GAS
  participant calendar as Googleカレンダー
  participant slack-api as Slack API
  participant slack as Slack
  actor user as チームメンバー
  trigger->>gas: 1日に1回実行
  gas->>calendar: 翌日の不在予定を取得
  calendar->>gas : 不在予定
  gas->>gas: 取得した不在予定から<br>勤怠連絡を作成
  gas->>slack-api: 勤怠連絡を送信
  slack-api->>slack: 勤怠連絡
  slack->>user: 勤怠連絡
```

仕組みはいたってシンプルで、Googleカレンダーの不在予定を取得して、それをもとにSlackに勤怠連絡を送信するだけです。

## Googleカレンダーの不在予定を取得

今回、実装にはGoogle Apps Script（GAS）を使用しました。GASであれば実行環境を用意する必要がなく、認証・認可まわりもとくに考える必要がないためです。

その上で少々面倒だったのが、GASが標準で用意しているGoogleカレンダーのAPIには不在予定を判断する昨日がないことです。取得自体はできるのですが、他の予定と区別する方法がありません。予定のプレフィックスとして `不在` をつけるといった「運用でカバー」もしたくはありません。

そこで、今回はサービスからGoogle Calendar APIを有効化し、不在予定のみを取得するようにしました。

![Google Calendar API](https://storage.googleapis.com/zenn-user-upload/2864ba38da3f-20231203.png)

今回利用するのは、その中でも[Events: list](https://developers.google.com/calendar/api/v3/reference/events/list)です。

https://developers.google.com/calendar/api/v3/reference/events/list

このAPIには `eventTypes` というパラメーターがあり、ここに `outOfOffice` を指定すると不在予定のみを取得できます。

GASでの実装は以下のようになります。

```js: code.gs
const events = Calendar.Events.list('自分のGmailアドレス', { // 不在予定は自分のGmailアドレスに紐づいているので自分のGmailアドレスを指定
  eventTypes: 'outOfOffice', // 不在予定のみ取得
  timeMax: '2023-12-02T00:00:00+09:00', // 取得したい日の翌日の00:00:00
  timeMin: '2023-12-01T00:00:00+09:00', // 取得したい日の00:00:00
  timeZone: 'JST', // レスポンスがJSTで返ってくるようにJSTを指定
  singleEvents: true, // orderByをしていするにはtrueにする必要がある
  orderBy: 'startTime', // 時刻の照準に並べたいのでstartTimeを指定
});
```

たとえば、2023年12月1日の不在予定を取得したい場合は、 `timeMax` に `2023-12-02T00:00:00+09:00` 、 `timeMin` に `2023-12-01T00:00:00+09:00` を指定します。

また、時刻順で返ってきてほしいので `orderBy` に `startTime` を指定します。ただし、これをしていするには `singleEvents` に `true` を指定する必要があります。

すると、 `events.items` に不在予定が配列で格納されているので、これをもとに勤怠連絡を作成します。

各 `events.items` には 開始時刻と終了時刻が `start.dateTime` と `end.dateTime` にそれぞれISO 8601形式で格納されているので、これをもとに勤怠連絡を作成します。

## Slackアプリの用意

次に、Slackに勤怠連絡を送信するためにSlackアプリを用意します。

そのためには、[chat.postMessage](https://api.slack.com/methods/chat.postMessage)を使用します。

https://api.slack.com/methods/chat.postMessage

細かい設定は割愛しますが、今回はbotとしてではなくユーザーの代わりに投稿してほしいので、OAuth & Permissionsで以下のスコープを設定します。

![User Token Scopes](https://storage.googleapis.com/zenn-user-upload/8321bceda878-20231203.png)

マニフェストファイルでは以下のように記述します。

```yml:app-manifest.yml
oauth_config:
  scopes:
    user:
      - chat:write
```

## User OAuth Tokenの発行

このあとUser OAuth Tokenが必要になるので、発行されたUser OAuth Tokenを控えておきます。

User OAuth Tokenは、OAuth & Permissionsのページの上部にある `OAuth Tokens for Your Workspace` から確認できます。

あとでGASのソースコードを他の人に共有するなどTokenをハードコーディングしたくない場合、[Properties Service](https://developers.google.com/apps-script/reference/properties)を使うと便利です。

https://developers.google.com/apps-script/reference/properties

以下は、Properties Serviceを使ってUser OAuth Tokenを保存する例です。こうすると、GASを実行したユーザーのみがUser OAuth Tokenを取得できるようになります。

```js: code.gs
// ユーザープロパティーにSLACK_USER_OAUTH_TOKENを保存
PropertiesService.getUserProperties().setProperty(
  'SLACK_USER_OAUTH_TOKEN',
  'xoxp-xxxxxxxxxxxx-xxxxxxxxxxxx-xxxxxxxxxxxx-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
);
// ユーザープロパティーからSLACK_USER_OAUTH_TOKENを取得
PropertiesService.getUserProperties().getProperty('SLACK_USER_OAUTH_TOKEN');
```

:::message alert
User OAuth Tokenを利用すると、紐づいたユーザーとして許可された操作を行えます。絶対に第三者に共有しないようにしてください。
:::

## Block Kit Builderでメッセージを作成

今回、勤怠報告のフォーマットは終日不在の場合と特定時間帯のみ不在の場合の2パターンを用意しました。

```markdown: 終日不在の場合
明日は終日不在です。
```

```markdown: 特定時間帯のみ不在の場合
明日は以下の時間帯不在の予定です。
- 10:00-11:00
- 13:00-14:00
```

Slack APIでリッチテキストを送信するには、[Block Kit](https://api.slack.com/block-kit)を使用します。

https://api.slack.com/block-kit

以下は、上記のようなメッセージを[Block Kit Builder](https://app.slack.com/block-kit-builder)で作成したものです。

```json: 終日不在の場合
{
  "blocks": [
    {
      "type": "section",
      "text": {
        "type": "plain_text",
        "text": "明日は終日不在です。",
        "emoji": true
      }
    }
  ]
}
```

```json: 特定時間帯のみ不在の場合
{
  "blocks": [
    {
      "type": "rich_text",
      "elements": [
        {
          "type": "rich_text_section",
          "elements": [
            {
              "type": "text",
              "text": "明日は以下の時間帯不在の予定です。"
            }
          ]
        },
        {
          "type": "rich_text_list",
          "style": "bullet",
          "elements": [
            {
              "type": "rich_text_section",
              "elements": [
                {
                  "type": "text",
                  "text": "10:00~11:00"
                }
              ]
            },
            {
              "type": "rich_text_section",
              "elements": [
                {
                  "type": "text",
                  "text": "13:00~14:00"
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

上記のJSONをBlock Kit Builderに貼り付けると、実際にどのようなメッセージが送信されるか確認できます。

![Blok Kit Builder](https://storage.googleapis.com/zenn-user-upload/5bc3e2b3ae5a-20231203.png)

https://app.slack.com/block-kit-builder

## GASからSlack APIを呼び出す

ここまで来たら、あとはGASからSlack APIを呼び出すだけです。

GASにはという[URL Fetch Service](https://developers.google.com/apps-script/reference/url-fetch)HTTPリクエストを送信するためのサービスがあります。

https://developers.google.com/apps-script/reference/url-fetch

[chat.postMessage](https://api.slack.com/methods/chat.postMessage)にしたがってリクエストを作成し、送信します。

```js: code.gs
const token = PropertiesService.getUserProperties().getProperty('SLACK_USER_OAUTH_TOKEN')
UrlFetchApp.fetch('https://slack.com/api/chat.postMessage', {
  contentType: 'application/json',
  method: 'post',
  headers: {
    Authorization: `Bearer ${token}`,
  },
  payload: JSON.stringify({
    channel: '投稿先のチャンネルID',
    blocks: JSON.stringify(blocks), // Block Kit Builderで作成したJSONを文字列に変換
  })
})
```

`投稿先のチャンネルID`はどうやったらわかるかというと、Slackのチャンネルを右クリックして**コピー > リンクをコピー**を選択すると、チャンネルのURLがコピーできます。この中の `<channel-id>` がチャンネルIDになります。

```markdown: チャンネルのURL
https://<workspace-id>.slack.com/archives/<channel-id>
```

すると、Slackにメッセージが投稿されます。botではなく、ユーザーとして投稿されていることがわかります。

![Slackの投稿](https://storage.googleapis.com/zenn-user-upload/f0b5767c3869-20231203.png)

## Triggerを設定する

最後に、定期的にGASを実行するためのTriggerを設定します。

UI上からも設定できますが、コードで書くと以下のようになります。

```js: code.gs
ScriptApp
  .newTrigger('関数名') // 実行する関数名を指定
  .timeBased() // 時間ベースのTriggerを作成
  .atHour(15) // 毎日15:00~16:00に実行
  .everyDays(1) // 1日おきに実行
  .create(); // Triggerを作成
```

これで、毎日15：00頃に翌日の不在予定をSlackに自動投稿できるようになりました。

## まとめ

今回は、GASを使ってSlackに自動投稿する方法を紹介しました。

Google Workspaceを利用している場合、GASを使うことで簡単に業務の効率化ができます。今回はGoogleカレンダーとSlackの連携でしたが、他にもさまざまなサービスを利用できるので、ぜひ試してみてください。
