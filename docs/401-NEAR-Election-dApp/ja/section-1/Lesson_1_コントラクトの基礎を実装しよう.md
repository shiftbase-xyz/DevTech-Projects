### ⛩ コントラクトの基盤を作ろう

**投票なのになぜ NFT？**

今回は投票システムを作るはずですがなぜ `NFT` なのでしょうか？

それは候補者たちの情報、投票券の情報をどちらも NFT として管理しようとしているからです。

具体的にいうと、候補者たちの情報は NFT として`コントラクト内`に保存されることになります。投票権に関しては投票をする人たちが mint して`一時的に彼らのものとなり`、投票を行うとコントラクトに`回収`されることになります。

このような理由からこのセクションでは NFT を作るための基礎の部分をコーディングをしていきます！

**NFT のコアとはなにか**

NFT を作るにあたって開発者に意識してほしい点があります。

それは、NFT はあくまで識別子(unique identifier)である点です。
NFT には、多くの要素(例えば画像)をつけることができ、ついそちらに目がいきがちになってしまうのですが、あくまでそれは、識別子に紐付けられたデータなのです。

ここを意識しながら、開発をすると、理解度がぜんぜん違うと思うので、ぜひそれを意識しながらこれからの section を見ていってください！

**ファイルを作成しよう**

コーディングに入る前に必要なファイルを作成していきましょう！ファイルを作成する方法は二つあります。

① ファイルを作るディレクトリに移動してターミナルで下のコマンドをターミナルで実行する

```bash
touch FILE_NAME
```

② 下のようにワークスペース上でファイルを作るディレクトリを右クリックしてファイル名を記入
![](/public/images/401-NEAR-Election-dApp/section-1/1_1_1.png)

個人的には ② の方が楽なのでこちらの方法でやることが多いです。このどちらかの方法を用いてコントラクトのディレクトリ(ここでは`near-election-dapp-contract`)に以下のようなファイル構造を作成してみましょう。

末尾が`/`となっているものはディレクトリ、そうでないものはファイルであることを示しています。

```diff
  near-election-dapp-contract/
  ├── Cargo.lock
  ├── Cargo.toml
  ├── src/
+ │   ├── vote.rs
+ │   ├── enumeration.rs
+ │   ├── inernal.rs
  │   ├── lib.rs
+ │   ├── metadata.rs
+ │   ├── mint.rs
+ │   └── nft_core.rs
  └── target/
      ├── CACHEDIR.TAG
      └── debug/
```

これでファイルの作成は完了です！

また、ここからはコントラクトの作成をメインに行うのでターミナルで`cd`コマンドを使って `near-election-dapp-contract`へ移動しておきましょう

**NFT に関する情報を格納するためのコードを記述しよう**

NFT を作るためのライブラリはありますが、今回はあえてライブラリは使わずに一から作っていきます。NFT に必要な要素として

1. 情報（画像の URL、名前、説明 etc）を格納できること

2. mint、transfer ができること

が挙げられます。

Lesson1 では 1. の情報の格納を実装していきます！

**Cargo.toml ファイルの編集**

序盤でも説明しましたが、`Cargo.toml` ファイルではこのコントラクトの情報が記述されています。作成者、バージョン、使用するライブラリなどの情報です。

特に最後の使用するライブラリを記述していないとコンパイル時に使用できずエラーが出てしまうので書き漏れのないように下のように書き換えましょう。

[Cargo.toml]

```
//　以下のように書き換えてください
[package]
name = "near-election-dapp-contract"
version = "0.1.0"
authors = ["YOUR_NAME", "YOUR_MAIL_ADDRESS"]
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
near-sdk = "4.0.0"

[lib]
crate-type = ["cdylib", "rlib"]

[profile.release]
codegen-units=1
opt-level = "z"
lto = true
debug = false
panic = "abort"
overflow-checks = true
```

※[dependencies]の near-sdk は 2023/7/23 時点でのバージョンなので、動かないようなら作成時の最新バージョンを入れてみてください

では次は `lib.rs` に移って、元々記述してあるコードを上書きして、使用するライブラリを下のように宣言してみましょう。

[lib.rs]

