### 🍵 mint 機能を完成させよう

では mint 機能を完成させてさせましょう。

internal.rs に移動して下のコードを追加しましょう

[internal.rs]

```diff
+ use crate::*;
+ use near_sdk::CryptoHash;
+ 
+ pub(crate) fn hash_account_id(account_id: &AccountId) -> CryptoHash {
+     let mut hash = CryptoHash::default();
+     hash.copy_from_slice(&env::sha256(account_id.as_bytes()));
+     hash
+ }
+ 
+ pub(crate) fn refund_deposit(storage_used: u64) {
+     let required_cost = env::storage_byte_cost() * Balance::from(storage_used);
+     let attached_deposit = env::attached_deposit();
+ 
+     assert!(
+         required_cost <= attached_deposit,
+         "Must attach {} yoctoNear to cover storage",
+         required_cost,
+     );
+ 
+     let refund = attached_deposit - required_cost;
+ 
+     if refund > 1 {
+         Promise::new(env::predecessor_account_id()).transfer(refund);
+     }
+ }

+ impl Contract {
+     pub(crate) fn internal_add_token_to_owner(
+         &mut self,
+         account_id: &AccountId,
+         token_id: &TokenId,
+     ) {
+         let mut tokens_set = self.tokens_per_owner.get(account_id).unwrap_or_else(|| {
+             UnorderedSet::new(
+                 StorageKey::TokensPerOwnerInner {
+                     account_id_hash: hash_account_id(&account_id),
+                 }
+                 .try_to_vec()
+                 .unwrap(),
+             )
+         });
+         tokens_set.insert(token_id);
+         self.tokens_per_owner.insert(account_id, &tokens_set);
+     }
+ 
+     pub(crate) fn internal_add_token_to_kind_map(
+         &mut self,
+         token_id: &TokenId,
+         token_kind: TokenKind,
+     ) {
+         let token_kind_clone = token_kind.clone();
+         let mut tokens_set = self
+             .tokens_per_kind
+             .get(&token_kind_clone)
+             .unwrap_or_else(|| {
+                 UnorderedSet::new(
+                     StorageKey::TokensPerKindInner {
+                         token_kind: token_kind,
+                     }
+                     .try_to_vec()
+                     .unwrap(),
+                 )
+             });
+         tokens_set.insert(&token_id);
+         self.tokens_per_kind.insert(&token_kind_clone, &tokens_set);
+     }
+ }

```

では順番に説明していきます。

`CryptoHash`は 32bytes のハッシュ値の型のことです。こちらでアカウントをハッシュ化します。

```rust
use near_sdk::CryptoHash;
```

この関数では Wallet Id をハッシュ化して返してくれます。

ここで使われている`pub(crate)`とは、このファイル内だけで使用できる関数であることを示しています。

```rust
pub(crate) fn hash_account_id(account_id: &AccountId) -> CryptoHash {
    let mut hash = CryptoHash::default();

    hash.copy_from_slice(&env::sha256(account_id.as_bytes()));
    hash
}
```

この関数では引数として受け取った`storage_used`をガス代に換算して、`required_cost`という変数に代入します。

次に`attached_deposit`にユーザーから deposit された NEAR を取得して、`assert`関数で deposit された NEAR の方が大きいことを確認します。

最後に`refund`という変数に返金するべき NEAR の量を代入してユーザーに transfer して返金します。ここで使われている Promise とは非同期処理を行うもので、その中の`predecessor_account_id`とはユーザーのアカウントのことを表しています。

```rust
pub(crate) fn refund_deposit(storage_used: u64) {
    let required_cost = env::storage_byte_cost() * Balance::from(storage_used);
    let attached_deposit = env::attached_deposit();

    assert!(
        required_cost <= attached_deposit,
        "Must attach {} yoctoNear to cover storage",
        required_cost,
    );

    let refund = attached_deposit - required_cost;

    if refund > 1 {
        Promise::new(env::predecessor_account_id()).transfer(refund);
    }
}
```

次のこの関数ではまず`tokens_set`という変数に、引数である`account_id`に対する token の値が紐づいている`tokens_per_owner`という map を入れます。

`tokens_set`の前についている`mut`は`mutable`のことで変更可能であることをしてしています。

`unwrap_or_else`というメソッドはもし引数である`account_id`に対する token の値(ベクター型)が存在していなければ新しくベクターを作るというものである。

`UnorderedSet`の`{}`では引数である`account_id`をハッシュ化した値によってユニークなストレージの接頭辞を作り出す目的がある。

その後変数`tokens_set`に引数である`token_id`を追加する

最後に`tokens_per_owner`に引数である`account_id`に紐付いた`token_set`の map を追加する。

```rust
pub(crate) fn internal_add_token_to_owner(
    &mut self,
    account_id: &AccountId,
    token_id: &TokenId,
) {
    let mut tokens_set = self.tokens_per_owner.get(account_id).unwrap_or_else(|| {
        UnorderedSet::new(
            StorageKey::TokensPerOwnerInner {
                account_id_hash: hash_account_id(&account_id),
            }
            .try_to_vec()
            .unwrap(),
        )
    });
    tokens_set.insert(token_id);
    self.tokens_per_owner.insert(account_id, &tokens_set);
}
```

この関数で行われていることは`internal_add_token_to_owner`関数で行われていることと同じで、token の id とその token の種類が紐づいているということだけが違うだけなので説明は割愛します。

```rust
pub(crate) fn internal_add_token_to_kind_map(
    &mut self,
    token_id: &TokenId,
    token_kind: TokenKind,
) {
    let token_kind_clone = token_kind.clone();
    let mut tokens_set = self
        .tokens_per_kind
        .get(&token_kind_clone)
        .unwrap_or_else(|| {
            UnorderedSet::new(
                StorageKey::TokensPerKindInner {
                    token_kind: token_kind,
                }
                .try_to_vec()
                .unwrap(),
            )
        });

    tokens_set.insert(&token_id);
    self.tokens_per_kind.insert(&token_kind_clone, &tokens_set);
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

これでやっと mint の準備は完了しました！

しかし今の状態では mint した NFT の情報を wallet 上で見ることができません。なぜなら、near の wallet では決められた名前の関数(`nft_tokens_for_owner関数`)をコントラクトから呼ぶことで NFT の metadata の一つである`media`の値から画像を wallet に表示するからです。

なので次は mint した NFT の情報をみることができるような実装をしましょう！
