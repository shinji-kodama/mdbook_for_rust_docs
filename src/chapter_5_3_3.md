# DBとの接続
### DBとの接続部分を作成

Repository型を以下の様に作成

DB接続処理（`new`）ではエラーが起こり得るため強制終了させる`expect`を使用

repository.rs

```rust
use crate::schema::*;
use diesel::r2d2::{self, ConnectionManager};
use diesel::sqlite::SqliteConnection;

type DbPool =
    r2d2::Pool<ConnectionManager<SqliteConnection>>;

pub struct Repository {
		pool: DbPool,
}

impl Repository {
		pub fn new(database_url: &str) -> Self {
        let manager = ConnectionManager::<
            SqliteConnection,
        >::new(database_url);
        let pool = r2d2::Pool::builder()
            .build(manager)
            .expect("Failed to create a pool.");  // エラーの場合はパニックさせて強制終了
        Self { pool }
    }
}
```

### DBへの接続テスト

同じファイルにテストを記述できるため、repository.rsの続きに以下を記述

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_conn() {
        let database_url =
            std::env::var("DATABASE_URL").unwrap();
        let repo = Repository::new(&database_url);
        assert!(repo.pool.get().is_ok());
    }
}
```

watchタスクが実行中(`cargo make --env-file .env watch`で実行)だと、自動テストが走ってターミナルには以下のような表示が出る

```bash
runnning 1 test
test repository::test::test_conn ... ok

test result:ok. 1 passed; 0 failed out; finised in 0.01s
```

### エラー定義を作成

RustアプリケーションとDB間での入出力はエラーの可能性が伴う

サーバー起動時はパニックさせたが、その後のエラー発生時はクライアントにエラーを返すようにする

返すエラーは以下の２つ

- ID 指定で記事を取得するときに対象の記事が存在しない（404 NOt Found）
- それ以外（503 Service Temporary Unavailable）

`From`トレイト：型の変換に伴う標準のトレイトで、`From`を実装していると、`?`の使用時に自動的に適用される

[RustのFromトレイトとIntoトレイト - dackdive's blog](https://dackdive.hateblo.jp/entry/2021/04/30/100000)

error.rsを編集

```rust
// error.rs

// NotFoundとOtherの2値を持つ列挙型として定義
#[derive(thiserror::Error, Debug)]
pub enum ApiError {
    #[error("Post not found")]
    NotFound,
    #[error(transparent)]
    Other(anyhow::Error),
}

// 定義の記述を楽にするための関数的マクロ
// 使うクレートから返る可能性のあるエラー型からApiError::Otherに変換するためのFromトレイトの実装
macro_rules! impl_from_trait {
    ($etype: ty) => {
        impl From<$etype> for ApiError {
            fn from(e: $etype) -> Self {
                ApiError::Other(anyhow::anyhow!(e))
            }
        }
    };
}

impl_from_trait!(diesel::r2d2::Error);
impl_from_trait!(diesel::r2d2::PoolError);
impl_from_trait!(diesel::result::Error);
impl_from_trait!(actix_web::error::BlockingError);
```