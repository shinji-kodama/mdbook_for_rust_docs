# 文字列、配列、ベクタ

## 文字列
    
### &str型

`let str = “world”`のように直接記述できる。固定サイズで変更不可能で、実体はu8のスライス。

### String型

`let str = String::from("world")`のように作成。値を変更可能で、実体はu8のベクタ。

`&str`と`String`直接比較可能


### 配列

`let array: [i32; 3] = [1,2,3]` の様に記述。要素はスタック領域に格納される。固定長。


![label](https://camo.qiitausercontent.com/72432d2421e08ce8fcea25f765df62e105c27d29/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f323639333335302f37333235333633382d633738662d373565632d646662622d3335386234313465623234332e706e67)

    
### ベクタ
    
`let vector: Vec<i32> = vec![1, 2, 3]`とマクロを使って作成。要素はヒープ領域に格納されるため可変長。ヒープへのポインタや要素数・容量がスタックに格納される。

![label](https://camo.qiitausercontent.com/e3766b26d1c669bd634c8b508afe2f9d465432c3/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f323639333335302f63316565636532372d636264332d663633342d343137362d3534356564353938326634652e706e67)
可変長

`vec![1,2,3]`

## スライス
スライスは、配列やベクタへの参照

```rust
fn main() {
    // 文字列は、大きく&str型とString型がある
    let str_slice: &str = "world";

    let _string: String = String::from(str_slice); // _で束縛後の変数を使用しないことを明示

    let string_format: String = format!("Hello {}", str_slice); // 文字列の合成によく使う


    println!("{}", string_format);


    // 要素数3の配列（固定長配列）
    let mut array: [i32; 3] = [1, 2, 3];
    
    array[0] = 10; // mutならば要素の値を変更できる

    // array.push(10); // mutでも要素の追加はできない

    // 要素数3のベクタ（可変長配列）
    let mut vec: Vec<i32> = vec![1, 2, 3]; 
    
    vec[0] = 10;    // mutならば要素の値を変更できる
    
    vec.push(10);   // mutならば要素を追加できる


    println!("array: {:?}, vec: {:?}", array, vec)
}
```

配列・ベクタ・スライスについて

[Rustの配列やベクタ、スライスについて - Qiita](https://qiita.com/k-yanai60/items/26bf1d2e372042eff022)