```rust
//　以下のように書き換えてください
use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};
use near_sdk::collections::{LazyOption, LookupMap, UnorderedMap, UnorderedSet};
use near_sdk::json_types::U128;
use near_sdk::serde::{Deserialize, Serialize};
use near_sdk::{env, near_bindgen, AccountId, Balance, CryptoHash, PanicOnDefault, Promise};

mod vote;
mod enumeration;
mod internal;
mod metadata;
mod mint;
mod nft_core;

pub use crate::enumeration::*;
use crate::internal::*;
pub use crate::metadata::*;
pub use crate::mint::*;
pub use crate::nft_core::*;
pub use vote::*;

```

それぞれのライブラリについては使用する際に細かく説明するのでここでは割愛しますが、大まかに何を記述しているのかを説明します。

最初の部分では Cargo.toml ファイルで追加した near-sdk の中で今回使う部分を宣言しています。

```rust
use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};
use near_sdk::collections::{LazyOption, LookupMap, UnorderedMap, UnorderedSet};
use near_sdk::json_types::U128;
use near_sdk::serde::{Deserialize, Serialize};
use near_sdk::{env, near_bindgen, AccountId, Balance, CryptoHash, PanicOnDefault, Promise};
```

次の部分ではこのプロジェクト内で作成したファイルを取り込んで使えるようにしています。(拡張子である`.rs`は除いて書くのがルールです。)

```rust
mod confirm;
mod enumeration;
mod internal;
mod metadata;
mod mint;
mod nft_core;
```

これは `lib.rs が crate ルート`というものであることから必要なことです。crate ルートとはコンパイルの開始するファイルのことです。

なのでここに使用するライブラリが書かれていなければ使わないと判断されてしまいます。

最後の部分では取り込んだファイルの内容を lib.rs で使えるようにしています。

```rust
pub use crate::enumeration::*;
use crate::internal::*;
pub use crate::metadata::*;
pub use crate::mint::*;
pub use crate::nft_core::*;
pub use vote::*;
```

`crate` というのは crate ルートの `lib.rs` のことであり、`::`というのはファイル構造の記述で使われる`/`と同じ意味で一つ下の階層のものを表すときに使われます。

末尾の`*`はそのファイルないの全てのメソッドが全て使えるよということを示しています。また、最初の`pub`はどこでも使える(public)ということを示しています。

後で説明しますが`internal.rs`は内部でしか使ってほしくないので`pub`キーワードはついていないはずです。

これでライブラリの宣言は完了したので、コントラクトが格納するデータの型を宣言していきましょう。

**格納するデータの型を宣言する**

次は `metadata.rs` へ移動して下のように格納するデータの型を構造体の形で宣言していきましょう

エラーが出ると思いますが、気にせず進みましょう。

[metadata.rs]

```rust
// 以下を追加してください
use crate::*;
pub type TokenId = u128;
pub type CandidateName = String;
pub type TokenKind = String;
pub type HasVoted = bool;
pub type ReceiverId = AccountId;
pub type Likes = f32;

#[derive(BorshDeserialize, BorshSerialize, Serialize, Deserialize, Clone)]
#[serde(crate = "near_sdk::serde")]

// metadata of contract
pub struct NFTContractMetadata {
    pub spec: String,
    pub name: String,
    pub reference: String,
}

```

下の部分ではクレートルートである`lib.rs`の中の全てのものを使うことができることが宣言されています。また、`pub type`で特定の文字列が型を表すようにしています。

```rust
use crate::*;
pub type TokenId = u128;
pub type CandidateName = String;
pub type TokenKind = String;
pub type HasVoted = bool;
pub type ReceiverId = AccountId;
pub type Likes = f32;
```

この部分では`derive`によって型に対して使えるトレイトを増やしています。トレイトとは特定の型に存在する共通の振る舞いのことです。

そのあとに続く`BorshDeserialize, ...`などが提供するトレイトを`NFTContractMetadata`という構造体が使えるようになるということです。

```rust
#[derive(BorshSerialize, BorshDeserialize, Serialize, Deserialize, Clone)]
#[serde(crate = "near_sdk::serde")]
```

