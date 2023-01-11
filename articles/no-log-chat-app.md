---
title: "【受験生が作る】リアルタイム性を重視した新感覚チャットアプリ「のーろぐちゃっと」 | Next.js + WebSocket"
emoji: "💬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - "nextjs"
  - "websocket"
  - "個人開発"
published: true
---

## きっかけ

受験勉強に疲れて開発に飢えていたその時、閃いてしまったのです、このアプリのアイデアを...！開発あんまめんどくなさそうだし作るっきゃねぇなという思いでサクっと作りました。

:::message
「サクっと」とは言っても[Code Time](https://www.software.com/product/code-time)のレポートによれば12時間ぐらいはこのアプリの開発にかけてたらしいです。
:::

ちなみにというかタイトルに書いてますが自分は受験生です。この記事が公開される3日後ぐらいに共通テストを控えてます。ホントなにやってんでしょうね。

## 作ったもの

「のーろぐちゃっと」というリアルタイムチャットアプリを作りました！！

https://no-log-chat.vercel.app

普通のチャットアプリとは違ってログが直近3件までしか残らないという仕様で、リアルタイムさを追及した設計になっています。オープンチャットなので思いがけない人と会えるかも...？

## 技術的な話

### 技術の選定

サクっと作りたかったのでフロントエンドは自分が慣れてる[Next.js](https://nextjs.org/)で作りました。ただUIは目新しいのが好きなので[Mantine UI](https://ui.mantine.dev/)を使って、React Contextがちょい使いづらかったので状態管理には[Jotai](https://jotai.org/)を採用しました。

使ってみた感想ですが、Jotaiは[Redux](https://no-log-chat.vercel.app/)を使うほどでもないけどReact Contextだと物足りない/使いづらいという人にかなりマッチするなと。こういうちょっとしたアプリには最適なんじゃないでしょうか。

Mantine UIは前からちょっと気になってたので初めて触れた～という感じです。見た目はイケイケで、開発体験も悪くない。コンポーネントも結構多い上にコンポーネントを利用したUIのテンプレが用意されてるのが便利✨(今回は使ってないけど) hooksも結構充実してて「実装できるけどちょっとめんどくさい」ぐらいのことが簡単にできたりします。

ESLintとかの設定をゼロからするのは時間とやる気と労力が足りないので大人しく[先人のテンプレートリポジトリ](https://zenn.dev/rabbit/articles/8a0f5e199be76d)を借りました。はねつきとうかさんに感謝🙏

バックエンドはNext.jsのAPI Routesでやる予定だったんですが、WebSocketを扱えないため断念。Next.jsのカスタムサーバーを使えば実現できるんですが、[Vercel](https://vercel.com/)でホスト出来なくなっちゃうので断念。結局[Deno](https://deno.land/)で書いて[Deno Deploy](https://deno.land/)にホストするという形に落ち着きました。

### WebSocketの難しさ

自分はWebSocketをほとんど使ったことがないので、何を使えばいいのかにまず悩みました。[Socket.IO](https://socket.io/)というWebSocketのラッパーライブラリを使おうかな～とも思ったんですが、そこまでめんどいことをしたいわけではないのでデフォルトのWebSocketを使うことにしました。

さらにNext.jsはコンポーネントをサーバー側で事前にレンダリングする関係上、`useEffect`の中にWebSocketの初期化処理を組み込まなきゃいけないのが少し手間でした。

参考までにWebSocket部分の実装をのっけておきます。省略した部分が多少あるのでご注意。

**クライント側**

```ts
import { useAtom } from 'jotai';
import { atom } from 'jotai';
import { useEffect } from 'react';

// WebSocketのオブジェクトを格納するatom
// この例だとstateでも可だが、他の場所でも使いたいのでatomにした
const socketAtom = atom<WebSocket | null>(null);

export const useChat = () => {
  const [socket, setSocket] = useAtom(socketAtom);

  useEffect(() => {
    setSocket((socket) => {
      // 既に入ってたら新しくsocketを作成しない
      // これをしないと2回以上接続するという変なことになってしまう
      if (socket) return socket;
      return new WebSocket(process.env.NEXT_PUBLIC_API_SERVER as string);
    });
  }, [setSocket]);

  useEffect(() => {
    if (socket) {
      // WebSocketからメッセージを受け取った時の処理
      socket.onmessage = (res) => {
        const payload = JSON.parse(res.data);
        
        // メッセージを受け取った時の処理
      };
    }
  }, [socket]);

  // メッセージを送信する処理
  const sendMessage = (message: string) => {
    socket?.send(
      // メッセージの内容
    );
  };

  return { sendMessage };
};
```

**サーバー側**

```ts
import { serve } from 'https://deno.land/std@0.156.0/http/server.ts';

// clientのWebSocketオブジェクトを格納しておくMap
const clients = new Map<number, WebSocket>();
let clientId = 0;

// client全員にメッセージを送る
const sendMessage = (message: string) => {
  clients.forEach((client) => {
    client.send(message);
  });
};

// WebSocketのハンドラー登録部分
const wsHandler = (ws: WebSocket) => {
  const id = ++clientId;
  clients.set(id, ws);
  // 接続時
  ws.onopen = () => {
    console.log('connected');
  };
  // メッセージ受信時
  ws.onmessage = (e) => {
    const payload = JSON.parse(e.data);
    sendMessage(e.data);
  };
  // 切断時
  ws.onclose = () => {
    clients.delete(id);
  };
};

serve(
  (req) => {
    // Deno.upgradeWebSocketでWebSocket用のサーバーになる
    // HTMLサーバーとWebSocketサーバーを兼ね備えるみたいなこともできるが、今回はフロントエンドのホストはVercelに任せているのでWebSocket専用サーバー
    const { response, socket } = Deno.upgradeWebSocket(req);

    // ハンドラー登録
    wsHandler(socket);

    return response;
  }
);
```

省略してないバージョンが見たい人は[クライアント側](https://github.com/cp-20/no-log-chat/blob/main/src/lib/chat.ts)と[サーバー側](https://github.com/cp-20/no-log-chat-server/blob/master/src/main.ts)でそれぞれGitHubで公開してるので覗いてみてください。

### 反省

時間なさ過ぎてエラーハンドリングとかまともにやってないのが心残りですね。そのうち何とかするかもしれないししないかもしれない。

あと環境構築にちょい手間取っちゃったのも反省ですね。オレオレなテンプレートリポジトリが欲しいところです。

え？受験直前にアプリ開発して記事まで書いて反省してないのかって？してないよ

## さいごに

気になった方は是非[アプリ](https://no-log-chat.vercel.app/)を使ってみて欲しいです！

記事＆ツイートの拡散もよろしくお願いします！！

https://twitter.com/\_\_cp20\_\_/status/1612963057055207425