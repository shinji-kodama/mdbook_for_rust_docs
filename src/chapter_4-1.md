# 環境構築

`aws-lambda-rust-runtime`と`cargo-lambda`を使って簡単なプログラムを`Lambda`にデプロイする準備をしていく

## cargo-lambdaをインストール

cargo-lambda
[Getting Started | Cargo Lambda](https://www.cargo-lambda.info/guide/getting-started.html)

- dockerの場合
    
    インストールまでしてくれる環境を作りました（さっきまでと同じリポジトリ）
    
    [GitHub shinji-kodama/rust_practice](https://github.com/shinji-kodama/rust_practice)
    
- macの場合
    
    ```bash
    $ brew tap cargo-lambda/cargo-lambda
    $ brew install cargo-lambda
    ```
    

- pipとかでも可
    
    ```bash
    $ pip3 install cargo-lambda
    ```