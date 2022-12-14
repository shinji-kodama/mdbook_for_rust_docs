# 関数、クロージャ
## 関数

`fn  fuction_name (arg: T) -> T { }`の様に作成

- （）内の引数の型は省略不可 

- `->`の右側に返り値の型を記述（省略可）

- 返り値は`return`は使わず、セミコロン省略で記述するのが慣例

- 戻り値の型が指定されていない場合、空のタプル`()`を返す

```rust

fn main() {
}


```

## クロージャ

- 要は無名関数とかlambda式

- `||`で囲った中に引数を記述し、その右に関数内の処理を記述

- 関数内の処理が１つの式で完結する場合は`{}`を省略可能

```rust
// 関数の定義(d1)
fn add(x: i32, y: i32) -> i32 {
    // returnと;を省略して記述（return x + y; でも別にok）
   x + y
}

// 戻り値は () と推論
fn make_nothing() {
    // この関数は戻り値が指定されないため () を返す
}

fn main() {
    // 関数の呼び出し(d2)
    let added = add(10, 20);
    println!("added: {}", added);
    // => 30


    let nothing = make_nothing();

    // 空を表示するのは難しいので、
    // a のデバッグ文字列を表示
    println!("nothing: {:?}", nothing);

    // クロージャ(d3)
    let z = 20;
    let add_z = |x: i32| x + z;
    println!("add_z: {}", add_z(10));
    // => 30
}
```