# async/await

言語仕様では記述方法のみ規定されている

実行時利用するランタイムは別途指定する必要がある（Tokio、async-std、smol等）

以下はTokioを使う場合のサンプルコード

ランタイムクレート（実行時に必要なライブラリ）はmain 関数をアトリビュートマクロでラップするとfn main()の関数の形に変換して実行する

```rust
#[tokio::main]
async fn main() -> std::io::Result<()> {
    let _ = tokio::fs::read("file.txt").await?;
    Ok(())
}
```