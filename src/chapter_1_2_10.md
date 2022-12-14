# 繰り返し処理

- `for`文では `0..10`みたいな書き方ができる

- `while`文は条件式がtrueである限り歯繰り返す

- `loop`文は条件が常にtrueなwhile文であるため中断処理が必須

```rust
fn main() {
    for i in 0..10 {
        println!("in for-loop: {}", i);
    }

    let mut count = 0;
    while count < 10 {
        count += 1;     // count++ のような表記はできない
    }

    loop {
        count -= 1;
        if count == 0 {
            break;
        }
    }
}
```