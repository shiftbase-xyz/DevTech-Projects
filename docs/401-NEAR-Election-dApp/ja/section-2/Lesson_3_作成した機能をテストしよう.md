### ✅ 投票機能をテストしてみよう

前回までのセクションで投票機能を実装してコントラクトとしては完成したので実際に機能しているかターミナル上で確認してみましょう！

テストの方法は 2 つあり

1. テスト用の関数を走らせて、思った通りの挙動をするか一気にテストする
2. 実際に deploy してターミナル上から関数を動かして確認する

です。

まずは手軽にできる 1 つ目の方法でテストして、それが成功したら 2 つ目のテストをしましょう。

一つ目のテストは他の人がすぐにコードのテストができるという意味でも重要なのでテストを入れることを癖づけていきましょう！

`lib.rs`に移動して以下のようにコードを追加してください。

注意点として、追加するテストは`impl Contract`の外に書いてください！

[lib.rs]

```diff
use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};
use near_sdk::collections::{LazyOption, LookupMap, UnorderedMap, UnorderedSet};
use near_sdk::json_types::U128;
use near_sdk::serde::{Deserialize, Serialize};
use near_sdk::{env, near_bindgen, AccountId, Balance, CryptoHash, PanicOnDefault, Promise};

mod enumeration;
mod internal;
mod metadata;
mod mint;
mod nft_core;
mod vote;

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

#[near_bindgen]
impl Contract {
    // function for initialization(new_default_meta)
    #[init]
    pub fn new(owner_id: AccountId, metadata: NFTContractMetadata) -> Self {
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

    // initialization function
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
}

+ #[cfg(all(test, not(target_arch = "wasm32")))]
+ mod tests {
+     use near_sdk::test_utils::{accounts, VMContextBuilder};
+     use near_sdk::testing_env;
+     use std::collections::HashMap;
+
+     use super::*;
+
+     const MINT_STORAGE_COST: u128 = 100000000000000000000000;
+
+     fn get_context(predecessor_account_id: AccountId) -> VMContextBuilder {
+         let mut builder = VMContextBuilder::new();
+         builder
+             .current_account_id(accounts(0))
+             .signer_account_id(predecessor_account_id.clone())
+             .predecessor_account_id(predecessor_account_id);
+         builder
+     }
+
+     #[test]
+     fn mint_test() {
+         let mut context = get_context(accounts(1));
+         testing_env!(context.build());
+         let mut contract = Contract::new_default_meta(accounts(1).into());
+         testing_env!(context
+             .storage_usage(env::storage_usage())
+             .attached_deposit(MINT_STORAGE_COST)
+             .predecessor_account_id(accounts(1))
+             .build());
+
+         assert_eq!(contract.owner_id, accounts(1));
+
+         contract.nft_mint(
+             TokenMetadata {
+                 title: None,
+                 description: None,
+                 media: "https...".to_string(),
+                 media_CID: "Qeo...".to_string(),
+                 candidate_name: None,
+                 candidate_manifest: None,
+                 token_kind: "candidate".to_string(),
+                 token_id: None,
+             },
+             accounts(1),
+         );
+
+         assert_eq!(u128::from(contract.nft_total_supply()), 1);
+
+         let nft_info = contract.nft_tokens(None, None);
+         assert_eq!(nft_info[0].metadata.media, "https...".to_string());
+         assert_eq!(u128::from(contract.nft_supply_for_owner(accounts(1))), 1);
+         assert_eq!(
+             nft_info[0].owner_id,
+             contract.nft_tokens_for_owner(accounts(1), None, None)[0].owner_id
+         );
+         assert_eq!(
+             nft_info[0].owner_id,
+             contract.nft_tokens_for_kind("candidate".to_string(), None, None)[0].owner_id
+         );
+    }
+
+     #[test]
+     fn vote_closed_test() {
+         let mut context = get_context(accounts(1));
+         testing_env!(context.build());
+         let mut contract = Contract::new_default_meta(accounts(1).into());
+         testing_env!(context
+             .storage_usage(env::storage_usage())
+             .attached_deposit(MINT_STORAGE_COST)
+             .predecessor_account_id(accounts(1))
+             .build());
+         assert_eq!(contract.is_election_closed, false);
+
+         contract.close_election();
+         assert_eq!(contract.is_election_closed, true);
+
+         contract.reopen_election();
+         assert_eq!(contract.is_election_closed, false);
+     }
+
+     #[test]
+     fn transfer_test() {
+         let mut context = get_context(accounts(1));
+         testing_env!(context.build());
+         let mut contract = Contract::new_default_meta(accounts(1).into());
+         testing_env!(context
+             .storage_usage(env::storage_usage())
+             .attached_deposit(MINT_STORAGE_COST)
+             .predecessor_account_id(accounts(1))
+             .build());
+
+         contract.nft_mint(
+             TokenMetadata {
+                 title: None,
+                 description: None,
+                 media: "https...".to_string(),
+                 media_CID: "Qeo...".to_string(),
+                 candidate_name: None,
+                 candidate_manifest: None,
+                 token_kind: "candidate".to_string(),
+                 token_id: None,
+             },
+             accounts(1),
+         );
+
+         testing_env!(context
+             .storage_usage(env::storage_usage())
+             .attached_deposit(1)
+             .predecessor_account_id(accounts(1))
+             .build());
+
+         contract.nft_transfer(accounts(2), 0);
+
+         let nft_info = contract.nft_tokens(None, None);
+         assert_eq!(nft_info[0].owner_id, accounts(2));
+     }
+ }

```

