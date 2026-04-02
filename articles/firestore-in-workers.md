---
title: "Cloudflare Workers 上で 'firebase/firestore' を import すると EvalError になる"
emoji: '😇'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ["firebase", "firestore", "workers"]
published: true
---

## 現象

このような hook を使うようなコンポーネントを書いて、OpenNext を使って Next.js on Cloudflare Workers で動かすと、ページにアクセスするたびに 500 Internal Server Error が返ってきます。(SSG ではなく SSR した場合)

```ts
import { getFirestore } from 'firebase/firestore';

const useFirebase = () => {
  const firestore = getFirestore();
  return { firestore };
};
```

## 直接的な原因

エラーの内容を見てみると、`eval()` や `Function()` などの文字列をそのままコードとして実行するようなコードが Workers 上で実行できないことが原因でした。

```
EvalError: Code generation from strings disallowed for this context
```

## 解決策

`@firebase/firestore` パッケージに対して次のようなパッチを当てると直ります。`patch-package` などのパッケージを使うか、`bun patch` などのコマンドを使うと良いでしょう。

```diff
--- a/package.json
+++ b/package.json
@@ -66,10 +66,6 @@
   "exports": {
     ".": {
       "types": "./dist/index.d.ts",
-      "node": {
-        "require": "./dist/index.node.cjs.js",
-        "import": "./dist/index.node.mjs"
-      },
       "react-native": "./dist/index.rn.js",
       "browser": {
         "require": "./dist/index.cjs.js",
```

これはどの場合でもブラウザ用のパッケージを使わせる設定ですが、テストなどの Node.js 環境でも `firebase/firestore` を使っている場合は、その場合だけ Node.js 用のパッケージを使わせるなどの工夫が必要になります。

このような monkey patch を当てたくない場合は、`firebase/firestore` パッケージを import するコンポーネントを全て `dynamic()` を使ってクライアントでしか実行されないようにするか、うまく `@firebase/firestore` パッケージをクライアントでしかロードしないようにするなどの工夫が必要になります。

## 根本原因の解説

`firebase/firestore` を import すると、このようなコールグラフを辿って `Function` コンストラクタが実行され、EvalError を引き起こします。

- 自分のコード (のサーバー側) から `firebase/firestore` を import する
- `firebase/firestore` は内部で `@grpc/proto-loader` を import する
  - https://npmx.dev/package-code/firebase/v/12.11.0/firestore%2Fdist%2Findex.cjs.js#L5
- `@grpc/proto-loader` は内部で `protobufjs/ext/descriptor` を import する
  - https://npmx.dev/package-code/@grpc/proto-loader/v/0.7.15/build%2Fsrc%2Findex.js#L23
- `protobufjs/ext/descriptor` は `$protobuf.Root.fromJSON()` を呼び出す
  - https://npmx.dev/package-code/protobufjs/v/7.5.4/ext/descriptor/index.js#L3
- `Root.fromJSON()` は `NameSpace.resolveAll()` を呼び出す
  - https://npmx.dev/package-code/protobufjs/v/7.5.4/src/root.js#L65
  - (直接的に import されているのは `index.js` だが `src/index.js`、`src/index-light.js` のように辿っていけて、`src/index-light.js` で `protobuf.Root` が設定されている)
- `NameSpace.resolveAll()` は再帰的に呼び出されながら、その中で `Type.resolveAll()` を呼び出す
  - https://npmx.dev/package-code/protobufjs/v/7.5.4/src%2Fnamespace.js#L361
- `Type.resolveAll()` は各フィールドの `resolve()` を呼び出す
  - https://npmx.dev/package-code/protobufjs/v/7.5.4/src%2Ftype.js#L314
- `Field.resolve()` が `this.parent.ctor` (`Type.ctor`) を触ることで getter が発火
  - https://npmx.dev/package-code/protobufjs/v/7.5.4/src%2Ffield.js#L356
- `Type.ctor` の getter が `Type.generateConstructor()` を呼び出す
  - https://npmx.dev/package-code/protobufjs/v/7.5.4/src%2Ftype.js#L155
- `Type.generateConstructor()` が `util.codegen()` を呼び出す
  - https://npmx.dev/package-code/protobufjs/v/7.5.4/src%2Ftype.js#L199
- `util.codegen` が `Function` コンストラクタを呼び出して EvalError を起こす
  - https://npmx.dev/package-code/@protobufjs/codegen/v/2.0.4/index.js#L52

ここで `@firebase/firestore` パッケージは Node.js 用とブラウザ用の exports をそれぞれ持っていて、バンドラーはそれらを適切に選択してバンドルを行います。Next.js のバンドラーはクライアント向けにはブラウザ用、SSRなどのサーバー向けには Node.js 用のパッケージを選択してバンドルします。

問題になっている `@grpc/proto-loader` パッケージは Node.js 用のパッケージにのみ含まれているので、サーバー側でもブラウザ用のパッケージを import するようにすれば解決します。一部の機能が正しく動かない可能性があるかもしれませんが、import しただけで落ちるという問題は回避できます。

サーバー側でも強制的にブラウザ用のパッケージを使わせるようにするのが前述した[解決策](#解決策)です。
