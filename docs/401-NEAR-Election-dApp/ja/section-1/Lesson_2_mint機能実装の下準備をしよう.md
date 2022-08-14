### 📝 コントラクトの情報の更新+NFT の mint の下準備を実装しよう

前回まででコントラクトに必要なデータは宣言することはできましたね。

ここからはデータの初期化、NFT の mint の下準備を実装していきます！

まずは lib.rs へ移動して下のように書き換えましょう。
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

+ #[near_bindgen]
+ impl Contract {
+     // function for initialization(new_default_meta)
+     #[init]
+     pub fn new(owner_id: AccountId, metadata: NFTContractMetadata) -> Self {
+         let this = Self {
+             owner_id,
+             tokens_per_owner: LookupMap::new(StorageKey::TokensPerOwner.try_to_vec().unwrap()),
+             tokens_per_kind: LookupMap::new(StorageKey::TokensPerKind.try_to_vec().unwrap()),
+             tokens_by_id: LookupMap::new(StorageKey::TokensById.try_to_vec().unwrap()),
+             token_metadata_by_id: UnorderedMap::new(
+                 StorageKey::TokenMetadataById.try_to_vec().unwrap(),
+             ),
+             metadata: LazyOption::new(
+                 StorageKey::NFTContractMetadata.try_to_vec().unwrap(),
+                 Some(&metadata),
+             ),
+             token_id_counter: 0,
+             likes_per_candidate: LookupMap::new(
+                 StorageKey::LikesPerCandidate.try_to_vec().unwrap(),
+             ),
+             added_voter_list: LookupMap::new(StorageKey::AddedVoterList.try_to_vec().unwrap()),
+             voted_voter_list: LookupMap::new(StorageKey::VotedVoterList.try_to_vec().unwrap()),
+             is_election_closed: false,
+         };
+         this
+     }
+
+     // initialization function
+     #[init]
+     pub fn new_default_meta(owner_id: AccountId) -> Self {
+         Self::new(
+             owner_id,
+             NFTContractMetadata {
+                 spec: "nft-1.0.0".to_string(),
+                 name: "Near Vote Contract".to_string(),
+                 reference: "This contract is design for fair election!".to_string(),
+             },
+         )
+     }
+ }
```

はじめの`#[near_bindgen]`は near のチェーンで有効な構造体、関数を宣言できるようにするためのものです。

次の`impl Contract`についてですが、これは`Contract`という構造体に`{}`内のメソッドを持たせるということを意味しています。

```rust
#[near_bindgen]
impl Contract
```

その次にある`#[init]`は初期化のための関数であることを示しています。lesson-1 で説明した`DefaultOnPanic`というトレイトを覚えているでしょうか？デプロイした後に初期化の関数をまず動かさないと、他の関数を走らせることはできないというものです。

このトレイトは`#[init]`という印が付いている関数を初期化のための関数と認識しています。

ここで宣言している`new関数`では、引数として`AccountId`型の`owner_id`という変数を必要とすることがわかります。`metadata`という変数についても同じことです。

そして`->Self`というのは`Contract`という構造体自身を返すということです。これはコントラクト自体のインスタンスを関数内で生成して、それを返り値として返すということです。

```rust
#[init]
pub fn new(owner_id: AccountId, metadata: NFTContractMetadata) -> Self {
```

中身ではそれぞれの変数の初期化をしています。

例えば tokens_per_owner を例にとると、`LookupMap`という型が持つ`new`というメソッドによって初期化されて新しいインスタンスが生み出されます。

