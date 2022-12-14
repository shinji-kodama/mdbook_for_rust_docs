# 完成形
### main.rs
    
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
    
### repoitory.rs
    
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
    
### error.rs
    
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
