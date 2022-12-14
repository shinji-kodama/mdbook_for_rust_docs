# Actix Webでサーバーを作成
### プロジェクト作成

`actix-web-helloというプロジェクト`を作成し、`actix-web`をインストール

```bash
$ cargo new actix-web-hello
$ cd actix-web-hello
$ cargo add actix-web
```

### 実装

`/`でGETリクエストを受け取り、JSONを返すだけのwebサーバーを作成


```rust
// main.rs

// このプログラムで使う型を指定
use actix_web::{ 
    http::header::ContentType, App, HttpResponse,HttpServer,
};

// アトリビュートマクロ。HTTPメソッドのルーティングを指定。
#[actix_web::get("/")] 
async fn hello() -> HttpResponse {
    HttpResponse::Ok()
        .append_header(ContentType::json())
        .body(r#"{"greet":"Hello, world!"}"#) // r#" "#は文字列リテラル
}

// 独自の非同期ランタイムを定義しているため、このアトリビュートが必要
#[actix_web::main] 
async fn main() -> std::io::Result<()> {
    // webアプリケーションの機能定義（App）を引数にとり、HttpServerインスタンスを作成
    HttpServer::new(|| App::new().service(hello)) 
        .bind(("0.0.0.0", 8080))?  // 自身のアドレスとポートを指定（bind）
        .run()
        .await
}
```

### サーバーの起動

以下のコマンドでサーバーを起動

```bash
$ cargo run
```

### GETリクエストをでレスポンスを受け取ってみる
curlで`localhost:8080`を叩く

```bash
$ curl "http://localhost:8080/"

{"greet":"Hello, world!"}
```