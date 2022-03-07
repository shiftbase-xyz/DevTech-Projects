
### ✨ Hardhat をインストールする

スマートコントラクトを素早くコンパイルし、ローカル環境にてテストを行うために、**Hardhat** というツールを使用します。

- Hardhat により、ローカル環境でイーサリアムネットワークを簡単に起動し、テストネットでイーサリアムを利用できるようになります。

- 「サーバー」がブロックチェーンであることを除けば、Hardhat はローカルサーバーと同じです。

まず、`node` / `npm` を取得する必要があります。お持ちでない場合は、[こちら](https://hardhat.org/tutorial/setting-up-the-environment.html)にアクセスしてください。

`node v16` をインストールすることを推奨しています。

次に、ターミナルに向かいましょう。

作業をはじめるディレクトリに移動したら、次のコマンドを実行します。

```bash
mkdir epic-nfts
cd epic-nfts
npm init -y
npm install --save-dev hardhat
```

> ✍️: `warning` について
> 最後のコマンドを実行して Hardhat をインストールすると、脆弱性に関するメッセージが表示される場合があります。
>
> 基本的に `warning` は無視して問題ありません。
>
> NPM から何かをインストールするたびに、インストールしているライブラリに脆弱性が報告されているかどうかを確認するためにセキュリティチェックが行われます。
### 👏 サンプルプロジェクトを開始する

次に、Hardhat を実行します。

ターミナルで `epic-nfts` ディレクトリに移動し、下記を実行します：

```bash
npx hardhat
```

⚠️: 注意 #1
> Windowsで Git Bash を使用してハードハットをインストールしている場合、このステップ (HH1) でエラーが発生する可能性があります。問題が発生した場合は、WindowsCMD（コマンドプロンプト）を使用して HardHat のインストールを実行してみてください。

⚠️: 注意 #2
> `npm` と一緒に `yarn` をインストールしている場合、`npm ERR! could not determine executable to run` などのエラーが発生する可能性があります。
> - この場合、`yarn add hardhat` のコマンドを実行しましょう。

⚠️: 注意 #3
> `npx hardhat` が実行されなかった場合、以下をターミナルで実行してください。
>
>```bash
>npm install --save-dev @nomiclabs/hardhat-waffle ethereum-waffle chai @nomiclabs/hardhat-ethers ethers
>```


`hardhat` がターミナル上で立ち上がったら、`Create a basic sample project` を選択します。
- ここでは、すべてに `yes` と言ってください。


次に、安全なスマートコントラクトを開発するために使用されるライブラリ **OpenZeppelin** をインストールします。

ターミナル上で下記を実行してください。

```bash
npm install @openzeppelin/contracts
```

[OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts) は イーサリアムネットワーク上で安全なスマートコントラクを実装するためのフレームワークです。

OpenZeppelin には非常に多くの機能が実装されておりインポートするだけで安全にその機能を使うことができます。
### ⭐️ 実行する

すべてが機能していることを確認するには、以下を実行します。

```
 npx hardhat compile
```
次に、以下を実行します。

```
 npx hardhat test
```
次のように表示されます。

![](/public/images/2-ETH-NFT-collection/section-1/1_2_1.png)

ターミナル上で `epic-nfts` に移動し、`ls` と入力してみて、下記のフォルダーとファイルが表示されていたら成功です。

```
README.md		hardhat.config.js	scripts
artifacts		node_modules		test
cache			package-lock.json
contracts		package.json
```

ここまできたら、フォルダーの中身を整理しましょう。

まず、`test` の下のファイル `sample-test.js` を削除します。

>1\. `test` フォルダーに移動: `cd test`
>
>2\. `sample-test.js` を削除: `rm sample-test.js`

また、`scripts` の下の `sample-script.js` を削除します。

>1\. 一つ上の階層のフォルダー（ `epic-nfts` ）に移動: `cd ..`
>
>2\. `cd scripts` フォルダーに移動: `cd scripts`
>
>3\. `sample-script.js` を削除: `rm sample-script.js`

次に、上記の手順を参考にして `contracts` の下の `Greeter.sol` を削除してください。実際のフォルダは削除しないように注意しましょう。

### ☀️ Hardhat の機能について

Hardhat は段階的に下記を実行しています。

1\. **Hardhat は、スマートコントラクトを Solidity からバイトコードにコンパイルしています。**
- バイトコードとは、コンピュータが読み取れるコードの形式のことです。

2\. **Hardhat は、あなたのコンピューター上でテスト用の「ローカルイーサリアムネットワーク」を起動しています。**

3\. **Hardhat は、コンパイルされたスマートコントラクトをローカルイーサリアムネットワークに「デプロイ」します。**

ターミナルに出力されたアドレスを確認してみましょう。

```bash
Greeter deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3
```

これは、イーサリアムネットワークのテスト環境でデプロイされたスマートコントラクトのアドレスです。
### 🙋‍♂️ 質問する

ここまでの作業で何かわからないことがある場合は、Discord の `#section-1` で質問をしてください。

ヘルプをするときのフローが円滑になるので、エラーレポートには下記の3点を記載してください✨
```
1. 何をしようとしていたか
2. エラー文をコピー&ペースト
3. エラー画面のスクリーンショット
```

---
次のレッスンに進んで、独自のNFTコントラクトの実装を開始しましょう🎉
