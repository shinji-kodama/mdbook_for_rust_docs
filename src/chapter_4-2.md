# 関数の作成
### プロジェクトを新規作成
以下のコマンドでlambda用のプロジェクトを作成。

AWSの認証情報をすぐに用意できない場合はこちらで用意した認証情報を使っていただけますが、deploy時に上書きされるため、プロジェクト名が被らない様にお願いします

```bash
$ cargo lambda new lambda-rust-test
```
対話形式のメニューが表示されるが、トリガーとするイベントは選択しなくてOK（Enter2回押すだけ）

### 関数が生成される

```rust
use lambda_runtime::{run, service_fn, Error, LambdaEvent};

use serde::{Deserialize, Serialize};

/// This is a made-up example. Requests come into the runtime as unicode
/// strings in json format, which can map to any structure that implements `serde::Deserialize`
/// The runtime pays no attention to the contents of the request payload.
#[derive(Deserialize)]
struct Request {
    command: String,
}

/// This is a made-up example of what a response structure may look like.
/// There is no restriction on what it can be. The runtime requires responses
/// to be serialized into json. The runtime pays no attention
/// to the contents of the response payload.
#[derive(Serialize)]
struct Response {
    req_id: String,
    msg: String,
}

/// This is the main body for the function.
/// Write your code inside it.
/// There are some code example in the following URLs:
/// - https://github.com/awslabs/aws-lambda-rust-runtime/tree/main/examples
/// - https://github.com/aws-samples/serverless-rust-demo/
async fn function_handler(event: LambdaEvent<Request>) -> Result<Response, Error> {
    // Extract some useful info from the request
    let command = event.payload.command;

    // Prepare the response
    let resp = Response {
        req_id: event.context.request_id,
        msg: format!("Command {}.", command),
    };

    // Return `Response` (it will be serialized to JSON automatically by the runtime)
    Ok(resp)
}

#[tokio::main]
async fn main() -> Result<(), Error> {
    tracing_subscriber::fmt()
        .with_max_level(tracing::Level::INFO)
        // disable printing the name of the module in every log line.
        .with_target(false)
        // disabling time is handy because CloudWatch will add the ingestion time.
        .without_time()
        .init();

    run(service_fn(function_handler)).await
}
```

### ローカルで関数をテスト
以下のコマンドでlistenする様になる

dockerの場合は`-a 0.0.0.0`をつける

```bash
$ cargo lambda watch -a 0.0.0.0
```

### リクエストを飛ばして関数を起動
以下のコマンドでリクエスト

```bash
$ cargo lambda invoke --data-ascii '{"command": "hi"}' --output-format json
```