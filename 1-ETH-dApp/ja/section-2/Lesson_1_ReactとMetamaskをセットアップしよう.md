### 🍽 フロントエンドのファイルをあなたの Github にフォークする。

このセクションでは、WEBサイトの構築を通して、フロントエンドがスマートコントラクトとどのように関連するのか学びます。

まだ Github のアカウントをお持ちでない方は、[こちら](https://qiita.com/okumurakengo/items/848f7177765cf25fcde0) の手順に沿ってアカウントを作成してください。

Github のアカウントをお持ちの方は、[こちら](https://github.com/shiftbase-xyz/dApp-starter-project) から、フロントエンドの基盤をあなたの Github にフォークしましょう。

フォークの方法は、[こちら](https://docs.github.com/ja/get-started/quickstart/fork-a-repo) を参照してください。

ご自身の Github アカウントにフォークした `dApp-starter-project` レポジトリをあなたのローカル環境にクローンしましょう。

下図のように、`Code` ボタンをクリックした後、`SSH` を選択し、git リンクをコピーしましょう。

![](/public/images/1-ETH-dApp/section-2/2_1_1.png)

ターミナルで任意のディレクトリに移動し、先ほどコピーしたリンクを貼り付け、下記を実行してください。

```bash
git clone コピーした_github_リンク
```

ターミナル上で `dApp-starter-project` に移動して下記を実行しましょう。

```bash
npm install
```

`npm` コマンドを実行することで、JavaScript ライブラリのインストールが行われます。

次に、下記を実行してみましょう。

```bash
npm run start
```

あなたのローカル環境で、WEBサイトのフロントエンドが立ち上がりましたか？

例）ローカル環境で表示されているWEBサイト

![](/public/images/1-ETH-dApp/section-2/2_1_2.png)

上記のような形でフロントエンドが確認できれば成功です。

これからフロントエンドの表示を確認する際は、`dApp-starter-project` ディレクトリ上で、`npm run start` を実行します。

ターミナルを閉じるときは、以下のコマンドが使えます✍️
- Mac: `ctrl + c`
- Windows: `ctrl + shift + w`
### 🦊 MetaMask をダウンロードする

次に、イーサリアムウォレットをダウンロードしましょう。

このプロジェクトでは MetaMask を使用します。
- [こちら](https://metamask.io/download.html) からブラウザの拡張機能をダウンロードし、MetaMask ウォレットをあなたのブラウザに設定します。

すでに別のウォレットをお持ちの場合でも、今回は MetaMask を使用してください。

> ✍️: MetaMask が必要な理由
> ユーザーが、スマートコントラクトを呼び出すとき、本人のイーサリアムアドレスと秘密鍵を備えたウォレットが必要となります。
> これは、認証作業のようなものです。
### 🙋‍♂️ 質問する

ここまでの作業で何かわからないことがある場合は、Discord の `#section-2` で質問をしてください。

ヘルプをするときのフローが円滑になるので、エラーレポートには下記の3点を記載してください✨

```
1. 何をしようとしていたか
2. エラー文をコピー&ペースト
3. エラー画面のスクリーンショット
```
---

MetaMask のダウンロードが完了したら次のレッスンに進みましょう🎉