３つあるテストの関数を 1 つずつみていきましょう。

まず最初に宣言している`get_context関数`というのはテストをしているのではなく、テストをするための仮想的なチェーン（Virtual Machine）をビルドするためのものです。

```rust
fn get_context(predecessor_account_id: AccountId) -> VMContextBuilder {
        let mut builder = VMContextBuilder::new();
        builder
            .current_account_id(accounts(0))
            .signer_account_id(predecessor_account_id.clone())
            .predecessor_account_id(predecessor_account_id);
        builder
    }
```

次にかかれている`mint_test関数`は文字通り`mint`がうまくできているかをテストしています。

最初の部分で初期化のための関数である`new_default_meta関数`を呼び、その後 mint のための関数である`nft_mint関数`を読んだ後に、NFT の情報を見るために前回までで作成した関数を呼び思ったような挙動をしているのかを確認します。

`assert_eq!()`というメソッドはひとつ目の引数と二つ目の引数が一致していなければ error を出すようになものです。

NFT の全ての情報を確認するのは面倒なので NFT の中の情報の一部がきちんと一致しているかをテストしています。

```rust
#[test]
    fn mint_test() {
        let mut context = get_context(accounts(1));
        testing_env!(context.build());
        let mut contract = Contract::new_default_meta(accounts(1).into());
        testing_env!(context
            .storage_usage(env::storage_usage())
            .attached_deposit(MINT_STORAGE_COST)
            .predecessor_account_id(accounts(1))
            .build());

        assert_eq!(contract.owner_id, accounts(1));

        contract.nft_mint(
            TokenMetadata {
                title: None,
                description: None,
                media: "https...".to_string(),
                media_CID: "Qeo...".to_string(),
                candidate_name: None,
                candidate_manifest: None,
                token_kind: "candidate".to_string(),
                token_id: None,
            },
            accounts(1),
        );

        assert_eq!(u128::from(contract.nft_total_supply()), 1);

        let nft_info = contract.nft_tokens(None, None);
        assert_eq!(nft_info[0].metadata.media, "https...".to_string());
        assert_eq!(u128::from(contract.nft_supply_for_owner(accounts(1))), 1);
        assert_eq!(
            nft_info[0].owner_id,
            contract.nft_tokens_for_owner(accounts(1), None, None)[0].owner_id
        );
        assert_eq!(
            nft_info[0].owner_id,
            contract.nft_tokens_for_kind("candidate".to_string(), None, None)[0].owner_id
        );
    }
```

この部分によって NEAR を deposit しています。これはストレージを確保するためのもので、`mint, transfer`のために必要になります。

```rust
testing_env!(context
            .storage_usage(env::storage_usage())
            .attached_deposit(MINT_STORAGE_COST)
            .predecessor_account_id(accounts(1))
            .build());
```

次の`vote_closed_test関数`では投票を終了することにまつわる関数がうまく動いているかを確認しています。

```rust
fn vote_closed_test() {
        let mut context = get_context(accounts(1));
        testing_env!(context.build());
        let mut contract = Contract::new_default_meta(accounts(1).into());
        testing_env!(context
            .storage_usage(env::storage_usage())
            .attached_deposit(MINT_STORAGE_COST)
            .predecessor_account_id(accounts(1))
            .build());
        assert_eq!(contract.is_election_closed, false);

        contract.close_election();
        assert_eq!(contract.is_election_closed, true);

        contract.reopen_election();
        assert_eq!(contract.is_election_closed, false);
    }
```

