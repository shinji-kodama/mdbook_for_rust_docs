# 条件分岐

- `if`が使える

- 文ではなく式なので三項演算子っぽく使える

- 条件式に()が不要

```rust
fn main() {
    let x = 100;
    let y = 50;

    if x == y {
        println!("same value!");
    }

    // if式の評価結果を変数に束縛する
    let z = if x != y { 500 } else { 300 };

    println!("z: {}", z);
}
```