# PythonとRustモジュール速度比較
## フィボナッチ数列の値を再起的に求める関数で比較する
フィボナッチ数列： 1 1 2 3 5 8 13 21 34 ・・・

### Rust側の実装
    
`lib.rs`に`fib`メソッドと`pyo_example`内にfib用の`m.add_function`を追記
    
```rust

#[pyfunction]
fn sum_as_string(a: usize, b: usize) -> PyResult<String> {
    Ok((a + b).to_string())
}

// この関数を追記
#[pyfunction]
fn fib(a: u64) -> PyResult<u64> {
    if a <= 2 {
        return Ok(1);
    }
    Ok(fib(a - 2)? + fib(a - 1)?)
}

/// A Python module implemented in Rust.
#[pymodule]
fn pyo3_example(_py: Python, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(sum_as_string, m)?)?;
    m.add_function(wrap_pyfunction!(fib, m)?)?;  // 追記
    Ok(())
}

```

### Python側の実装
    
`fib.py`という名前でプロジェクト直下に以下を作成
    

```python
import pyo3_example
from time import time

def fib(a):
    if a <= 2:
        return 1
    return fib(a - 2) + fib(a - 1)

# pythonでの算出
start = time()
_ = fib(40)
print("python: ", time() - start, "sec")

# rustでの算出
start = time()
_ = pyo3_example.fib(40)
print("rust  : ", time() - start, "sec")
```

### 実行

`fib.py`を実行すると、pythonとrustで書いたfib関数が実行される

速度が10倍程度違うのがわかる。

```python
$ maturin develop  
$ python3 fib.py

python:  9.516207695007324 sec
rust  :  0.9243161678314209 sec

（実際に私のdocker環境で実行してこれくらいの速度です）
```