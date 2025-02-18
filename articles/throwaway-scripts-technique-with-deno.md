---
title: "既存プロジェクトでDenoのスクリプトを使うテクニック"
emoji: "🦕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - deno
published: true
published_at: 2025-02-19 12:00
publication_name: "trap"
---

## はじめに

プロダクトのコードとは別でスクリプト的なコードを書きたい場面があったとき、Deno を使うことで細かい設定[^1]なしに TypeScript や豊富なライブラリの恩恵を受けたスクリプトを書くことができます。しかしナイーブに組み込んだだけでは既存のプロジェクトに影響を及ぼしてしまいがちです。この記事ではその解決策を提示します。

[^1]: `tsconfig.json` や `tsc` のアレコレなど

## 結論

`.vscode/settings.json`

```json
{
  "deno.enable": true,
  "deno.enablePaths": ["./path-to-deno-scripts-dir"]
}
```

その上で次のコマンドのように実行する (`--no-lock` オプションを付ける)

```
$ deno run --no-lock script.ts
```

## ナイーブな方法の問題点

Deno を既存のプロジェクト内で使おうとすると、いくつか困るポイントが出てくると思います。出てこないとしたらこの記事を読む必要はないと思います。困るポイントはおそらく「Deno: Enable をする必要がある」「`deno.lock` が生成されてしまう」の2つだと思います。これらの問題について具体的な例を用いて見ていきましょう。

例えば次のようなディレクトリ構成を考えます。

```
/
├── src/
│   ├── components/
│   ├── pages/
│   ├── App.tsx
│   └── index.tsx
└── package.json
```

この環境に対して Deno で書いた次のようなコードを `scripts/walk.ts` として加えることにします。

```ts
import * as fs from "jsr:@std/fs";

for await (const file of fs.walk(".")) {
  console.log(file);
}
```

まず VSCode の設定でワークスペースを Deno のワークスペースとして認識させるために「Deno: Enable」する必要があることに気付きます。そうしないと `jsr:@std/fs` という特殊な形式のモジュールを認識してくれません。

しかし「Deno: Enable」するとワークスペース全体が Deno のコードと認識してしまい、既存のコードの import などが正しく解決されなくなってしまうことがあります。これが1つ目の問題です。

さらにこの状態で `deno run scripts/walk.ts` すると `deno.lock` が生成されます。書き捨てのスクリプト等であればわざわざ `deno.lock` をリポジトリに含める必要はないでしょうから、生成しないでもらえるなら生成しないで貰いたいところです。これが2つ目の問題です。[^2]

[^2]: 継続的に使っていくスクリプトなど、`deno.lock` をリポジトリ内に含めたい場合は問題になりませんが、問題になる場合もあるだろうということで取り上げました。

## 解決策

1つ目の問題に関しては次のような設定で一部のディレクトリ以下のみ Deno の機能を有効化することで解決します。

`.vscode/settings.json`

```json
{
  "deno.enable": true,
  "deno.enablePaths": ["./path-to-deno-scripts-dir"]
}
```

2つ目の問題に関しては次のように `--no-lock` オプションを付けた上で実行すると解決します。

```
$ deno run --no-lock script.ts
```

通常 `deno run` しただけでは `deno.lock` は生成されないのですが、`package.json` がある環境で実行すると自動で `deno.lock` が生成されてしまうようです。

## おわりに

この記事で触れている問題が原因で Deno を敬遠している方がいれば、ぜひ試してみてください。ちなみに解決策は [@ras0q](https://x.com/ras0q) さんと [@SSlime](https://x.com/_SSlime) さんに教えて頂きました。ありがとうございました！

![「Deno、依存周りをシングルファイルで書けて書き捨てのスクリプトに良いよねとか思っていたが、VSCode で deno: enable が必要だったり deno.lock が生成されちゃったりでもにょっとする」という cp20 による投稿](/images/traq-deno-post.png)
*これを呟いただけで解決策が降ってくる環境、最高過ぎない？*
