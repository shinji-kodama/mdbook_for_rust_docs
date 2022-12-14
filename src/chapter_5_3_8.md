# ミドルウェアの追加

## Actix webでミドルウェアを実装

以下を実装する

- Logger：アクセスログの出力
- NormalizePath：リクエストパスの正規化

### Logger
    
Loggerの機能を使うため、ログ出力用のクレートを追加

今回は`tracing`を使用

tracing：非同期アプリケーション内のトレーシングログを記録する機能がメイン（今回は標準出力にログを出力するために使用）

```bash
$ cargo add tracing tracing-subscriber
```
    
### NormalizePathによるリクエストパスの正規化
    
現在のエンドポイント：`http://localhost:8080/posts` 

最後に`/`がある(trailing slash)とアクセスできないので、リクエストを処理できるようにする（wasm使用時の動作確認時の制約があるため）

main.rsのuse行にmiddlewareを追加

```rust
// main.rs

use actix_web::middleware::{Logger, NormalizePath}
```

main()関数内で`.wrap`でミドルウェアを追加

```rust
// main.rs

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