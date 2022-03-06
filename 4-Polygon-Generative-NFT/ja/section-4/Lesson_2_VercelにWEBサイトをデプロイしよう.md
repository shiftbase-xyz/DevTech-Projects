### 🤵 UX のアップグレード

これで、ユーザーが NFT を Mint できる、WEB3フロントエンドが完成しました。

しかし、お気づきのように、WEBサイトのUXには多くの不都合があります。

以下のポイントを考慮して、UX/UI をアップグレードしましょう。
### ✅ ユーザーが正しいネットワークに接続されていることを確認する

あなたのWEBサイトでは、ユーザーは Polygon Testnet に接続すれば、NFT を Mint することができます。

OpenSea のように、ユーザーが間違ったネットワークに接続している場合、警告を出す機能を実装してみてはどうでしょうか？

![](/public/images/4-Polygon-Generative-NFT/section-4/4_2_1.png)

また、ユーザーが間違ったネットワークに接続しているときに、`Mint NFT` ボタンを見えなくする機能も有効な手段でしょう。
### 💫 取引状況の表示

現在、WEBサイトでは、取引状況を `Console` に出力しています。

実際のプロジェクトでは、ユーザーがWEBサイトを操作している間に `Console` を開くことはほぼありません。

トランザクションの状態を追跡し、リアルタイムでユーザーにフィードバックするような状態を実装できないでしょうか。

取引処理中はローダーを表示し、取引に失敗した場合はユーザーに通知し、取引に成功した場合は取引ハッシュを表示する必要があります。
### 👉 資金が存在しない場合でも Metamask を促す

Metamask のウォレットに ETH がない場合、`Mint NFT` をクリックしても Metamask は反応しません。

よって、ユーザーには何のフィードバックもありません。

ユーザーが資金不足の場合でも Metamask がプロンプトされるようにできませんか？

ETH がいくら必要で、いくら足りないかをユーザーに知らせるのは Metamask であることが理想的です。
### 🛀 その他の Quality Of Life のアップグレード

その他に検討可能な QOL の変更をいくつか紹介します。

- ユーザーが一度に 1 つ以上の NFT を Mint できるようにする。

- Openseaにあなたのコレクションへのリンクを追加する。

- コントラクトで何が起こっているのかをユーザーが確認できるように、検証済みのスマートコントラクトのアドレスを追加する。

- あなたの Twitter、Discord へのリンクを追加する。

アップグレードされた UX はこのようになります。

![](/public/images/4-Polygon-Generative-NFT/section-4/4_2_2.png)

