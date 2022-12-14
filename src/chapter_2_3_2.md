# テスト
Rustのソースコードに直接書いたユニットテストやtestsディレクトリに置いたテストコードを`cargo test`で実行できる

### テストの作成

ソースコード上に`#[test]`アトリビュートをつけて関数をかくと、`cargo test`で実行される

ゼロ除算のようにクラッシュ（panic）するものは`#[should_panic]`アトリビュートをつける

新規プロジェクトを作成

```bash
$ cargo new third-project
```

main.rsを以下のように編集

```rust
fn main() {
    println!("{}", div(4, 2));
}

fn div(x: i32, y: i32) -> i32 {
    x / y
}

#[test]
fn div_test() {
    assert_eq!(div(10,3), 3);
}

#[test]
#[should_panic]
fn div_panic_test() {
    div(2,0);
}
```

### 実行

以下のコマンドで実行

2つのテストが問題なく通る

```bash
$ cargo test

Compiling third-project v0.1.0 (/Users/kdm_snj/dev/practice/third-project)
    Finished test [unoptimized + debuginfo] target(s) in 0.40s
     Running unittests src/main.rs (target/debug/deps/third_project-ae498afcbed4497c)

running 2 tests
test div_test ... ok
test div_panic_test - should panic ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

# テスト2つ全てクリア
```

### 一般的な書き方、参考

テスト時以外はコンパイルされないよう、以下のように別モジュールとする書き方が一般的

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn div_test() {
		    assert_eq!(div(10, 3), 3);
		}
}
```

- `cfg`
    
    *configuration*の略
    
    ある特定の設定オプションを与えられたら、コンパイルに含まれるよう指示
    
- `#[cfg(test)]`
    
    コンパイラに`cargo test`を走らせた時だけコンパイルするよう指示
    
    `cargo build`ではコンパイルしない
    
- `mod`
    
    参考：
    
    [The Rust Programming Language 日本語版](https://doc.rust-jp.rs/book-ja/ch07-02-defining-modules-to-control-scope-and-privacy.html)
    
    モジュールの定義
    
    モジュールを使うことで関連する定義を一つにまとめ、関連する理由を名前で示せる
    
    以下のように、mod内にmodやenum, trait, struct, fn等を入れ子で持たせることも可能
    
    ```rust
    mod front_of_house {
        mod hosting {
            fn add_to_waitlist() {}
    
            fn seat_at_table() {}
        }
    
        mod serving {
            fn take_order() {}
    
            fn serve_order() {}
    
            fn take_payment() {}
        }
    }
    ```
    
- `super`
    
    `super::要素名`で親が持つ要素を参照する。直下の子モジュール以外を参照する場合は`use`が必要