# チュートリアル

チュートリアルをやっていくよ！ [公式ページ](https://www.rust-lang.org/learn/get-started#installing-dependencies)

dockerで環境を用意しています。[GitHub shinji-kodama/rust_practice](https://github.com/shinji-kodama/rust_practice)

### dockerを使う場合

コンテナの中で作業します

```bash
$ docker compose up -d
$ docker compose exec rust-app bash
```


### hello-rustプロジェクトを生成

```bash
$ cargo new hello-rust
```

### 実行
Hello worldできるか確認

```bash
$ cargo run
	Compiling hello-rust v0.1.0 (/Users/ag_dubs/rust/hello-rust)
    Finished dev [unoptimized + debuginfo] target(s) in 1.34s
     Running `target/debug/hello-rust`
Hello, world!
```

### tutorial用のクレートを追加

`Cargo.toml`に以下の依存性を追加

```toml
[dependencies]
ferris-says = "0.2"
```

`cargo add ferris-says`でもokですが、ver. 0.3 になるとsay関数の仕様が少し変わるためエラー出ます

### build

```bash
$ cargo build
```

### main.rsを書き換える

```rust
use ferris_says::say; // from the previous step
use std::io::{stdout, BufWriter};

fn main() {
    let stdout = stdout();
    let message = String::from("Hello fellow Rustaceans!");
    let width = message.chars().count();

    let mut writer = BufWriter::new(stdout.lock());
    say(message.as_bytes(), width, &mut writer).unwrap();
}
```

### 実行

Rustのイメージキャラクターのferrisくんが挨拶してくれました！

```bash
$ cargo run

----------------------------
< Hello fellow Rustaceans! >
----------------------------
              \
               \
                 _~^~^~_
             \) /  o o  \ (/
               '_   -   _'
               / '-----' \
```

 
参考：BufWriterについてとか（[Rustで高速な標準出力 | κeenのHappy Hacκing Blog](https://keens.github.io/blog/2017/10/05/rustdekousokunahyoujunshutsuryoku/)）