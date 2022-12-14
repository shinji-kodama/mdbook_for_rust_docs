# AWS LambdaでRustを使う
既存システムにRustを組み込む場合、全体のリプレイスは現実的ではないため、言語バインディングを用いたRustモジュールの追加やFaas等を使ったAPIの一部の置き換えになることが多い。

ここではFaasを用いたAPIを実装していく

## Faas等で一部の関数をRustで置き換える

各プラットフォームでの対応状況を確認する

### AWSの対応状況

GitHubでRust製のクレートをいくつか公開している

- `aws-lambda-rust-runtime`：`Lambda`向けにのRust用のランタイムクレート
- `rusoto`：デファクトとして存在していたが現在はメンテナンスモード
- `aws-sdk-rust`：サービス開始から１年経ってもアルファ版
- `rust-s3`：使うサービスが限定的ならこういうのもある

### GCPの対応状況

公式にクライアントSDKは提供されていないので、自力でAPIに合わせてクライアントを実装するか、サードパーティのクレートを使う必要がある

- Cloud Functionsはランタイムサポートがなく、Lambdaのようなカスタムランタイムも対応していないのでRustを使うのは困難
- RDBMS（Cloud SQL）やRedis（Cloud Memorystore）ベースのサービスは接続用クレート（`Diesel`, `redis-rs`）があれば利用可能

### Azureの対応状況

GitHub上のAzure公式でクライアントSDKが提供されている。が、「unofficial」と明記されている（有志が作成したらしい）

- Azure Functionsでは公式でRustの関数のデプロイ方法が公開されている

### Cloudflareの対応状況

`Cloudflare Workers`がRustをサポートしている。V8のWevAssemblyエンジンを使ってWebAssembly化したRustプログラムを動作させている