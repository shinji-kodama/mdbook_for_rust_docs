# パッケージの追加
cargo addもしくはCargo.tomlの編集でサードパーティのパッケージを追加できる。今回はrand クレートをプロジェクトに追加（[https://crates.io/crates/rand](https://crates.io/crates/rand)）

※クレート：JavaScript等で言うところのライブラリ

```bash
$ cargo add rand
```

Cargo.tomlに依存関係が追記される

```toml
[dependencies]
rand = "0.8.5" # 追記される
```

main.rsでrandを使うように書き換える

```rust
use rand::random; // useで読み込み

fn main() {
    let random_number: i32 = random();
    println!("`random_number` is {}", random_number);
}
```

実行

```bash
$ cargo run
   Compiling cfg-if v1.0.0
   Compiling ppv-lite86 v0.2.17
   Compiling libc v0.2.137
   Compiling getrandom v0.2.8
   Compiling rand_core v0.6.4
   Compiling rand_chacha v0.3.1
   Compiling rand v0.8.5
   Compiling first-project v0.1.0 (/Users/kdm_snj/dev/practice/first-project)
    Finished dev [unoptimized + debuginfo] target(s) in 1.21s
     Running `target/debug/first-project`
`random_number` is 123821674
```

参考：Rustのパッケージ公開サイト  [crates.io: Rust Package Registry](https://crates.io/)

## このサイトについて
このサイトもRustを使って作成しています

mdbookというクレートでmarkdownからwebサイトを生成できます

参考：[mdbookについて](https://www.notion.so/kame-kame/mdbook-91844acf12e84af0aa0d5fb65dae4906)