# 列挙型
`enum`キーワードを使い「複数の異なる要素のうちの一つ」のような構造を定義する

列挙子は`::`で指定する

構造体の定義を満たせばどのような値でも持てる

```rust
enum Color {
    Red,
    Green,
    Blue,
    Custom(u8, u8, u8),
}

impl Color {
    fn color_code(&self) -> String {
        match *self {
            Color::Red => String::from("#ff0000"),
            Color::Green => String::from("#00ff00"),
            Color::Blue => String::from("#0000ff"),
            Color::Custom(r, g, b) => {
                format!(
                    // 2桁の16進数での出力指定
                    "#{:02x}{:02x}{:02x}",
                    r, g, b
                )
            }
        }
    }
}

fn main() {
    let color = Color::Blue;
    println!("{}", color.color_code()); // #0000ff

    let color = Color::Custom(10, 123, 255);
    println!("{}", color.color_code()); // #0a7bff
}
```