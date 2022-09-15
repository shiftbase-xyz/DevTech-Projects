### ✅ 環境構築を行う

このプロジェクトの全体像は次のとおりです。

1 \. **スマートコントラクトを作成します。**

- ユーザーがメッセージをスマートコントラクト上でやり取りのするための処理方法に関するすべてのロジックを実装します。
- 開発は `Ethereum` のローカルチェーンを利用して作成します。

2 \. **スマートコントラクトをブロックチェーン上にデプロイします。**

- スマートコントラクトを `Avalanche` の `C-Chain` (のテストネット)へデプロイします。
- 世界中の誰もがあなたのスマートコントラクトにアクセスできます。
- **ブロックチェーンは,サーバの役割を果たします。**

3 \. **Web アプリケーション（dApp）を構築します**。

- ユーザーは Web サイトを介して,ブロックチェーン上に展開されているあなたのスマートコントラクトと簡単にやりとりできます。
- スマートコントラクトの実装 + フロントエンドユーザー・インタフェースの作成 👉 dApp の完成を目指しましょう 🎉

### ✨ Hardhat をインストールする

スマートコントラクトをすばやくコンパイルし,ローカル環境でテストするために,**Hardhat** というツールを使用します。

- Hardhat により,ローカル環境でイーサリアムネットワークを簡単に起動し,テストネットで ethereum を利用できます。
- 「サーバ」がブロックチェーンであることを除けば,Hardhat はローカルサーバと同じです。

まず,`node` / `npm` を取得する必要があります。  
お持ちでない場合は,[こちら](https://hardhat.org/tutorial/setting-up-the-environment#installing-node.js) にアクセスし`Node.js`をインストールしてください。  
`Node.js`をインストールすると, そのパッケージ管理ツールである`npm`も同時にインストールされます。

> 動作確認。
>
> ```
> $ node -v
> v18.6.0
> ```
>
> ```
> $ npm -v
> 8.13.2
> ```
>
> 併せて本プロジェクトの実行環境(上記のバージョン)もトラブル時の参考にしてください。

続いて, `typescript`を使用するのでターミナルで以下を実行しましょう。  
`$`で始まる行はターミナル上で実行していることを表していて, `$`を除いたものが実行コマンドです。

```
$ npm install -g typescript
```

> 動作確認。
>
> ```
> $ tsc -v
> Version 4.7.4
> ```
>
> `tsc`は`typescript`のコマンドです。

### 🛫 プロジェクトを作成しよう

作業したいディレクトリに移動したら,次のコマンドを実行します。

```bash
$ mkdir Avalanche-dApp
$ cd Avalanche-dApp
$ mkdir messenger-contract
$ cd messenger-contract
$ npm init -y
$ npm install --save-dev hardhat @openzeppelin/test-helpers
$ npm install dotenv
```

dapp 全体のディレクトリ(`Avalanche-dApp`)とコントラクト実装に使用するディレクトリ(`messenger-contract`)を用意しました。  
次に`npm init`により npm パッケージを管理するための環境をセットアップを行いました。  
最後にスマートコントラクトの開発に必要な以下のパッケージを`npm`コマンドを利用してインストールしています。

- `hardhat`
- `dotenv`: 環境変数の設定で必要になります。コントラクトをデプロイする際に利用します。
- `@openzeppelin/test-helpers`: テストを支援するライブラリです。コントラクトのテストを書く際に利用します。

> `--save-dev`とは  
> アプリの開発時のみ必要なパッケージ(テストツールなど)のインストールに使用するオプション指定です。  
> アプリ自体が動くのに必要なパッケージのインストールには`--save`または何も指定せずに`npm install`を使用します。  
> [こちら](https://stackoverflow.com/questions/22891211/what-is-the-difference-between-save-and-save-dev)を参考にしてください。

### 👏 サンプルプロジェクトを開始する

次に,Hardhat を実行します。

ターミナルで `messenger-contract` に移動し,下記を実行します。  
`npx hardhat`を実行すると対話形式で指示を求められるので下記のように回答します。  
`Create a TypeScript project`を選択するところ以外は`enter`を押せば例通りになるはずです。

```
$ npx hardhat
...
✔ What do you want to do? · Create a TypeScript project
✔ Hardhat project root: · path/to/messenger-contract /contract
✔ Do you want to add a .gitignore? (Y/n) · y
✔ Do you want to install this sample project's dependencies with npm (@nomicfoundation/hardhat-toolbox)? (Y/n) · y
```

### ⭐️ 実行する

すべてが機能していることを確認するには,以下を実行します。

```
$ npx hardhat compile
```

次に,以下を実行します。

```
$ npx hardhat test
```

次のように表示されたら成功です！ 🎉

![](/public/images/AVAX-msg/section-1/1_1_1.png)

ここまできたら,フォルダーの中身を整理しましょう。

`messenger-contract`内は以下のようなフォルダ構成になっているはずです。

```
messenger-contract
├── README.md
├── artifacts
├── cache
├── contracts
├── hardhat.config.ts
├── node_modules
├── package-lock.json
├── package.json
├── scripts
├── test
├── tsconfig.json
└── typechain-types
```

まず,`test` の下のファイル `Lock.ts` を削除します。

1. `test` フォルダーに移動: `cd test`
2. `Lock.ts` を削除: `rm Lock.ts`

次に,上記の手順を参考にして `contracts` の下の `Lock.sol` を削除してください。実際のフォルダは削除しないように注意しましょう。

### 🙋‍♂️ 質問する

ここまでの作業で何かわからないことがある場合は,Discord の `#messenger-dapp` で質問をしてください。

ヘルプをするときのフローが円滑になるので,エラーレポートには下記の 3 点を記載してください ✨

```
1. 質問が関連しているセクション番号とレッスン番号
2. 何をしようとしていたか
3. エラー文をコピー&ペースト
4. エラー画面のスクリーンショット
```

---

環境設定が完了したら,次のレッスンに進んでください 🎉