その後`try_to_vec()`によって Result 型のベクター（他の言語では配列）が作られて、`unwrap()`というメソッドによって
`Result<Vec<u8>>->Vec<u8>`に変換されます。くわしくは[Result の説明](https://doc.rust-lang.org/std/result/)と[unwrap の説明](https://ja.stackoverflow.com/questions/1730/rust%E3%81%AEunwrap%E3%81%AF%E4%BD%95%E3%82%92%E3%81%99%E3%82%8B%E3%82%82%E3%81%AE%E3%81%A7%E3%81%99%E3%81%8B)をご覧ください

`UnorderedMap`と`LookupMap`はどちらも Map 形式の型なのですが`UnorderedMap`はそれぞれの map がインデックス化されておりベクター型の map なのですが、`LookupMap`はインデックス化されておらず、map だけが存在しているものです。

最後の this というのはここで宣言した this という変数を返り値とすることを示しています。rust では`return`を明示的に使わなくても、最後に返したい値を`;`なしで記述すれば暗黙的に返り値なんだと解釈してくれます。

```rust
{
    let this = Self {
        owner_id,
        tokens_per_owner: LookupMap::new(StorageKey::TokensPerOwner.try_to_vec().unwrap()),
        tokens_per_kind: LookupMap::new(StorageKey::TokensPerKind.try_to_vec().unwrap()),
        tokens_by_id: LookupMap::new(StorageKey::TokensById.try_to_vec().unwrap()),
        token_metadata_by_id: UnorderedMap::new(
            StorageKey::TokenMetadataById.try_to_vec().unwrap(),
        ),
        metadata: LazyOption::new(
            StorageKey::NFTContractMetadata.try_to_vec().unwrap(),
            Some(&metadata),
        ),
        token_id_counter: 0,
        likes_per_candidate: LookupMap::new(
            StorageKey::LikesPerCandidate.try_to_vec().unwrap(),
        ),
        added_voter_list: LookupMap::new(StorageKey::AddedVoterList.try_to_vec().unwrap()),
        voted_voter_list: LookupMap::new(StorageKey::VotedVoterList.try_to_vec().unwrap()),
        is_election_closed: false,
    };

    this
}
```

次の`new_default_meta関数`はコントラクトのオーナーの Wallet Id を引数として受け取ります。`Self::new`というのは`Self`つまりこのコントラクト自体がもつ`new`という関数を呼び出していることを表しています。

これは`new_default_meta関数`を呼び出すことで`new関数`と`new_default_meta関数`の両方を読んでいることになるので、一度で初期化が完了していることを意味しています。なので new 関数を読んでないけど初期化できているの？と疑問に思う時がきてもこのような理由で完了していることを理解しておいてください。

中身では`spec,name,description`の３つのコントラクトに関する情報と`owner_id`を更新しています。

ここで`owner_id`に何も値が入っていないと疑問に思った方もいるかもしれませんが大丈夫です。rust では引数と同じものであれば省略できるというルールがあるからです。なのでここでは引数として入れられた値がそのまま`owner_id`に入っていることになります。

それ以外の変数は好きなように変えていただいて構いません。

ただ、ここでどの変数にも`to_string()`というメソッドが適用されていることに疑問を抱いた方が多いと思います。その解説については[こちら](https://colorfulcompany.com/rust/str-and-string)をご覧ください。

```rust
#[init]
pub fn new_default_meta(owner_id: AccountId) -> Self {
    Self::new(
        owner_id,
        NFTContractMetadata {
            spec: "nft-1.0.0".to_string(),
            name: "Near Vote Contract".to_string(),
            description: "This contract is design for fair election!".to_string(),
        },
    )
}
```

これでコントラクトの初期化機能の実装は完了しました。次のレッスンでは NFT を mint,transfer 機能を実装しましょう。

### 🎭 NFT を mint, transfer しよう

まずは mint 機能を実装するために `mint.rs` に移動して以下のコードを記述しましょう。

エラーが発生しているとおもいますがそれは他のファイルで定義すべきメソッドを定義していないことが原因なので今は気にしないで大丈夫です。

[mint.rs]

```diff
+ use crate::*;
+ 
+ #[near_bindgen]
+ impl Contract {
+     #[payable]
+ 
+     //mint token
+     pub fn nft_mint(&mut self, mut metadata: TokenMetadata, receiver_id: AccountId) {
+         // set token id
+         assert!(
+             !(&self.is_election_closed),
+             "You can add candidate or voter because this election has been closed!"
+         );
+         metadata.token_id = Some(self.token_id_counter);
+         let initial_storage_usage = env::storage_usage();
+         let receiver_id_clone = receiver_id.clone();
+         let token = TokenOwner {
+             owner_id: receiver_id,
+         };
+         let token_id = self.token_id_counter;
+         let token_kind = metadata.token_kind.clone();
+ 
+         assert!(
+             self.tokens_by_id
+                 .insert(&self.token_id_counter, &token)
+                 .is_none(),
+             "Token already exists"
+         );
+ 
+         // add info(key: receiver_id, value: token metadata ) to map
+         self.token_metadata_by_id
+             .insert(&self.token_id_counter, &metadata);
+ 
+         // add info(key: receiver id, value: token id ) to map
+         self.internal_add_token_to_owner(&token.owner_id, &token_id);
+ 
+         // add info(key: token id, value: token kind ) to map
+         self.internal_add_token_to_kind_map(&token_id, token_kind);
+ 
+         // add data(key: token id, value: number of likes)
+         self.likes_per_candidate
+             .insert(&self.token_id_counter, &(0 as Likes));
+ 
+         // add info(key: receiver id, value: token id ) to map(-> this list is for check voter get vote ticket)
+         self.added_voter_list
+             .insert(&receiver_id_clone, &self.token_id_counter);
+ 
+         // increment token id counter
+         self.token_id_count();
+ 
+         // calculate storage user used
+         let required_storage_in_bytes = env::storage_usage() - initial_storage_usage;
+ 
+         // refund unused payment deposit
+         refund_deposit(required_storage_in_bytes);
+     }
+ 
+     // count token id
+     pub fn token_id_count(&mut self) {
+         self.token_id_counter = self.token_id_counter + 1;
+     }
+ 
+     // get next token id
+     pub fn show_token_id_counter(&self) -> u128 {
+         self.token_id_counter
+     }
+ }
```

次に`nft_mint`という関数について説明します。この関数では引数として NFT の情報、受け取るユーザーの Wallet Id をもらいます。

初めの`#[payable]`は token を授受できるようにするための注釈です。

```rust
#[payable]

//mint token
pub fn nft_mint(&mut self, mut metadata: TokenMetadata, receiver_id: AccountId)
```

関数の中身としてはまずコントラクトの`is_election_closed`という変数が`false`である、つまりまだ投票が終了していないことを確認して、もししまっている(true)のときはもう投票できないというメッセージをコンソールに出力します。

```rust
// set token id
assert!(
    !(&self.is_election_closed),
    "You can add candidate or voter because this election has been closed!"
);
```

次の部分では`metadata`という引数のあるプロパティを更新します。最終的にこの値は NFT のメタデータとしてコントラクトに格納されることになります。この metadata のうちの`token_id`というプロパティを更新します。

入る値としてはコントラクトで独立に定義されている`token_id_counter`という変数の値が入ることになります。`self`はこのコントラクト自体を示すので`self.token_id_counter`と記述されています。この値を`Some()`で囲んでいるのは`token_id`が Option 型だからです。

Option 型というのは値があれば`Some(値)`となり、値が存在しない場合は`None`となるものです。これで値が存在しないストレージにアクセスするというバグがなくなるのです。詳しくは[こちら](https://doc.rust-lang.org/std/option/)をご覧ください。

```rust
metadata.token_id = Some(self.token_id_counter);
```

この次の部分では関数で使う変数を定義しています。
次の`initial_storage_usage`という変数には`storage_usage()`という関数でその時点でのコントラクトが占有しているストレージの領域が格納されます。これは mint によって占有されたストレージの領域を計算するために使われます。

`receiver_id_clone`には引数として受け取った`receiver_id`を格納されています。
`clone()`というメソッドはデータの値をコピーして、全く新しいストレージにそのデータを格納することをしめしています。これは引数として受け取った`receiver_id`の所有権が次に宣言する`token`に移動してしまうため、その前に値をコピーして他のストレージに移動するという意図があって行われています。

`token`という変数はトークンのオーナーを格納しています。

`token_id`にはコントラクトに入れられている token の id が格納されています。

`token_kind`には引数として受け取った`metadata`に格納されている`token_kind`というプロパティが入れられます。

```rust
let initial_storage_usage = env::storage_usage();
let receiver_id_clone = receiver_id.clone();
let token = TokenOwner {
    owner_id: receiver_id,
};
let token_id = self.token_id_counter;
let token_kind = metadata.token_kind.clone();
```

次の`assert!()`というメソッドでは mint するユーザーが所有する NFT の内、これから mint する NFT の token の id を持つものが含まれていないかを確認しています。これは二重に mint することを防いでいます。

```rust
assert!(
    self.tokens_by_id
        .insert(&self.token_id_counter, &token)
        .is_none(),
    "Token already exists"
);
```

次の部分では token の id と NFT の metadata を紐づけるためのマップである`token_metadata_by_id`にデータを格納しています。

```rust
self.token_metadata_by_id.insert(&self.token_id_counter, &metadata);
```

この二つの関数は internal.rs で定義する関数です。

1 つ目の`internal_add_token_to_owner`という関数は token の id とその保有者の Wallet Id を紐づけるための map である`tokens_per_owner`にそれぞれの値を格納する関数です。

次の`internal_add_token_to_kind_map`は token の id とその種類(候補者の NFT か投票券の NFT か)を紐づける map である`tokens_per_kind`にそれぞれの値を格納する関数です。

```rust
self.internal_add_token_to_owner(&token.owner_id, &token_id);
self.internal_add_token_to_kind_map(&token_id, token_kind);
```

ここでは候補者の token の id とその得票数を紐づける map である`likes_per_candidate`にそれぞれの値を格納しています。

```rust
self.likes_per_candidate.insert(&self.token_id_counter, &(0 as Likes));
```

ここでは投票者とその token の id を紐づけるためのベクターにそれぞれの値を格納しています。

```rust
self.added_voter_list.insert(&receiver_id_clone, &self.token_id_counter);
```

これは次の部分で記述する関数を呼び出しており、次に mint される NFT のために token の id をインクリメント(1 大きくすること)しています。

```rust
self.token_id_count();
```

この変数は今のコントラクトが占有しているストレージの領域から mint 前に占有していた領域を引くことで mint によって占有されたストレージの領域を導き出しています。

```rust
let required_storage_in_bytes = env::storage_usage() - initial_storage_usage;
```

この関数は`internal.rs`ファイルで宣言する関数で、ユーザーが mint 時に deposit してくれた NEAR(暗号通貨)に対して多すぎた場合に返金する関数です。

```rust
refund_deposit(required_storage_in_bytes);
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

これで mint の下準備はできました！しかしまだ実装されていない関数があるので次のレッスンで mint 機能 を完成させましょう！
