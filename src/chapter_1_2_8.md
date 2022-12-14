# Option型、Result型

Rustでは`null`とか`例外機構`も型で表現される

## `Option型`

存在しない可能性のある値

存在しない時に`None`、存在する時に`Some(T)`を返す

値を取り出すにはパターンマッチでの検証が必要なため、**変数に唐突にnullが入ってクラッシュすることがない**

```rust
enum Option<T> {
	None,
	Some(T),
}

以下のような値を持つことができる

Option<i32>ならSome(10)
Option<Color>ならSome(Color::Custom(10,123,255))
```

## Result型
処理成功時にOk、失敗時にErrを返す型

非同期処理の様に失敗する可能性のある処理の返り値として用いる

```rust
enum Result<T, E> {
	Ok(T),
	Err(E),
}
```