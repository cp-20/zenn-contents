---
title: "WebUI をサッと作るツールとしての htmx"
emoji: "📨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "html", "htmx"]
published: true
published_at: 2024-12-02 21:30
---

この記事は [SecHack365 Advent Calendar 2024](https://qiita.com/advent-calendar/2024/sechack365) 2日目の記事です。先に断っておくと SecHack365 要素は1ミリもありません、ごめんなさい。

## はじめに

htmx というツール (ライブラリ？) をご存じでしょうか？ちょっと前に話題になったっきりで特に広く使われていることもなく、今や情報の海に溺れている悲しいやつです。

実際に使ってみると確かにプロジェクトの規模にスケールしなさそうだなと思った一方で、簡単に WebUI とかを生やすのには結構向いているのでは？と感じました。なのでこの記事ではそれを布教していきます。

## htmx とは

最近のモダンなWebアプリ (RESTful な SPA を想像してください) はサーバーから JSON 形式でデータを受け取ってそれを JavaScript を用いて画面にレンダリングすると思います。ここでいう JavaScript は素の JavaScript というよりも React や Vue などを使ったものであることが多いでしょう。

それとは対照的に htmx はサーバーから HTML 形式でデータを受け取ります。どこからデータを取ってくるのか、どこに描画するのかといった情報は JavaScript で記述するのではなく、HTML の attributes (`class="btn"` みたいなやつ) で記述します。htmx の解説記事ではないので具体的にどう記述するかには触れませんが、興味がある人は次の記事を読んでみると良いでしょう。

https://zenn.dev/ak/articles/735e05d2ba8da0

htmx の背後にある思想とかを解説した良い記事も合わせて読んでみると良いかもしれません。

https://qiita.com/tsmd/items/0d07feb8e02cfa213cc4

## htmx を使う

### htmx の基本

[公式ページ](https://htmx.org/) からミニマルな例を持ってきました。

```html
<script src="https://unpkg.com/htmx.org@2.0.3"></script>
<!-- have a button POST a click via AJAX -->
<button hx-post="/clicked" hx-swap="outerHTML">
  Click Me
</button>
```

このボタンをクリックすると `/clicked` にリクエストが飛んで、返ってきた内容で DOM を書き換えます。例えば `/clicked` から　`<p>The button was clicked!</p>` が返ってきたとすると、ボタンを押した後の DOM は次のようになります。 

```html
<script src="https://unpkg.com/htmx.org@2.0.3"></script>
<p>The button was clicked!</p>
```

`hx-swap` が `outerHTML` だったので `<button>` をまるごと置き換える形で DOM が書き変わりました。

### びみょいポイント: スケールしない

これが htmx の基本で、DOM 操作をサーバーを経由して行うのが特徴です。普段モダンなフレームワークを使って開発しているみなさまからすると「スケールしなさそ～」と思われるかもしれません。ボクもそう思います。

明らかにサーバーとクライアントが密結合します。特に複雑な DOM 操作をしようとするとこれは顕著になります。密結合すると大規模になってきたときに収集がつかなくなることが知られているので、大きいプロジェクトで htmx を採用することはまずないと思います。

### 嬉しいポイント: コードが短い

サーバーとクライアントが密結合するのは大規模なプロジェクトにおいてはデメリットになりますが、小規模なプロジェクトではむしろ良い方向に働くこともあります。React や Vue のようなフレームワークでさっきの htmx の例と同じことをやろうと思ったらどうなるでしょう？きっとそれなりの行数のコードになるのではないでしょうか？特にサーバー側のコードによって描画するものが変化する場合はさらに行数がかさむことでしょう。

つまりサーバーに処理を送る (そしてその結果をもらう) ことがメインのアプリなら htmx はむしろ向いていると言えるのではないでしょうか？そんなアプリあるかな...？

## WebUI という文脈

タイトルで出オチ感がありますが、サーバーに処理を送るアプリとして代表的なものに WebUI がありますね。別に何に対する WebUI でも良いのですが、ここではイメージのしやすさを考えて特定のコマンドを実行してその結果を見る WebUI を考えてみましょう。

```html
<script src="https://unpkg.com/htmx.org@2.0.3"></script>
<div>
  <div>
    <!-- `/cmd?${encodeURIComponent(command)}` -->
    <button hx-post="/cmd?.%2Fscript1.sh" hx-swap="innerHTML#output">script1</button>
    <button hx-post="/cmd?.%2Fscript2.sh" hx-swap="innerHTML#output">script2</button>
    <button hx-post="/cmd?.%2Fscript3.sh" hx-swap="innerHTML#output">script3</button>
  </div>
  <div id="output">
    ここに結果が表示されます
  </div>
</div>
```

なんとクライアントはたったのこれだけのコードで書けます。サーバーも単に与えられたスクリプトを実行するだけなので、何も難しくはないと思います。ユーザーが触るものならユーザー体験が求められるので話は別ですが、単に動く事が求められているなら htmx の方がむしろシンプルに書けるのではないでしょうか。

### ちょっと応用: 色を付ける

ターミナルに色を付けるために特殊文字を使うことがあると思いますが、htmx を使えばその色をそのままクライアントに渡すことができます。例えば TypeScript でサーバーを書くなら次のようなヘルパー関数を使えば簡単に実現できます。

```ts
const ansiToHtml = (ansiString: string) => {
  const ansiMap: Record<string, string> = {
    '0;34': 'color: blue;',
    '0;32': 'color: green;',
    '0;31': 'color: red;',
    // 必要ならもっと多くの色に対応する
  };

  const ansiRegex = /\x1b\[(\d;?\d*)m/g;

  return ansiString
    .replace(ansiRegex, (_, ansiCode) => {
      const style = ansiMap[ansiCode];
      return style ? `<span style="${style}">` : '</span>';
    })
    .replace(/\x1b\[0m/g, '</span>');
};
```

### 具体例: ISUCON のデプロイツール

htmx を使って ISUCON のデプロイツールの WebUI を作ってみました。基本はデプロイスクリプトを実行するだけなのでこの章の最初の例と同じ感じで、ただログの配信に WebSocket を使っていたり、複数人が同時にデプロイできないように排他制御をしたりしています。でもそれだけやっても全部で250行いかないぐらいに収まってます。すごい。

![Remote Provisioning Controller](/images/remote-provisioning-controller.png)

ちなみにこのツールを作るために htmx を使ったら思ったより使いやすかったので布教するためにこの記事を書きました。

## おわりに

簡単な記事でしたが、htmx が使いやすい文脈もあるんだよ～というお話でした。幅広く使えるというわけではないですが、ここ使い時かもな？と思ったらぜひ使ってみてください。

明日の担当は...と言いたいところですが、明日の担当がいないんですよね、深刻な人手不足。気が向いたら明日もボクが書くかもしれません。期待しないでください。

追記: [ふたばと](https://x.com/01futabato10) さんが3日目の記事として [【WSL2】仮想ハードディスクのサイズを拡張する](https://01futabato10.hateblo.jp/entry/2024/12/03/235955) を出してくれました。
