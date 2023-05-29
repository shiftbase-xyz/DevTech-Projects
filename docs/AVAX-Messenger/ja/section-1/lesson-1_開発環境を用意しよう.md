### ✅ 環境構築を行う

このプロジェクトの全体像は次のとおりです。

1 \. **スマートコントラクトを作成します。**

- ユーザーがメッセージをスマートコントラクト上でやり取りのするための処理方法に関するすべてのロジックを実装します。
- 開発は`Ethereum`のローカルチェーンを利用して作成します。

2 \. **スマートコントラクトをブロックチェーン上にデプロイします。**

- スマートコントラクトを`Avalanche`の`C-Chain`（のテストネット）へデプロイします。
- 世界中の誰もがあなたのスマートコントラクトにアクセスできます。
- **ブロックチェーンは,サーバーの役割を果たします。**

3 \. **Web アプリケーション（dApp）を構築します**。

- ユーザーはWebサイトを介して,ブロックチェーン上に展開されているあなたのスマートコントラクトと簡単にやりとりできます。
- スマートコントラクトの実装 + フロントエンドユーザー・インタフェースの作成 👉 dAppの完成を目指しましょう 🎉

まず、`node` / `yarn`を取得する必要があります。お持ちでない場合は、[こちら](https://hardhat.org/tutorial/setting-up-the-environment.html)にアクセスしてください。

`node v16`をインストールすることを推奨しています。

それでは本プロジェクトで使用するフォルダーを作成してきましょう。作業を始めるディレクトリに移動したら、次のコマンドを実行します。

```bash
mkdir AVAX-Messenger
cd AVAX-Messenger
yarn init --private -y
```

AVAX-Messengerディレクトリ内に、package.jsonファイルが生成されます。

```bash
AVAX-Messenger
 └── package.json
```

それでは、`package.json`ファイルを以下のように更新してください。

```json
{
  "name": "AVAX-Messenger",
  "version": "1.0.0",
  "description": "Creating NFT Collections",
  "private": true,
  "workspaces": {
    "packages": [
      "packages/*"
    ]
  },
  "scripts": {
    "contract": "yarn workspace contract",
    "client": "yarn workspace client",
    "test": "yarn workspace contract test"
  }
}
```

`package.json`ファイルの内容を確認してみましょう。

モノレポを作成するにあたり、パッケージマネージャーの機能である[Workspaces](https://classic.yarnpkg.com/lang/en/docs/workspaces/)を利用しています。

この機能により、yarn installを一度だけ実行すれば、すべてのパッケージ（今回はコントラクトのパッケージとクライアントのパッケージ）を一度にインストールできるようになります。

**workspaces**の定義をしている部分は以下になります。

```json
"workspaces": {
  "packages": [
    "packages/*"
  ]
},
```

また、ワークスペース内の各パッケージにアクセスするためのコマンドを以下の部分で定義しています。

```json
"scripts": {
  "contract": "yarn workspace contract",
  "client": "yarn workspace client",
  "test": "yarn workspace contract test"
}
```

これにより、各パッケージのディレクトリへ階層を移動しなくてもプロジェクトのルート直下から以下のようにコマンドを実行することが可能となります（ただし、各パッケージ内に`package.json`ファイルが存在し、その中にコマンドが定義されていないと実行できません。そのため、現在は実行してもエラーとなります。ファイルは後ほど作成します）。

```bash
yarn <パッケージ名> <実行したいコマンド>
```

それでは、ワークスペースのパッケージを格納するディレクトリを作成しましょう。

以下のようなフォルダー構成となるように、`packages`ディレクトリとその中に`contract`ディレクトリを作成してください（`client`ディレクトリは、後ほどのレッスンでスターターコードをクローンする際に作成したいと思います）。

```diff
AVAX-Messenger
 ├── package.json
+└── packages/
+    └── contract/
```

`contract`ディレクトリには、スマートコントラクトを構築するためのファイルを作成していきます。

最後に、AVAX-Messengerディレクトリ下に`.gitignore`ファイルを作成して以下の内容を書き込みます。

```bash
**/yarn-error.log*

# dependencies
**/node_modules

# misc
**/.DS_Store
```

最終的に以下のようなフォルダー構成となっていることを確認してください。

```bash
AVAX-Messenger
 ├── .gitignore
 ├── package.json
 └── packages/
     └── contract/
```

これでモノレポの雛形が完成しました！

### ✨ Hardhat をインストールする

スマートコントラクトをすばやくコンパイルし、ローカル環境にてテストを行うために、**Hardhat** というツールを使用します。

- Hardhatにより、ローカル環境でイーサリアムネットワークを簡単に起動し、テストネットでイーサリアムを利用できます。

- 「サーバー」がブロックチェーンであることを除けば、Hardhatはローカルサーバーと同じです。

それでは、先ほど作成した`packages/contract`ディレクトリ内にファイルを作成します。ターミナルに向かい、packages/contract`ディレクトリ内で以下のコマンドを実行します。

```bash
cd packages/contract
yarn init --private -y
```

次に作成されたpackage.jsonを下のように編集してください。

```
```diff
{
  "name": "contract",
  "version": "1.0.0",
  "private": true,
  "devDependencies": {
    "@nomicfoundation/hardhat-chai-matchers": "^1.0.6",
    "@nomicfoundation/hardhat-network-helpers": "^1.0.8",
    "@nomicfoundation/hardhat-toolbox": "^2.0.2",
    "@nomiclabs/hardhat-ethers": "^2.2.2",
    "@nomiclabs/hardhat-etherscan": "^3.1.7",
    "@typechain/ethers-v5": "^10.2.0",
    "@typechain/hardhat": "^6.1.5",
    "chai": "^4.3.7",
    "ethers": "^6.1.0",
    "hardhat": "^2.13.0",
    "hardhat-gas-reporter": "^1.0.9",
    "solidity-coverage": "^0.8.2",
    "typechain": "^8.1.1"
},
```
```

> ✍️: `warning`について
> Hardhat をインストールすると、脆弱性に関するメッセージが表示される場合があります。
>
> 基本的に`warning`は無視して問題ありません。
>
> YARN から何かをインストールするたびに、インストールしているライブラリに脆弱性が報告されているかどうかを確認するためにセキュリティチェックが行われます。

### 👏 サンプルプロジェクトを開始する

次に、Hardhatを実行します。

`packages/contract`ディレクトリにいることを確認し、次のコマンドを実行します。

```bash
npx hardhat
```

`hardhat`がターミナル上で立ち上がったら、それぞれの質問を以下のように答えていきます。

```
・What do you want to do? →「Create a JavaScript project」を選択
・Hardhat project root: →「'Enter'を押す」 (自動で現在いるディレクトリが設定されます。)
・Do you want to add a .gitignore? (Y/n) → 「y」
```

（例）
```bash
$ npx hardhat

888    888                      888 888               888
888    888                      888 888               888
888    888                      888 888               888
8888888888  8888b.  888d888 .d88888 88888b.   8888b.  888888
888    888     "88b 888P"  d88" 888 888 "88b     "88b 888
888    888 .d888888 888    888  888 888  888 .d888888 888
888    888 888  888 888    Y88b 888 888  888 888  888 Y88b.
888    888 "Y888888 888     "Y88888 888  888 "Y888888  "Y888

👷 Welcome to Hardhat v2.13.0 👷‍

✔ What do you want to do? · Create a JavaScript project
✔ Hardhat project root: · /AVAX-Messenger/packages/contract
✔ Do you want to add a .gitignore? (Y/n) · y

✨ Project created ✨

See the README.md file for some example tasks you can run

Give Hardhat a star on Github if you're enjoying it! 💞✨

     https://github.com/NomicFoundation/hardhat
```

> ⚠️: 注意 #1
>
> Windows で Git Bash を使用してハードハットをインストールしている場合、このステップ (HH1) でエラーが発生する可能性があります。問題が発生した場合は、WindowsCMD（コマンドプロンプト）を使用して HardHat のインストールを実行してみてください。

> ⚠️: 注意 #2
>
> `npx hardhat`が実行されなかった場合、以下をターミナルで実行してください。
>
> ```bash
> yarn add --dev @nomicfoundation/hardhat-toolbox
> ```

この段階で、フォルダー構造は下記のようになっていることを確認してください。

```diff
AVAX-Messenger
 ├── .gitignore
 ├── package.json
 └── packages/
     ├── client/
     └── contract/
+        ├── .gitignore
+        ├── README.md
+        ├── contracts/
+        ├── hardhat.config.js
+        ├── package.json
+        ├── scripts/
+        └── test/
```

それでは、`contract`ディレクトリ内に生成された`package.json`ファイルを以下を参考に更新をしましょう。

```diff
{
  "name": "contract",
  "version": "1.0.0",
-  "main": "index.js",
-  "license": "MIT",
  "private": true,
  "devDependencies": {
    "@nomicfoundation/hardhat-chai-matchers": "^1.0.6",
    "@nomicfoundation/hardhat-network-helpers": "^1.0.8",
    "@nomicfoundation/hardhat-toolbox": "^2.0.2",
    "@nomiclabs/hardhat-ethers": "^2.2.2",
    "@nomiclabs/hardhat-etherscan": "^3.1.7",
    "@typechain/ethers-v5": "^10.2.0",
    "@typechain/hardhat": "^6.1.5",
    "chai": "^4.3.7",
    "ethers": "^6.1.0",
    "hardhat": "^2.13.0",
    "hardhat-gas-reporter": "^1.0.9",
    "solidity-coverage": "^0.8.2",
    "typechain": "^8.1.1"
  },
+  "scripts": {
+    "test": "npx hardhat test"
+  }
}
```

不要な定義を削除し、hardhatの自動テストを実行するためのコマンドを追加しました。

次に、安全なスマートコントラクトを開発するために使用されるライブラリ **OpenZeppelin** をインストールします。

`packages/contract`ディレクトリにいることを確認し、以下のコマンドを実行してください。

```bash
yarn add --dev @openzeppelin/contracts
```

[OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts) はイーサリアムネットワーク上で安全なスマートコントラクトを実装するためのフレームワークです。

OpenZeppelinには非常に多くの機能が実装されておりインポートするだけで安全にその機能を使うことができます。

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

![](/public/images/AVAX-Messenger/section-1/1_2_2.png)

ターミナル上で`ls`と入力してみて、下記のフォルダーとファイルが表示されていたら成功です。

```bash
README.md         cache             hardhat.config.js package.json      test
artifacts         contracts         node_modules      scripts
```

ここまできたら、フォルダーの中身を整理しましょう。

まず、`test`の下のファイル`Lock.js`を削除します。

1. `test`フォルダーに移動: `cd test`

2. `Lock.js`を削除: `rm Lock.js`

次に、上記の手順を参考にして`contracts`の下の`Lock.sol`を削除してください。実際のフォルダは削除しないように注意しましょう。


### ☀️ Hardhat の機能について

Hardhatは段階的に下記を実行しています。

1\. **Hardhat は、スマートコントラクトを Solidity からバイトコードにコンパイルしています。**

- バイトコードとは、コンピュータが読み取れるコードの形式のことです。

2\. **Hardhat は、あなたのコンピュータ上でテスト用の「ローカルイーサリアムネットワーク」を起動しています。**

3\. **Hardhat は、コンパイルされたスマートコントラクトをローカルイーサリアムネットワークに「デプロイ」します。**

ターミナルに出力されたアドレスを確認してみましょう。

```bash
Greeter deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3
```

これは、イーサリアムネットワークのテスト環境でデプロイされたスマートコントラクトのアドレスです。

### 🐊 `github`にソースコードをアップロードしよう

本プロジェクトの最後では, アプリをデプロイするために`github`へソースコードをアップロードする必要があります。

**Avax-Messenger**全体を対象としてアップロードしましょう。

今後の開発にも役に立つと思いますので, 今のうちに以下にアップロード方法をおさらいしておきます。

`GitHub`のアカウントをお持ちでない方は,[こちら](https://qiita.com/okumurakengo/items/848f7177765cf25fcde0) の手順に沿ってアカウントを作成してください。

`GitHub`へソースコードをアップロードをしたことがない方は以下を参考にしてください。

[新しいレポジトリを作成](https://docs.github.com/ja/get-started/quickstart/create-a-repo)（リポジトリ名などはご自由に）した後,  
手順に従いターミナルからアップロードを済ませます。  
以下ターミナルで実行するコマンドの参考です。(`Avax-Messenger`直下で実行することを想定しております)

```
$ git init
$ git add .
$ git commit -m "first commit"
$ git branch -M main
$ git remote add origin [作成したレポジトリの SSH URL]
$ git push -u origin main
```

> ✍️: SSH の設定を行う
>
> Github のレポジトリをクローン・プッシュする際に,SSHKey を作成し,GitHub に公開鍵を登録する必要があります。
>
> SSH（Secure SHell）はネットワークを経由してマシンを遠隔操作する仕組みのことで,通信が暗号化されているのが特徴的です。
>
> 主にクライアント（ローカル）からサーバー（リモート）に接続をするときに使われます。この SSH の暗号化について,仕組みを見ていく上で重要になるのが秘密鍵と公開鍵です。
>
> まずはクライアントのマシンで秘密鍵と公開鍵を作り,公開鍵をサーバーに渡します。そしてサーバー側で「この公開鍵はこのユーザー」というように,紐付けを行っていきます。
>
> 自分で管理して必ず見せてはいけない秘密鍵と,サーバーに渡して見せても良い公開鍵の 2 つが SSH の通信では重要になってきます。
> Github における SSH の設定は,[こちら](https://docs.github.com/ja/authentication/connecting-to-github-with-ssh) を参照してください!

### 🙋‍♂️ 質問する

ここまでの作業で何かわからないことがある場合は,Discordの`#avalanche`で質問をしてください。

ヘルプをするときのフローが円滑になるので,エラーレポートには下記の3点を記載してください ✨

```
1. 質問が関連しているセクション番号とレッスン番号
2. 何をしようとしていたか
3. エラー文をコピー&ペースト
4. エラー画面のスクリーンショット
```

---

環境設定が完了したら,次のレッスンに進んでください 🎉