最後の`transfer_test関数`では NFT の transfer 機能がうまく動いているのか、つまりきちんと NFT の所有者が移譲されているかを確認しています。

```rust
fn transfer_test() {
        let mut context = get_context(accounts(1));
        testing_env!(context.build());
        let mut contract = Contract::new_default_meta(accounts(1).into());
        testing_env!(context
            .storage_usage(env::storage_usage())
            .attached_deposit(MINT_STORAGE_COST)
            .predecessor_account_id(accounts(1))
            .build());

        contract.nft_mint(
            TokenMetadata {
                title: None,
                description: None,
                media: "https...".to_string(),
                media_CID: "Qeo...".to_string(),
                candidate_name: None,
                candidate_manifest: None,
                token_kind: "candidate".to_string(),
                token_id: None,
            },
            accounts(1),
        );

        testing_env!(context
            .storage_usage(env::storage_usage())
            .attached_deposit(1)
            .predecessor_account_id(accounts(1))
            .build());

        contract.nft_transfer(accounts(2), 0);

        let nft_info = contract.nft_tokens(None, None);
        assert_eq!(nft_info[0].owner_id, accounts(2));
    }
```

それでは下のコマンドをターミナルで実行させることでテストをしてみましょう！

```bash
cargo test
```

下のような結果が返ってきていれば成功です！

