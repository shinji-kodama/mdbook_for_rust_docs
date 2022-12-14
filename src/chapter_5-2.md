# Serdeを使用したAPIの実装
jsonでのデータのやり取りを簡単に実装

## Serdeとは？
**Rustの型から他のデータ形式へのシリアライズとデシリアライズをサポート**するクレート

- アプリケーション外部のデータと型の変換をフレームワークにお任せできるようになる

- 基本的にはderiveでの自動生成機能を使える（自前で実装するケースは非常に稀とのこと）

- 説明は以下のページとか見るとわかりやすい（かも）

[[Rust] Serdeのシリアライズ/デシリアライズを試してみる | DevelopersIO](https://dev.classmethod.jp/articles/rust-serde-getting-started/)

[crates.io: Rust Package Registry](https://crates.io/crates/serde)

[Using derive](https://serde.rs/derive.html)

### インストール
以下のコマンドを実行してserdeをインストール

- `-F`オプション：クレートの機能選択に使う「features」を有効にするオプション

- `derive`：deriveフィーチャを指定。deriveアトリビュートで型にシリアライズ/デシリアライズの機能を追加できるようになる

```bash
$ cargo add serde -F derive
```

### APIの実装

nameとageのクエリを受け取り、レスポンス内の文字列の一部にクエリの値をはめて返すAPI

- リクエストパラメータ
    
    Actix Webではリクエストパラメータを関数の引数で受け取る
    
    **正しく引数の型に変換できないパラメータを受け取ると`400（BaeRequest）`を返す**
    
    （ここでは、`u32`で宣言している`age`に負の値や文字列を受け取る場合）
    
- レスポンス
    
    jsonメソッドに「Serializeをderiveした型のデータ」を渡すと
    
    自動的に シリアライズ → Content-Typeの設定 まで行ってくれる
    

```rust
// main.rs

use actix_web::{web, App, HttpResponse, HttpServer};
// useでSerdeの型をインポート
use serde::{Deserialize, Serialize};

// deriveアトリビュートでDeserializeを指定し、コンパイル時にtraitを自動生成
#[derive(Deserialize)]
struct HelloQuery {
    name: String,
    age: u32,
}

// Serializeをderive　JSONに変換するtraitを自動生成
#[derive(Serialize)]
struct HelloResponse {
    greet: String,
}

#[actix_web::get("/")]
async fn hello(
    query: web::Query<HelloQuery>,  // 仮引数にクエリ -> リクエストパラメータを引数で受け取る
) -> HttpResponse {
    let query = query.into_inner(); // クエリから値を取り出す
    let message = format!(          // 文字列メッセージを作成
        "Hello, my name is {}! I am {} years old!",
        query.name, query.age
    );
    let h = HelloResponse { greet: message }; // HelloResponse型にレスポンスするメッセージ入れ
		
    HttpResponse::Ok().json(h) // 200 OKのレスポンスに渡す
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| App::new().service(hello))
        .bind(("127.0.0.1", 8080))?
        .run()
        .await
}
```

### apiを叩いてみる

curlとかで確かめてみる

```bash
$ curl "http://localhost:8080/?name=Taro&age=20"
{"greet":"Hello, my name is Taro! I am 20 years old!"}

$ curl "http://localhost:8080/"
Query deserialize error: missing field `name`

$ curl "http://localhost:8080/?name=Taro"
Query deserialize error: missing field `age`

$ curl "http://localhost:8080/?name=Taro&age=xxx"
Query deserialize error: invalid digit found in string
```