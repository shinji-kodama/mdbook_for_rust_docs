# PyO3でPythonからRustを呼び出す
## `PyO3`
PythonとRustの相互運用をサポートするクレート
([公式](https://pyo3.rs/))

Pythonから呼び出せるライブラリを作成するには、`maturin`が便利


```bash
$ cargo install maturin # dockerの場合はコンテナにインストール済み
$ maturin new pyo3-example --bindings pyo3
$ cd pyo3-example
```

lib.rsにsum_as_stringメソッドが定義された状態でファイル一式が作成される

lib.rsの中身

```rust
use pyo3::prelude::*;

/// Formats the sum of two numbers as string.
#[pyfunction]
fn sum_as_string(a: usize, b: usize) -> PyResult<String> {
    Ok((a + b).to_string())
}

/// A Python module implemented in Rust.
#[pymodule]
fn pyo3_example(_py: Python, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(sum_as_string, m)?)?;
    Ok(())
}
```

pythonを実行して、対話モードで`sum_as_string`を実行

```bash
$ python3 -m venv .env # 仮想環境作る（venvとかcondaで仮想環境に入らないとmaturin developが失敗する）
$ source .env/bin/activate
$ maturin develop  #ここでRustのコードをコンパイルしてる
$ python3
>>> import pyo3_example
>>> pyo3_example.sum_as_string(10, 20)
'30'
```

Rustのsum_as_stringメソッドで足し算ができた！