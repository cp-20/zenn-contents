---
title: "Vercel の Middleware 課金対策"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vercel", "nextjs"]
published: true
published_at: 2025-08-31 12:00
---

## 記事を書くきっかけ

個人開発のプロジェクトで急に Fluid Active CPU の無料枠を突破してしまいました。75%のアラートが鳴ったときに気付けば良かったんですが「ユーザー増えてるせいかな？」と無視してしまい、事態の発覚が遅れました。

https://x.com/\_\_cp20\_\_/status/1961809606386360491

流石にまずいと思って調査した結果、急にユーザーが増えたわけではなくVercel側の仕様変更によるもの (要出典) だとわかりました。思ったより事故りやすい気がしていて、パッと調べても出てこなかったので原因と対応策を書き残しておきます。少しでも参考になれば嬉しいです！

## 原因

タイトルの出オチ感ありますが、Middleware が原因でした。Usage を見てみると8/6ぐらいから急に利用が発生していているのが分かりました。SSR などのサーバーで実行される機能はないはずなのに何が使っているんだろうと思ってType別に見てみると、Middleware の利用率が100%であることが分かりました。

![Vercel Fluid compute Usage の推移、8/6ごろから急に発生している](/images/vercel-fluid-compute-usage.png)

軽く調べてみたのですが、↓の記事を見る感じは2024/09時点では少なくとも Middleware のCPU時間に課金はされないという仕様だったみたいですが、引用している Vercel のドキュメント ([Manage and optimize usage for Edge Middleware](https://vercel.com/docs/pricing/edge-middleware)) はリンク切れを起こしてしまっています。(おい！)

https://zenn.dev/chot/articles/3e5e5dacd2ff48

現在の仕様を確認したところ、Middleware の実行は Fluid compute として利用料金が加算されるようでした。

> Routing Middleware is priced using the [fluid compute](https://vercel.com/docs/fluid-compute) model, which means you are charged by the amount of compute resources used by your Routing Middleware. See the [fluid compute pricing documentation](https://vercel.com/docs/fluid-compute/pricing) for more information.
> 
> [Routing Middleware > Pricing](https://vercel.com/docs/routing-middleware#pricing)

これまでの情報を合算すると、おそらく8/6頃に Middleware も課金対象に入るように仕様変更が起こったのではないかと思っています。が、公式のアナウンスメントなどを見つけることができていません。もし何か知っている人がいたら教えてください🙏

記事を書いている今 (2025/08/31) 現在で Vercel に問い合わせ中ですが、まだ返答は返ってきていません。

## 対策

Middleware で何をやるかは人次第なので一概には言えないですが、一般的な対策を書いておきます。

### Matcher を適切に使う

Middleware は雑に使うと全リクエスト (静的アセットなども含む) をインターセプトしてしまい、結果的にリクエスト数やCPU時間を消費してしまいます。なのでそもそもリクエストをインターセプトさせないことが非常に重要になります。

これを実現する仕組みが Matcher で、特定の条件を満たしたリクエストのみに Middleware を噛ませることができるようになります。例えば↓のように特定のパスのみに制限することができます。

```ts
export const config = {
  matcher: '/about/:path*',
};
```

詳しくはドキュメントを読んでください。

https://vercel.com/docs/routing-middleware/api#config-object

### そもそも Middleware を使わない

ボクはもともと i18n 用のパスの rewrite を Middleware でやっていたのですが、実は Middleware を利用しなくても直接 rewrite する方法があります。

https://vercel.com/docs/rewrites

redirect も同様にできます。

https://vercel.com/docs/redirects

また (静的な) http header の付与もできます。

https://vercel.com/docs/headers

こっちの方が無駄なJSランタイムを噛ませない分たぶん速いので (要検証) この機能では不可能なほど高度な rewrite / redirect / header 付与 でなければこちらを使うのが良いかと思います。

## おわりに

ボクのアプリでは最終的に Middleware を全部消すことができたので、たぶん Fluid compute 問題は解決するはずです。(頼む🙏)

Middleware によってコストが爆発しそう (している) 人を救えたら嬉しいです。ありがとうございました！
