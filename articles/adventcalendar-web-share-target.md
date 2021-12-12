---
title: "Web Share APIでPWAに共有機能を実装する"
emoji: "🐕"
type: "tech"
topics: ["PWA", "pwanight", "frontend", "adventcalendar"]
published: true
---

:::message
これは [PWA（Progressive Web Apps） Advent Calendar 2021](https://qiita.com/advent-calendar/2021/pwa) の 13 日目の記事です。
:::

Web アプリを PWA 化することで様々な機能を実装することができるようになってきたが、その中の一つにスマホでよく見かける「共有機能」がある。

今回は、そんな共有機能を PWA で実装する方法を紹介する。

## PWA から共有したいときは Web Share API を使う

PWA に限らず、Web アプリからスマホでよく見かける共有メニューを開くには、 `Web Share API` を利用することで実現できる。

### 実装方法

実装するには、 `navigator.share()` を呼び出すことで可能となる。

```typescript:index.ts
const shareContent = async (title: string, text: string, url: string): Promise<void> => {
    if(nagigator.share) {
        try {
            await navigator.share({ title, text, url });
            console.log("successed")
        } catch (err) {
            console.log("failed")
        }
    } else {
        console.log("Web Share API not support")
    }
};

```

あとは、この `shareContent()` に対して共有したいコンテンツを渡して呼び出せばよい。

共有時には `title` , `text` , `url` の３つの値を共有することができる。

詳しい仕様については以下を参照。

https://developer.mozilla.org/ja/docs/Web/API/Navigator/share

### メジャーなブラウザはサポート

対応ブラウザに関しては、Chrome, Safari, Edge など、メジャーなブラウザはサポートしている。

https://caniuse.com/web-share

### Web Share API Level 2 では画像の共有も可能に

`Web Share API` では基本的にテキストしか共有できないが、実装されようとしている `Web Share API Level 2` では、画像の共有もサポートされる予定となっている。

現在だと試せるブラウザが Chrome のみとなっているが、「画像を加工して他のアプリに共有」といった機能は需要があると思うので、実装が待ち遠しい。

https://chromestatus.com/feature/4777349178458112

## PWA に共有したいときは Web Share Target API を使う

逆に、他のアプリから PWA で実装したアプリケーションに対して何かを共有したい場合、 `Web Share Target API` を利用することで実装することができる。

### `manifest.json` に定義することで実装する

以下は、実際に Web Share Target API を使いたい場合の `manifest.json` の実装例。

```json:manifest.json
{
    "share_target": {
        "action": "/foo",
        "method": "GET",
        "params": {
            "title": "title",
            "text": "text",
            "url": "url"
        }
    }
}
```

各パラメータの説明は以下の通り。

| パラメータ | 説明                                                                                     |
| ---------- | ---------------------------------------------------------------------------------------- |
| `action`   | 共有時に                                                                                 |
| `method`   | HTTP Method、 `GET` と `POST` が指定できる                                               |
| `params`   | どのように情報を渡すかの定義、 `method` で `GET` を指定した場合は `query` として渡される |

これを `manifest.json` に記述している場合、他のアプリの共有メニューの候補として PWA が表示されるようになる。

実際にこのような `manifest.json` が定義された PWA に対して共有メニューからコンテンツを共有すると、以下のような HTTP リクエストが PWA に対して送信される。

```bash
curl GET 'https://example.com/foo?title=aaa&text=bbb&url=ccc'
```

あとは、このリクエストを PWA 側で解析し、好きなように扱うことができる。

より詳しいことは、以下の記事を参考にすると詳細が書かれている。

https://web.dev/web-share-target/

### モバイル版 Chrome はサポートされるも Safari はまだ

気になる対応ブラウザについては、以下のように記載されている。

> As of early 2021, the Web Share Target API is supported by:
>
> - Chrome and Edge 76 or later on Android.
> - Chrome 89 or later on Chrome OS.
>
> On all platforms, your web app has to be installed before it will show up as a potential target for receiving shared data.

https://web.dev/web-share-target/#browser-support

残念ながら Safari はサポートしていないが、Android や Chrome OS であればほぼすべてカバーされているといってもいい。

## 最後に

PWA とネイティブアプリの差は着実に小さくなってきたが、共有機能に関してもネイティブアプリとかなり近いものが実現できるようになってきた。

SNS やコミュニケーションツールでは特に共有機能は需要が高いと思うので、ぜひ取り入れていきたい。