[こちら](https://nft-collectible-demoo.vercel.app/) のWEBサイトは、UX のアップグレードの大部分を実装しています。

- 実際に NFT を Mint して、あなたのWEBウェブサイトとの違いを検証してみてください。

- あなたの Github アカウントにこのレポジトリをフォークしましょう。

- クローンしたレポジトリをあなたのローカル環境にダウンロードしてください。

- ターミナルを開き、ディレクトリのルートで `npm install` を実行します。

- `npm run start` を実行してプロジェクトを開始します。

- `localhost:3000` でWEBサイトをホストしながら、コードの中身を調べてみてください。

UI をアップデートする際に参考になるコードが、`App.js` / `App.cs` / `Header.js` / `Footer.js` に隠されています✨

![](/public/images/4-Polygon-Generative-NFT/section-4/4_2_3.png)

ぜひ、あなたのWEBアプリをアップグレードして、HTML/CSS/Javascript への理解を深めましょう。

web3フロントエンドのマスターに一歩近づくことができます。

ターミナルを閉じるときは、以下のコマンドが使えます✍️
- Mac: `ctrl + c`
- Windows: `ctrl + shift + w`
### 🤟 Vercel に WEBサイトをデプロイする

最後に、[Vercel](https://vercel.com/) にWEBサイトをホストする方法を学びます。

Vercel はサーバレス機能のホスティングを提供するクラウドプラットフォームです。

スケーリングやサーバーの監視は Vercel が行うため、開発者は Vercel へデプロイするだけでアプリケーションを公開・運用できます。

Vercel に関しする詳しい説明は、[こちら](https://zenn.dev/lollipop_onl/articles/eoz-vercel-pricing-2020)をご覧ください。

まず、Github に向かい、新しいリポジトリを作成していきましょう。

`Your repositories` ページを開き、`New` ボタンを押してください。

![](/public/images/4-Polygon-Generative-NFT/section-4/4_2_4.png)

リポジトリに、`nft-collectible-frontend-git` と名前をつけたら、`Create repository` ボタンを押してください。

次に、ディレクトリのリンクをコピーしましょう。

![](/public/images/4-Polygon-Generative-NFT/section-4/4_2_6.png)

ターミナルに向かい、任意のディレクトリに移動し、コピーしたリンクを下記に貼り付け絵、実行しましょう。

```
git clone コピーしたリンクを貼り付け
```

次に、`nft-collectible-frontend` を開き、中に入っているフォルダとファイルを全て `nft-collectible-frontend-git` に移動させましょう。

`nft-collectible-frontend` は排除してください。

それでは、Github の `nft-collectible-frontend-git` にローカルファイルをアップロードしていきます。

ターミナル上で `nft-collectible-frontend-git` に移動して、下記を実行しましょう。

```
git add .
git commit -m "upload to github"
git push
```

次に、[Vercel](https://vercel.com/) のアカウントを取得して、下記を実行していきます。

1\. `Dashboard` へ進んで、`New Project` を選択してください。

![](/public/images/4-Polygon-Generative-NFT/section-4/4_2_7.png)

2\. `Import Git Repository` で自分のGithubアカウントを接続したら、`nft-collectible-frontend-git` を選択し、`Import` してください。

![](/public/images/4-Polygon-Generative-NFT/section-4/4_2_8.png)

3\. プロジェクトを作成します。Environment Variable に下記を追加します。

`NAME`＝`CI`、`VALUE`＝`false`（下図参照）

![](/public/images/4-Polygon-Generative-NFT/section-4/4_2_9.png)

4\. `Deploy`ボタンを推しましょう。

VercelはGithubと連動しているので、Githubが更新されるたびに自動でデプロイを行ってくれます。

下記のように、`Building` ログが出力されます。

基本的に `warning` は無視して問題ありません。

![](/public/images/4-Polygon-Generative-NFT/section-4/4_2_10.png)

デプロイが完了したら、自分のWEBサイトに向かい、NFT を Mint してみましょう。

![](/public/images/4-Polygon-Generative-NFT/section-4/4_2_11.png)
### 🙋‍♂️ 質問する

ここまでの作業で何かわからないことがある場合は、Discordの `#section-3` で質問をしてください。

ヘルプをするときのフローが円滑になるので、エラーレポートには下記の3点を記載してください✨
```
1. 何をしようとしていたか
2. エラー文をコピー&ペースト
3. エラー画面のスクリーンショット
```
### 🎉 おつかれさまでした！

あなたのオリジナルの Generative Art NFT Collection を販売できるWEBサイトが完成しました。

あなたが習得したスキルは、分散型WEBアプリがより一般的になる社会の中で、世界を変える重要なスキルです。

これからもweb3への旅をあなたが続けてくれることを願っています🚀
### 🎫 NFTを取得しよう！

NFT を取得する条件は、以下のようになります。

1. MVP の機能がすべて実装されている（実装 OK）

2. WEBアプリで MVP の機能が問題なく実行される（テスト OK）

3. このページの最後にリンクされている Project Completion Form に記入する

4. Discord の `🔥｜post-your-project` チャンネルに、あなたのWEBサイトをシェアしてください😉🎉 Discord に投稿する際に、追加実装した機能とその概要も教えていただけると幸いです！

プロジェクトを完成させていただいた方には、NFT をお送りします。※ 現在準備中なので、今しばらくお待ちください！
### 🎉 おつかれさまでした！

あなたのオリジナルの NFT Collection が完成しました。

あなたは、コントラクトをデプロイし、NFT が Mint できるWEBサイトを立ち上げました。

これらは、分散型WEBサイトがより一般的になる社会の中で、世界を変える2つの重要なスキルです。

これからもweb3への旅をあなたが続けてくれることを願っています🚀
