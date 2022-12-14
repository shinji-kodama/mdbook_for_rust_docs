# 変数

- letで宣言（Rustでは「束縛」という）

- 型はtypescriptっぽく変数の後ろに書く

- 基本immutable （mutableにしたい場合はmutをつけて束縛）

- 変数は基本的にスタック領域に格納される

```rust
fn main() {
    let x: i32 = 100; 

    // mutで変更可能。型は推論してくれるので明示しなくてもOK
    let mut y = 50; 

    // yは値を変更可能
    y = 300; 

    println!("x: {}, y: {}", x, y);
}
```

immutableな変数を変更すると、コンパイルエラーがこの様な感じで出ます
``` rust
fn main() {
  let x = 100;
  x = 150;  // ここでコンパイルエラー
  println!("{}", x);
}
```

### 基本的な型一覧

| 型の種別 | 型の種類 |
| --- | --- |
| ブール型 | bool |
| 符号なし整数型 | u8, u32, u64, u128 |
| 符号付き整数型 | i8, i32, i64, i128 |
| ポインタサイズ整数型 | usize, isize |
| 浮動小数点数型 | f32, f64 |

構造体や列挙型で定義されるのも型

参考：[[Rust] isize、usizeとは - Qiita](https://qiita.com/osorezugoing/items/23940e2507ae6149f12d)