それぞれのライブラリの意味は以下のようになっています。

`BorshSerialize`: バイナリ形式でデータ構造をシリアライズすること。シリアライズとはプログラムの実行状態や複雑なデータ構造などを一つの文字列やバイト列で表現することでデータをやりとりしやすい形にすることです。

`BorshDeserialize`:バイナリ形式でシリアライズされたデータを元のデータに戻すこと。

`Serialize`: json 形式でデータ構造をシリアライズすること。バイナリ形式でやることに対して人が読むことができるという利点がある一方でデータの容量をより多くとってしまうという短所がある。

`Deserialize`: json 形式でリアライズされたデータを元のデータに戻すこと。

`Clone`: 元のデータとは違う保管場所を準備して、そこに元のデータをコピーして保管できる。これは rust の所有権の概念と関係してくるので[こちら](https://doc.rust-jp.rs/book-ja/ch04-01-what-is-ownership.html)を参照してください。

`#[serde(crate = "near_sdk::serde")]`:　`Serialize`を使う時に必要なもの

これらの下に記述している`NFTContractMetadata`という構造体はコントラクトそのものの情報を記述しています。スペック、名前、説明文を入れることができるようになっています。

また、String や u128 という型については[こちら](https://zenn.dev/mebiusbox/books/22d4c1ed9b0003/viewer/fb866b)を、`Option`という型については[こちら](https://zenn.dev/mebiusbox/books/22d4c1ed9b0003/viewer/0d7a68)をご覧ください。

```rust
pub struct NFTContractMetadata {
    pub spec: String,
    pub name: String,
    pub reference: String,
}
```

同じようにして NFT の持つべき情報をもった構造体(`TokenMetadata`)やトークンのオーナーとトークンのメタデータを一緒にしている構造体(`JsonToken`)を宣言していきましょう。

[metadata.rs]

```diff
use crate::*;
pub type TokenId = u128;
pub type CandidateName = String;
pub type TokenKind = String;
pub type HasVoted = bool;
pub type ReceiverId = AccountId;
pub type Likes = f32;

#[derive(BorshDeserialize, BorshSerialize, Serialize, Deserialize, Clone)]
#[serde(crate = "near_sdk::serde")]
pub struct NFTContractMetadata {
    pub spec: String,
    pub name: String,
    pub reference: String,
}

+ #[derive(BorshDeserialize, BorshSerialize, Serialize, Deserialize)]
+ #[serde(crate = "near_sdk::serde")]
+ pub struct TokenMetadata {
+     pub title: Option<String>,
+     pub description: Option<String>,
+     pub media: String,
+     pub media_CID: String,
+     pub candidate_name: Option<String>,
+     pub candidate_manifest: Option<String>,
+     pub token_kind: String,
+     pub token_id: Option<u128>,
+ }
+
+ #[derive(BorshDeserialize, BorshSerialize)]
+ pub struct TokenOwner {
+     pub owner_id: AccountId,
+ }
+
+ #[derive(Serialize, Deserialize)]
+ #[serde(crate = "near_sdk::serde")]
+
+ // metadata of type of Json
+ pub struct JsonToken {
+     pub owner_id: AccountId,
+     pub metadata: TokenMetadata,
+ }
```

`TokenMetadata`という構造体について注意しておかなければいけないことを説明します。構造体の中で宣言されている名前についてです。

どのような名前にしても NFT 自体に値として保存されるのですが、`title`, `media`に関しては間違えないようにしてください。 Wallet に画像やその NFT の名前が表示されるのはこれらの名前の変数に格納されている値を参照しているからです。

特に今回作成するシステムでは投票券がユーザーの wallet 上で見ることができる必要があるのでここの部分は十分注意して記述しないと何のエラーもないのに見れないという地獄に陥ってしまいます。これで１日潰す人がいるほどですから 🫣

では最後に先ほど出てきた`トレイト`を自分で実装してみましょう！

[metadata.rs]

```diff
use crate::*;
pub type TokenId = u128;
pub type CandidateName = String;
pub type TokenKind = String;
pub type HasVoted = bool;
pub type ReceiverId = AccountId;
pub type Likes = f32;

#[derive(BorshDeserialize, BorshSerialize, Serialize, Deserialize, Clone)]
#[serde(crate = "near_sdk::serde")]

pub struct NFTContractMetadata {
    pub spec: String,
    pub name: String,
    pub reference: String,
}

#[derive(BorshDeserialize, BorshSerialize, Serialize, Deserialize)]
#[serde(crate = "near_sdk::serde")]

//metadata of Token
pub struct TokenMetadata {
    pub title: Option<String>,
    pub description: Option<String>,
    pub media: String,
    pub media_CID: String,
    pub candidate_name: Option<String>,
    pub candidate_manifest: Option<String>,
    pub token_kind: String,
    pub token_id: Option<u128>,
}

// metadata of Token Owner
#[derive(BorshDeserialize, BorshSerialize)]
pub struct TokenOwner {
    pub owner_id: AccountId,
}

#[derive(Serialize, Deserialize)]
#[serde(crate = "near_sdk::serde")]

// metadata of type of Json
pub struct JsonToken {
    pub owner_id: AccountId,
    pub metadata: TokenMetadata,
}

+ pub trait NFTTokenMetadata {
+     fn nft_metadata(&self) -> NFTContractMetadata;
+ }

+ // view function for contract info
+ #[near_bindgen]
+ impl NFTTokenMetadata for Contract {
+     fn nft_metadata(&self) -> NFTContractMetadata {
+         self.metadata.get().unwrap()
+     }
+ }
```

ここでは`NFTTokenMetadata`という構造体で使うことができる関数を追加しました。内容としてはコントラクトに格納されている NFT のメタデータを見ることができるというものです。

これで metadata.rs は完成です。

次にコントラクトが持つ情報を記述していきましょう。なので次は lib.rs に移動してコントラクトが持つべき情報を下のように記述していきましょう。

[lib.rs]

```diff
use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};
use near_sdk::collections::{LazyOption, LookupMap, UnorderedMap, UnorderedSet};
use near_sdk::json_types::U128;
use near_sdk::serde::{Deserialize, Serialize};
use near_sdk::{env, near_bindgen, AccountId, Balance, CryptoHash, PanicOnDefault, Promise};

mod vote;
mod enumeration;
mod internal;
mod metadata;
mod mint;
mod nft_core;

pub use crate::enumeration::*;
use crate::internal::*;
pub use crate::metadata::*;
pub use crate::mint::*;
pub use crate::nft_core::*;
pub use vote::*;

+ #[near_bindgen]
+ #[derive(BorshDeserialize, BorshSerialize, PanicOnDefault)]
+ pub struct Contract {
+     // contract state value
+     pub owner_id: AccountId,
+     pub tokens_per_owner: LookupMap<AccountId, UnorderedSet<TokenId>>,
+     pub tokens_per_kind: LookupMap<TokenKind, UnorderedSet<TokenId>>,
+     pub tokens_by_id: LookupMap<TokenId, TokenOwner>,
+     pub token_metadata_by_id: UnorderedMap<TokenId, TokenMetadata>,
+     pub metadata: LazyOption<NFTContractMetadata>,
+     pub token_id_counter: u128,
+     pub likes_per_candidate: LookupMap<TokenId, Likes>,
+     pub added_voter_list: LookupMap<ReceiverId, TokenId>,
+     pub voted_voter_list: LookupMap<ReceiverId, u128>,
+     pub is_election_closed: bool,
+ }

+ #[derive(BorshSerialize)]
+ pub enum StorageKey {
+     TokensPerOwner,
+     TokensPerKind,
+     TokensPerOwnerInner { account_id_hash: CryptoHash },
+     TokensPerKindInner { token_kind: TokenKind },
+     TokensById,
+     TokenMetadataById,
+     TokensPerTypeInner { token_type_hash: CryptoHash },
+     NFTContractMetadata,
+     LikesPerCandidate,
+     AddedVoterList,
+     VotedVoterList,
+ }
```

追加したものを上から順番に説明していきましょう。まず`#[near_bindgen]`は near のチェーン上で使えるようにするためのものです。

次の derive の中身にある`PanicOnDefault`は次のレッスンで出てくる初期化の関数がまだ実行されていない時にエラーを出してくれるものです。

次に構造体の内部の変数は以下の意味を持っています。

- `owner_id`: コントラクトのオーナーが誰かが入ることになります。AccountId という型は String 型なのですが、自動的に Wallet Id として正しいかチェックしてくれる便利な型です。

- `tokens_per_owner`: ユーザーの Wallet Id が key となって、所有する NFT のメタデータが格納されます

- `tokens_per_kind`: kind(種類)が key となって、それに分類される（属性をもつ）NFT が格納されます。具体的には投票券と候補者情報を区別するために用意されています。

- `tokens_by_id`: token の id が key となって、それに対応する token のオーナーが格納されます。この変数の目的としては NFT を他人に送る時にそのトークンが存在しているのか、その保持者は送信者と同一ではないかを確認するためです。
- `token_metadata_by_id`: token の id が key となって、それに対応する NFT のメタデータが格納されます。
- `metadata`: NFT のメタデータが配列に格納されています。
- `token_id_counter`: token の Id をカウントするための変数です。mint されるたびに 1 足されていきます。
- `likes_per_candidate`: token の id が key となって、それに対応する得票数が格納されています。
- `added_voter_list`: ユーザーの Wallet Id が key となって、それに対応する token の Id が格納されます。これは投票券がすでに配布されているかを確認するために必要になります。
- `voted_voter_list`: ユーザーの Wallet Id が key となって、0 か 1 の値が返されます。`added_voter_list`と違って token の Id が返ってこないのはフロントでの実装上必要がないからです。ここで説明してもピンとこないかもしれないのでフロントでの実装時に説明します。
- `is_election_closed`: この選挙が終わっているかどうかを bool 型で格納します。この管理はコントラクトを deploy した人のみができるようにする必要があります。

```rust
#[near_bindgen]
#[derive(BorshDeserialize, BorshSerialize, PanicOnDefault)]
pub struct Contract {
    // contract state value
    pub owner_id: AccountId,
    pub tokens_per_owner: LookupMap<AccountId, UnorderedSet<TokenId>>,
    pub tokens_per_kind: LookupMap<TokenKind, UnorderedSet<TokenId>>,
    pub tokens_by_id: LookupMap<TokenId, TokenOwner>,
    pub token_metadata_by_id: UnorderedMap<TokenId, TokenMetadata>,
    pub metadata: LazyOption<NFTContractMetadata>,
    pub token_id_counter: u128,
    pub likes_per_candidate: LookupMap<TokenId, Likes>,
    pub added_voter_list: LookupMap<ReceiverId, TokenId>,
    pub voted_voter_list: LookupMap<ReceiverId, u128>,
    pub is_election_closed: bool,
}
```

その次に記述しているのはそれぞれの変数に対応する enum 型の変数で初期化の際に必要になります。これはそれぞれの変数が格納されるストレージのアドレスの接頭辞をユニークなものにするためのものです。

詳しくは[こちら](https://www.near-sdk.io/contract-structure/nesting)をご覧ください。

```rust
#[derive(BorshSerialize)]
pub enum StorageKey {
    TokensPerOwner,
    TokensPerKind,
    TokensPerOwnerInner { account_id_hash: CryptoHash },
    TokensPerKindInner { token_kind: TokenKind },
    TokensById,
    TokenMetadataById,
    TokensPerTypeInner { token_type_hash: CryptoHash },
    NFTContractMetadata,
    LikesPerCandidate,
    AddedVoterList,
    VotedVoterList,
}
```

### 🙋‍♂️ 質問する

ここまでの作業で何かわからないことがある場合は、Discord の `#section-1` で質問をしてください。

ヘルプをするときのフローが円滑になるので、エラーレポートには下記の 4 点を記載してください ✨

```
1. 質問が関連しているセクション番号とレッスン番号
2. 何をしようとしていたか
3. エラー文をコピー&ペースト
4. エラー画面のスクリーンショット
```

---

長かったですがこれでデータとそれらの型の宣言は完了しました！

次からは実際にこれらのデータに値を入れてコントラクトの情報を更新の準備をしていきましょう！
