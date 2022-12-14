# フォーマッタ、静的解析
### フォーマッタ（rustfmt）

Rustツールチェインインストール時にあわせてインストールされている

静的解析前のコード

```rust
use rand::random; // useで読み込み

fn main() {
    let random_number: i32 =
     random();
    println!("`random_number` is {}", random_number);
}
```

rustfmtを実行

```rust
$ cargo fmt
```

ちゃんとこうなる

```rust
use rand::random; // useで読み込み

fn main() {
    let random_number: i32 = random();
    println!("`random_number` is {}", random_number);
}
```

### 静的解析（Clippy）

解析前のコード

```rust
use rand::random; // useで読み込み

fn main() {
    let random_number: i32 = random();
    println!("`random_number` is {}", random_number);

    if true == true {
        println!("trueです！");
    }
}
```

clippyを実行すると、

```rust
$ cargo clippy

Checking second-project v0.1.0 (/Users/kdm_snj/dev/practice/second-project)
error: equal expressions as operands to `==`.  // エラーの概要
 --> src/main.rs:8:8                           // エラーの位置
  |
8 |     if true == true {                      // エラー箇所のスニペット
  |        ^^^^^^^^^^^^
  |
  = note: `#[deny(clippy::eq_op)]` on by default 
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#eq_op

error: could not compile `second-project` due to previous error
```
