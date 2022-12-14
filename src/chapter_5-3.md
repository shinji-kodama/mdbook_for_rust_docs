# DieselでDBを扱うAPIを実装
以下のAPIを持つブログ記事投稿のAPIを作成

- 記事を投稿する

- 記事データの一覧を取得する

- 指定したID

入出力は全てJSON、記事データはレコードとして永続化




## 実装

### 概要

実装内容

1. 記事を投稿
    
    記事タイトルと本文を含むJSONをPOSTでうけとりDBに保存
    
2. 記事データの一覧を取得
    
    GETを受けてすべての記事リストをJSONで返す
    
3. 指定したIDの記事を取得
    
    記事IDを指定したGETを受け該当記事のJSONを返す
    

実装の流れ

1. 依存クレートの追加
2. 実装するファイルの準備
3. 実装を始める
    1. DBとの接続処理の作成
    2. エラー定義の作成
    3. DBへの入出力処理の作成
    4. APIの作成
    5. ミドルウェアの追加

### クレートの追加

以下のクレートを追加

既出：`actix-web`, `serde`, `diesel`

SQLite3同梱の接続用クレート：`libsqlite3-sys`

エラーを簡単に扱うためのクレート：`anyhow`, `thiserror`

```bash
$ cargo add actix-web
$ cargo add serde -F derive
$ cargo add diesel -F r2d2 -F returning_clauses_for_sqlite_3_35
$ cargo add libsqlite3-sys -F bundled
$ cargo add anyhow thiserror
```

実装用のファイルを準備

```bash
src
├── error.rs  # 追加
├── main.rs
├── repository.rs # 追加
└── schema.rs
```

main.rsに以下を記述

```rust
mod error;
mod repository;
mod schema;
```

### DBとの接続部分を作成

Repository型を以下の様に作成

DB接続処理（`new`）ではエラーが起こり得るため強制終了させる`expect`を使用

repository.rs

```rust
use crate::schama::*;
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

watchタスクが実行中(`cargo make --env-file .env watch`)だと、自動テストが走ってターミナルには以下のような表示があるはず

```
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

error.rs

```rust
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

### DBとの入出力を作成

入出力用のデータ型の定義

repository.rsのuse行のあとに以下を記述

```rust
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

記事のレコードをINSERTする`create_post`メソッドを作成

repository.rsのuse行へ追記

```rust
use diesel::prelude::*;
use crate::error::ApiError;
use actix_web::web;
```

Repository型にcreate_postメソッドを追記

`web::block`について 

`diesel::insert_into`は同期的なメソッドであり、そのまま使うとブロッキングする

このようなブロッキングする入出力をasyncで使いたい時のために、起動時に用意したスレッドプールで実行する機能

`move`で引数のクロージャを別スレッドに移しているため、借用ではデータを受け渡せない。

```rust

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

main.rs

```rust
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

error.rs

```rust
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

---

### ここまでの.rsファイル途中経過

- error.rs
    
    ```rust
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
    
- main.rs
    
    ```rust
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
    
- repository.rs
    
    ```rust
    use crate::error::ApiError;
    use crate::schema::*;
    use diesel::prelude::*;
    use diesel::r2d2::{self, ConnectionManager};
    use diesel::sqlite::SqliteConnection;
    use serde::{Deserialize, Serialize};
    use actix_web::web;
    
    type DbPool =
        r2d2::Pool<ConnectionManager<SqliteConnection>>;
    
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
        .await??; // この??は?を２回適用しているだけ
    
        Ok(post)
      }
    }
    
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
    

### 記事の作成ができる様になったので動かしてみる

以下のコマンドでサーバーを起動

```bash
$ cargo make --env-file .env run
```

HTTPリクエストを投げると、APIからは投稿した内容が返ってくる

```bash
$ curl -X POST 'http://localhost:8080/posts' -H 'Content-Type: application/json' -d 
'{"title": "こんにちは", "body": "初投稿です"}'

{"id": 1, "title": "こんにちは", "body": "初投稿です", "published": false }
```

sqlite3コマンドを使える場合は、selectクエリを実行して確認

```bash
$ sqlite3 posts.db

sqlite> select * from posts;

1|こんにちは|初投稿です|f
```

### 記事の一覧を取得するAPIを実装

GETリクエストで記事を取得するAPIを実装する

repository.rsに追加

```rust
impl Repository {
    pub fn new(database_url: &str) // (省略)
    pub async fn create_post(&self, new_post: NewPost) // (省略)

		
    pub async fn list_posts(
        &self,
    ) -> Result<Vec<Post>, ApiError> {
        let mut conn = self.pool.get()?;
        let res = web::block(move || {
            posts::table.load(&mut conn)
        })
        .await??;

        Ok(res)
    }

    pub async fn get_post(
        &self,
        id: i32,
    ) -> Result<Post, ApiError> {
        let mut conn = self.pool.get()?;
        let res = web::block(move || {
            posts::table
                .find(id)
                .first(&mut conn)
                .optional()
        })
        .await??
        .ok_or(ApiError::NotFound)?;

        Ok(res)
    }
}
```

main.rsへの追加

