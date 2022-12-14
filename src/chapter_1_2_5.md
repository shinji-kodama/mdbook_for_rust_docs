# 構造体
`struct`キーワードを使うデータ型

以下の３つの構造体がある

- 名前付きのフィールドを持つ形
- 名前を付けずに複数の値を持つ形（タプル構造体）
- 値を持たない構造体（ユニット構造体）

`impl`を使ってメソッドを定義できる

インスタンスメソッド：`&self`のような自身を指す引数を持つ

関連関数：`&self`なし（所謂スタティックメソッド。`::`で呼び出す）

```rust
struct Fruit {
    name: String,
}

impl Fruit {
    fn get_name(&self) -> &str {
        &self.name
    }
}

struct Rectangle(i32, i32);

impl Rectangle {
    fn calc_area(&self) -> i32 {
        self.0 * self.1 
    }
}


struct Unit; 

fn main() {
    // 定義した構造体の利用
    let banana = Fruit {
        name: String::from("Banana"),
    };
    println!("{}", banana.get_name()); // Banana

    let rect = Rectangle(10, 20);
    println!("{}", rect.calc_area()); // 200

  
    let _unit = Unit;
}
```