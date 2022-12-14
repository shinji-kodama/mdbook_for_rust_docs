# http関数を作成

関数作成時に`--http`をつけるところだけ違う（オプションなしで選択してもOK）

```bash
$ cargo lambda new rust_lambda_http --http
$ cd rust_lambda_http
```



受け取ったリクエストの中身を

`tracing`という非同期アプリケーションのロギングクレートでログを確認

```rust
// main.rs

use lambda_http::{run, service_fn, Body, Error, Request, Response};

async fn function_handler(event: Request) -> Result<Response<Body>, Error> {

    // ここだけ追加！！ 受け取ったeventの中身をログに残す
    tracing::info!("event: {:#?}", event);  

    let resp = Response::builder()
        .status(200)
        .header("content-type", "text/html")
        .body("Hello AWS Lambda HTTP request".into())
        .map_err(Box::new)?;
    Ok(resp)
}

#[tokio::main]
async fn main() -> Result<(), Error> {
    tracing_subscriber::fmt()
        .with_max_level(tracing::Level::INFO)
        .with_target(false)
        .without_time()
        .init();

    run(service_fn(function_handler)).await
}
```

リッスンして

```bash
$ cargo lambda watch
```

テストデータをpostしてみる

[aws-lambda-events/example-apigw-request.json at master · calavera/aws-lambda-events](https://github.com/calavera/aws-lambda-events/blob/master/aws_lambda_events/src/generated/fixtures/example-apigw-request.json)

```bash
$ cargo lambda invoke --data-example apigw-request
```

デプロイ方法は一緒なので割愛