```rust
#[actix_web::get("/posts")]
async fn list_posts(
    repo: web::Data<Repository>,
) -> Result<HttpResponse, ApiError> {
    let res = repo.list_posts().await?; // Repository.rsのlists_posts
    Ok(HttpResponse::Ok().json(res))
}

#[actix_web::get("/posts/{id}")]
async fn get_post(
    repo: web::Data<Repository>,
    path: web::Path<i32>,
) -> Result<HttpResponse, ApiError> {
    let id = path.into_inner();
    let res = repo.get_post(id).await?; // Repository.rsのget_post
    Ok(HttpResponse::Ok().json(res))
}

```

main.rsのmain関数に追記

```rust
App::new()
	  .app_data(repo.clone())
	  .service(create_post)
	  .service(list_posts)  // 追加   
	  .service(get_post)    // 追加
```

### 各APIの動作を確認

```bash
# 新しい記事をpost
$ curl -X POST 'http://localhost:8080/posts' -H 'Content-Type: applocation/json' -d \
'{"title":"こんばんは","body":"2回目の投稿です"}'
{"id":2,"title":"こんばんは","body":"2回目の投稿です","published":false}

$ curl 'http://localhost:8080/posts'
[{"id":1,"title":"こんにちは","body":"初投稿です","published":false},{"id":2,"title":"こんばんは","body":"2回目の投稿です","published":false}]

$ curl 'http://localhost:8080/posts/1'
{"id":1,"title":"こんにちは","body":"初投稿です","published":false}

ステータスコード404を確認するためのコマンド
$ curl 'http://localhost:8080/posts/1000' -o /dev/null -w '%{http_code}\n' -s
404
```

### ミドルウェアを追加

Actix webではミドルウェアを実装できる。

以下を実装する

- Logger：アクセスログの出力
- NormalizePath：リクエストパスの正規化

- Logger
    
    Loggerの機能を使うため、ログ出力用のクレートを追加
    
    今回は`tracing`を使用
    
    tracing：非同期アプリケーション内のトレーシングログを記録する機能がメイン（今回は標準出力にログを出力するために使用）
    
    ```bash
    $ cargo add tracing tracing-subscriber
    ```
    
- NormalizePathによるリクエストパスの正規化
    
    現在のエンドポイント：`http://localhost:8080/posts` 
    
    最後に`/`がある(trailing slash)とアクセスできないので、リクエストを処理できるようにする
    
    （wasm使用時の動作確認時の制約があるため）
    
    main.rs
    
    use行に追加
    
    ```rust
    use actix_web::middleware::{Logger, NormalizePath}
    ```
    
    main()に`.wrap`でミドルウェアを追加
    
    ```rust
    async fn main() -> std::io::Result<()> {
    	tracing_subscriber::fmt::init(); // 追加
    
    	// 省略
    
    	HttpServer::new(move || {
    		App::new()
    			.app_date(repo.clone())
    			.wrap(Logger::default()) // 追加
    			.wrap(NormalizePath::trim()) // 追加
    			// 省略
    
    	})
    
    	// 省略
    
    }
    ```
    

## 完成系

- main.rs
    
    ```rust
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
    
    #[actix_web::get("/posts")]
    async fn list_posts(
        repo: web::Data<Repository>,
    ) -> Result<HttpResponse, ApiError> {
        let res = repo.list_posts().await?; // Repository.rsのlists_posts
        Ok(HttpResponse::Ok().json(res))
    }
    
    #[actix_web::get("/posts/{id}")]
    async fn get_post(
        repo: web::Data<Repository>,
        path: web::Path<i32>,
    ) -> Result<HttpResponse, ApiError> {
        let id = path.into_inner();
        let res = repo.get_post(id).await?; // Repository.rsのget_post
        Ok(HttpResponse::Ok().json(res))
    }
    
    #[actix_web::main]
    async fn main() -> std::io::Result<()> {
    
    	tracing_subscriber::fmt::init(); // ミドルウェア追加時に追加
    
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
    			.wrap(Logger::default()) // ミドルウェア追加
    			.wrap(NormalizePath::trim()) // ミドルウェア追加
                .service(create_post)
                .service(list_posts)  // main.rsにlist_postsメソッド追加後に追加   
                .service(get_post)    // 同情
        })
        .bind(("0.0.0.0", 8080))?
        .run()
        .await
    }
    ```
    
- repoitory.rs
    
    ```rust
    use crate::error::ApiError;
    use crate::schema::*;
    use diesel::prelude::*;
    use diesel::r2d2::{self, ConnectionManager};
    use diesel::sqlite::SqliteConnection;
    use serde::{Deserialize, Serialize};
    use actix_web::web;
    
    type DbPool =
        r2d2::Pool<ConnectionManager<SqliteConnection>>;
    
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
        .await??; // この??は?を２回適用しているだけ
    
        Ok(post)
      }
    
      // list_postsメソッドを追加
      pub async fn list_posts(
        &self,
      ) -> Result<Vec<Post>, ApiError> {
        let mut conn = self.pool.get()?;
        let res = web::block(move || {
            posts::table.load(&mut conn)
        })
        .await??;
    
        Ok(res)
      }
    
      // get_postメソッドを追加
      pub async fn get_post(
        &self,
        id: i32,
      ) -> Result<Post, ApiError> {
        let mut conn = self.pool.get()?;
        let res = web::block(move || {
            posts::table
                .find(id)
                .first(&mut conn)
                .optional()
        })
        .await??
        .ok_or(ApiError::NotFound)?;
    
        Ok(res)
      }
    
    }
    
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
    
- error.rs
    
    ```rust
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
    
- 実行
    
    ```bash
    $ curl -X POST
    ```