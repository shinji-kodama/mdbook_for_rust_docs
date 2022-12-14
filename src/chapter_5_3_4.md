# DBとの入出力
### 入出力用のデータ型の定義

repository.rsのuse行のあとに以下を記述

```rust
// repository.rs

use serde::{Deserialize, Serialize};

type DbPool =
    r2d2::Pool<ConnectionManager<SqliteConnection>>; // ここは不変

// POSTリクエスト時に受け取る記事データを持つ型を定義
#[derive(Deserialize, Insertable)] // Insertable: DieselがNewPost型からDBへINSERTできるようになる
#[diesel(table_name = posts)] // 対象テーブルの指定
pub struct NewPost {
    title: String,
    body: String,
}

// レコードの全フィールドを持つ型
#[derive(Serialize, Queryable)] // Queryable: DieselがDBのレコードを格納できるようにする
pub struct Post {
    id: i32,
    title: String,
    body: String,
    published: bool,
}

```

### 記事のレコードをINSERTする`create_post`メソッドを作成

repository.rsのuse行へ追記
`use serde::{Deserialize, Serialize};`を忘れないよう注意

```rust
// repository.rs

use diesel::prelude::*;
use crate::error::ApiError;
use actix_web::web;
```

### Repository型にcreate_postメソッドを追記

`web::block`について 

`diesel::insert_into`は同期的なメソッドであり、そのまま使うとブロッキングする

このようなブロッキングする入出力をasyncで使いたい時のために、起動時に用意したスレッドプールで実行する機能

`move`で引数のクロージャを別スレッドに移しているため、借用ではデータを受け渡せない。

```rust
// repository.rs

impl Repository {
	pub fn new(database_url: &str) // 前述のため省略

	// create_postメソッドを追加
	// NewPost型の値を引数に取り、DBにINSERT
	// 成功時：登録したレコード(Post型)を返す
  // エラー時は?が書かれているタイミングでApiError型が返る
	pub async fn create_post(
        &self,
        new_post: NewPost,
    ) -> Result<Post, ApiError> {
        let mut conn = self.pool.get()?;
        let post = web::block(move || {
            diesel::insert_into(posts::table)
                .values(new_post)
                .get_result(&mut conn)
        })
        .await??; // この??は?を２回適用している

        Ok(post)
    }
}
```

### 記事を投稿するAPIを実装

main.rsとerror.rsを書き換える

各エンドポイントはエラー発生の可能性があるため`Result`型を返せる様にする。エラー時に`ApiError`型からHTTPレスポンスへ変換できるよう、`ApiError`に`ResponseError`トレイトを実装してエラーごとの`HttpResponse`を返すようにしている

- main.rs

```rust
// main.rs

mod error;
mod repository;
mod schema;

use actix_web::{web, App, HttpResponse, HttpServer};
// 追加
use actix_web::middleware::{Logger, NormalizePath};
use error::ApiError;
use repository::{NewPost, Repository};

// エラーの可能性があるため、Result型を返すエンドポイント
// 引数はrepoとnew_post
#[actix_web::post("/posts")]
async fn create_post(
    repo: web::Data<Repository>,  // サーバー起動時に①で共有データとして登録されるコレクションプール
    new_post: web::Json<NewPost>, // リクエストボディのJSONをNewPost型にでシリアライズした値
) -> Result<HttpResponse, ApiError> {
    let new_post = new_post.into_inner();
    let post = repo.create_post(new_post).await?; // エラー発生の可能性があり、Result型を返す.
    Ok(HttpResponse::Ok().json(post))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {

		// 環境変数からURLを取り出す
    let database_url =
        std::env::var("DATABASE_URL").unwrap();

		// repoを作る
		// web::DataはArcという参照カウントを使ったスマートポインタを拡張した型
    // Repositoryのデータ、つまりコネクションプールを各POSTリクエスト処理で共有できる様になる 
    let repo = web::Data::new(Repository::new(  
        &database_url,
    ));

    HttpServer::new(move || {
        App::new()
            .app_data(repo.clone()) // ① 引数のrepoはここで共有データとして登録され、各エンドポイントの関数で参照できる
            .service(create_post)
    })
    .bind(("0.0.0.0", 8080))?
    .run()
    .await
}
```

- error.rsに追記

```rust
// error.rs

use actix_web::{HttpResponse, ResponseError};

// エラー時にApiError型からHTTPレスポンスへ変換できるよう
// ApiErrorにRespinseError traitを実装し、エラーごとのHttpResponseを返す様にしている
impl ResponseError for ApiError {
    fn error_response(&self) -> HttpResponse {
        match self {
            ApiError::NotFound => {
                HttpResponse::NotFound().finish()
            }
            ApiError::Other(_) => {
                HttpResponse::ServiceUnavailable()
                    .finish()
            }
        }
    }
}
```