```bash
running 3 tests
test tests::vote_closed_test ... ok
test tests::mint_test ... ok
test tests::transfer_test ... ok

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests near-election-dapp-contract

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

これで基本的な関数は動いていることが確認できました。

次は実際の testnet にデプロイして挙動を確認してみましょう！

前回の実装でそれぞれの候補者に紐づいた得票数の map への処理を加えたましたねが、それまでに deploy された NFT の情報は map に反映されずエラーが起きる可能性があります。

なので再度新しい wallet の id を作成して新しいコントラクトとして deploy してみましょう！

[こちら](https://wallet.testnet.near.org/)から作成できます。

section1-lesson4 で行った`new_default_meta関数をターミナルで走らせるところまで`行いましょう。

具体的には`export...`から`near call $NFT_CONTRACT_ID new_default_meta...`のところまでやりましょう！

それが完了したら下のコマンドをターミナルで実行させて、候補者の NFT を mint してみましょう！

画像の CID（IPFS で保存された画像の URI）や title は自由に変えてもらって大丈夫です！

```bash
near call $NFT_CONTRACT_ID nft_mint '{"metadata": {"title": "Rob Stark(candidate)", "description": "Forth Token", "media": "https://gateway.pinata.cloud/ipfs/QmQaBSeg58JcWkCxzGhqHiy9SSUugH9MtV8UnZQ3siMRYA", "media_CID": "QmQaBSeg58JcWkCxzGhqHiy9SSUugH9MtV8UnZQ3siMRYA", "candidate_name": "Rob Stark", "candidate_manifest": "In three words I can sum up everything I have learned about life it goes on.", "token_kind": "candidate"}, "receiver_id": "'$NFT_CONTRACT_ID'"}' --accountId $NFT_CONTRACT_ID --amount 0.1
```

これによって下のように新しく作成した wallet に候補者の NFT が mint されているはずです。
![](/public/images/401-NEAR-Election-dApp/section-2/2_3_1.png)

まずは下のコマンドをターミナルで実行させることで mint した NFT の値を確認してみましょう！

```bash
near view $NFT_CONTRACT_ID nft_tokens
```

下のように NFT の metadata が返ってくるはずです。

```bash
[
  {
    owner_id: 'YOUR_WALLET_ID',
    metadata: {
      title: 'Rob Stark(candidate)',
      description: 'Forth Token',
      media: 'https://gateway.pinata.cloud/ipfs/QmQaBSeg58JcWkCxzGhqHiy9SSUugH9MtV8UnZQ3siMRYA',
      media_CID: 'QmQaBSeg58JcWkCxzGhqHiy9SSUugH9MtV8UnZQ3siMRYA',
      candidate_name: 'Rob Stark',
      candidate_manifest: 'In three words I can sum up everything I have learned about life it goes on.',
      token_kind: 'candidate',
      token_id: 0
    }
  }
```

次に下のコマンドをターミナルで実行させることでこの NFT に紐づいた得票数を確認してみましょう。

```bash
near call $NFT_CONTRACT_ID nft_return_candidate_likes '{"token_id": 0}' --accountId $NFT_CONTRACT_ID
```

`0`という値が返ってくるはずです。
次に下のコマンドをターミナルで実行させることでこの NFT に紐づいた得票数を 1 大きくしてみましょう。

```bash
near call $NFT_CONTRACT_ID nft_add_likes_to_candidate '{"token_id": 0}' --accountId $NFT_CONTRACT_ID
```

返り値はないですが、再び下のコマンドをターミナルで実行させることでこの NFT に紐づいた得票数を確認してみましょう。

```bash
near call $NFT_CONTRACT_ID nft_return_candidate_likes '{"token_id": 0}' --accountId $NFT_CONTRACT_ID
```

`1`という値が返ってくれば成功です！

次は投票の締め切りができるか、また締め切っているのか確認できるかをテストしてみましょう！

下のコマンドをターミナルで実行させることで現在投票が締め切られているか確認してみましょう。

```bash
near view $NFT_CONTRACT_ID if_election_closed
```

`false`が返ってくるでしょう。

これは投票が締まっていないことを示しています。

では次に下のコマンドをターミナルで実行させることで投票を締め切りましょう。

```bash
near call $NFT_CONTRACT_ID close_election --accountId $NFT_CONTRACT_ID
```

再度、下のコマンドを入力してみてください。

```bash
near view $NFT_CONTRACT_ID if_election_closed
```

`true`に変わっているはずです。

では試しに下のコマンドをターミナルで実行させて違う候補者の NFT を mint してみましょう。

```bash
near call $NFT_CONTRACT_ID nft_mint '{"metadata": {"title": "Jenny Lind(candidate)", "description": "Seventh Token", "media": "https://gateway.pinata.cloud/ipfs/QmWUzLowW5ErzoezkpdSVZNF5LFgWTtMhiwfAdZU9LhcgF", "media_CID": "QmWUzLowW5ErzoezkpdSVZNF5LFgWTtMhiwfAdZU9LhcgF", "candidate_name": "Jenny Lind", "candidate_manifest": "Be yourself everyone else is already taken.", "token_kind": "candidate"}, "receiver_id": "'$NFT_CONTRACT_ID'"}' --accountId $NFT_CONTRACT_ID --amount 0.1
```

panic(rust における error)が起こっていれば OK です。

なぜなら投票が終了していたら mint も transfer も投票もできないようにしていますからね。

では下のコマンドをターミナルで実行させることで投票を再開してみましょう。

```bash
near call $NFT_CONTRACT_ID reopen_election --accountId $NFT_CONTRACT_ID
```

では下のコマンドをターミナルで実行させることで現在投票が再開されているか確認してみましょう。

```bash
near view $NFT_CONTRACT_ID if_election_closed
```

これで`false`に戻っていれば成功です。

では先ほど試みた mint をもう一度してみましょう!下のコマンドをターミナルで実行させてください。

```bash
near call $NFT_CONTRACT_ID nft_mint '{"metadata": {"title": "Jenny Lind(candidate)", "description": "Seventh Token", "media": "https://gateway.pinata.cloud/ipfs/QmWUzLowW5ErzoezkpdSVZNF5LFgWTtMhiwfAdZU9LhcgF", "media_CID": "QmWUzLowW5ErzoezkpdSVZNF5LFgWTtMhiwfAdZU9LhcgF", "candidate_name": "Jenny Lind", "candidate_manifest": "Be yourself everyone else is already taken.", "token_kind": "candidate"}, "receiver_id": "'$NFT_CONTRACT_ID'"}' --accountId $NFT_CONTRACT_ID --amount 0.1
```

投票が再開されているので mint が成功して下のようになっているはずです。

![](/public/images/401-NEAR-Election-dApp/section-2/2_3_2.png)

### 🙋‍♂️ 質問する

ここまでの作業で何かわからないことがある場合は、Discord の `#section-2` で質問をしてください。

ヘルプをするときのフローが円滑になるので、エラーレポートには下記の 4 点を記載してください ✨

```
1. 質問が関連しているセクション番号とレッスン番号
2. 何をしようとしていたか
3. エラー文をコピー&ペースト
4. エラー画面のスクリーンショット
```

---

おめでとうございます！

これで投票機能がきちんと実装できていることが確認できました。

セクション 2 は終了です。

投票が再開され、mint が成功しているフロントエンドのスクリーンショットを `#section-2` に投稿してください 😊

あなたの成功をコミュニティで祝いましょう 🎉

こう見るとシンプルですごさがあまりわかりませんが、たくさんの NFT が mint されてその中から特定の NFT に紐づいた得票数が取得できればすごいと思いませんか！？

次のセクションからはいよいよフロントエンドでコントラクトとの接続+UI の作成を行なっていきます。

楽しんでいきましょう！
