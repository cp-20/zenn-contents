---
title: "Webフロントエンドエンジニアってどんな人？何をしてるの？年収は？彼女はいるの？調べてみました！"
emoji: "💭"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["ポエム"]
published: true
published_at: 2023-12-20 20:30
---

:::message
これを書いてる人は職業エンジニアではなく、ただのしがない学生エンジニアです。なのでもしかしたら実際のお仕事とは違う部分があるかもしれませんが、そこはそういうものなんだな、とご理解いただければと思います。
:::

この記事は [CA Tech Lounge Advent Calendar 2023](https://qiita.com/advent-calendar/2023/catechlounge) 20日目の記事です。

## Webフロントエンドエンジニアってどんな人？

Webフロントエンドと言われるような領域を専門にするエンジニアのことをWebフロントエンドエンジニアとよく呼びます。しばしば単にフロントエンドエンジニアとも呼ばれます。^[一般にフロントエンドと言うだけでは表側に見えてくる部分、つまりPC・スマホのネイティブアプリなども含まれますが、それらはネイティブアプリと呼ばれることが多いので、フロントエンドと言えばWebフロントエンドを指すことが多いです](この記事の中でフロントエンドと言えばWebフロントエンドを指すことにします)

![](/images/web-frontend-engineer-image.png)
*Webフロントエンドエンジニアのイメージ画像 by GPT-4*

## Webフロントエンドエンジニアって何をしてるの？

WebフロントエンドエンジニアというぐらいですからWebのフロントエンド、つまりHTMLとかCSSとかJSとかをガチャガチャして画面を組み立てているわけです。ここ最近 (直近10年ぐらい) で生まれてきて、ホントに最近激アツになってきた分野 (ボク調べ) です。

一昔前は、データ処理は主にサーバー側で行い、その結果をHTMLテンプレートに流し込んで動的なサイトを構築するのが一般的でしたが、モダンWebフロントと呼ばれるような技術スタックでは、サーバーはデータ (主にJSON) を返すだけで、そのデータをもとに画面を組み立てる、という方式が増えてきているように感じます。

その理由は単純で、その方がユーザー体験が良くなるからです。サーバー側でHTMLを生成するとなると何かアクションを起こすたび、データが変わるたびにページ全体の読み込み直しが発生します。一方フロントエンドはデータだけを受け取ることによって、追加のデータを受け取ることで柔軟に画面を構築することができ、画面の一部分のみを更新するといったことが容易にできるようになります。

### 他の分野との違いは？

WebフロントエンドとWebデザインがよく混同されがちだと思っています。個人開発、あるいは小規模チームなどではフロントエンドエンジニアがデザイナーを兼任することもあると思いますが、しっかりとした大きなチームではデザイナーが画面を作ることがよくあります。となるとフロントエンドエンジニアがするべきことはなんでしょう？Figmaとかで画面を作ればHTML/CSS形式で出力してくれるのでこれでもうフロントエンドエンジニアはいりませんね、完。

とまぁ事はそう単純じゃないわけです。コーポレートサイトなどではデザインをそのまま書き起こすだけ^[デザインをHTML/CSSに書き起こす人材を「コーダー」と呼ぶこともあるらしいです (やることに対して言葉の意味広過ぎね...？)]でほぼほぼ完成みたいなところはありますが、いわゆるWebアプリと呼ばれるようなものだとロジックを実装していく必要があります。

ロジックはバックエンドで書くんじゃないの？と思ったそこのあなた、間違ってはないです。ただ本質的にバックエンドでやらなければいけない処理というのは実は多くなくて、実質的にバックエンドでやらなきゃいけないこと、バックエンドでやった方が良いことなどが大半なことが多い印象です。誤解を生む発言をすると、バックエンドはDBの呼び出しをちょっと抽象化しているだけなので、権限周りさえどうにかすればバックエンドは必要ないんですね。実際FirebaseのRealtime DatabaseとかCloud Firestoreとかはセキュリティルールで権限周りを担保することでバックエンドをサービス化するということをやっていますね。(それが必ずしも良いとは言わないけれども)

### やってて大変なポイント・楽しいポイントは？

よく勘違いされがち(?)なポイントとして、画面を作るの大変そうだね、というご意見を頂くことがあります。聞くところによると、CSSという言語は詳しくない人 (特にフルスタックでないバックエンドの人とか) から見るとよくわからないし書きたくない言語らしいんですよね。そんな言語を書いて頑張って画面を作っているなぁ、大変だなぁと思われるかもしれないんですが、実は画面を作るのは楽しいです。(ボク調べ)

CSS、実は和解すればかなり良いやつで、表現力は相当高いし、基礎だけ抑えればあとはパターン学習でかなり書けるようになります。すると画面を書くのはひたすらHTML (JSX) とCSSをバリバリと書いていくだけなので実はそこまで苦ではない、というかむしろ画面が出来上がっていく様を見るのはかなり楽しいです。^[相当凝ったUIでなければデザインを見ればすすすっと画面は書けます。例えば今書いてるこの `zenn-cli` のプレビューとかも、画面デザインを起こすだけなら1,2時間あれば書けそうな気がしています (書かないけど)]

逆に何がつらいのかというと、難しいロジックや設計、キャッチアップですね。設計についてはどの分野でも存在して、難しいと言われるところだと思うので、ことさら強調する必要はないでしょう。難しいロジックについても他の分野でもよくあることに挙げられると思いますが、Webフロント特有の難しいロジックと言うのも存在しています。

例えばデータ取得周りで言うと、Optimistic Update (楽観的更新) というものがあります。例えばTwitterでいいねをするとき、いいねというアクションをサーバーに送信していいねされたのを確認してから画面に反映する、というのだとどうしても遅延が発生します。

![](/images/favorite-tweet-request.png)
*いいねをするアクションのリクエストが203msで返ってきている図*

でも実際Twitterでいいねをするときってクリック/タップしてから200msも待たずにアニメーションが再生されてハートに色がつきますよね？逆にいいねを取り消すときも遅延なく消えますよね？つまりサーバーの返信を待つ前に画面を書き換えている、つまり画面の状態を書き換えているわけです。これをOptimistic Updateと言います。これの難しさは、単に画面を更新するだけならそうすればいいのですが、サーバーからデータが返ってきたとき、そしてそのデータが現状フロントで仮に更新したデータと違ったときにどうするかということです。食い違っていたら普通はサーバー側の状態に合わせる必要がありますが、それをうまくコードで書くのは難しいし、複雑性もグンと上がります。ただOptimistic UpdateはUXにかなり大きく響くので、難しいところです、、、、

他の難しい例として、アニメーションが挙げられます。これはモダンWebフロントになって難しくなったところだと思います。モダンWebフロントは UI = f(state) の考え方に代表されるように、state (やサーバーから取得したデータ) のみによってUIは定まり、その過程には関知しない (宣言的UI) という方向で設計されていることが多いです。つまり状態の間を繋ぐようなアニメーションは宣言的UIには表れて来づらいです。

特にあるあるの例として、画面から削除するときのアニメーション (フェードアウトなど) を実装する時に、愚直に実装するとアニメーションが終わる前にDOMから消えてアニメーションが再生されないという事件が起こります。これを回避するためにアニメーションが終わるまではDOMに存在させておく、ということが必要になり、これもまたコードの複雑性を増させる原因になります。細かいところのUXを上げようとすると、それに伴ってコードの複雑性と実装の難易度も上がるので本当に難しいところです。

さらにキャッチアップが大変というのもフロントエンドあるあるだと思います。Webフロントは技術革新の流れが相当早く、数年前は覇権を争っていた技術が今では下火になってしまったという例がいくつもあります。新しいライブラリやフレームワークもたくさん出てきますし、その流れの中でトレンドを掴んでいくというのは大変なところもあります。ただボク的には素晴らしい技術が次々と生まれてDXとUXを上げてくれるならそれは歓迎すべきことだと思っているので、キャッチアップはそれほど苦ではないです。

Twitterをチラチラ眺めてみんなが発言してたらトレンドなんだな～程度の認識で、面白そうなやつがあれば見てみる、触ってみるということをやっています。ただあんまり触るところまではやれてなくて、もっと頑張らないとなぁ、、という気持ちです。

## Webフロントエンドエンジニアの年収は？彼女はいる？

ボク自身は就職していないので何とも言えないんですが、調べてみたところ平均年収は550万円ぐらいらしいです。実際どうなんでしょうか、人によりそうですね。

彼女がいるかどうかも人によりけりな気がしますが、ボクはいないらしいです。この件に関しての統計情報は見つかりませんでした。

## まとめ

いかがでしたか？この記事では

- Webフロントエンドエンジニアはどういうことをしているのか
- どういうところが楽しいのか
- どういうことが楽しいのか

などについて見ていきました。知らないことも多かったのではないでしょうか？

明日は [@fki813](https://qiita.com/fki813) さんと [@showsay](https://qiita.com/showsay) さんの記事です！お楽しみに！

---

## 余談 (記事を書いた背景)

ボクの所属してる[traP](https://trap.jp)というサークルでB1、B2にバックエンド人間が多くてフロントエンド人間が少ないという状況がありました。またこのアドベントカレンダーの主催元(?)である [CA Tech Lounge](https://www.cyberagent.co.jp/careers/special/students/tech_lounge/) というコミュニティではフロントエンドの会員が現時点でボクしかいません。事あるごとに布教を進めていて、だんだん興味を持ってくれている人が増えてきているのですが、でもフロントエンドって結局何をやるんだ？という疑問に対しての問いにボク自身が完璧に把握できてないなと思って、言語化ついでに記事化しようと思った次第です。

フロントエンドエンジニアに興味を持ってくれたそこのあなた🫵、[CA Tech Lounge](https://www.cyberagent.co.jp/careers/special/students/tech_lounge/) のフロントエンドエンジニア会員になってボクと握手🤝