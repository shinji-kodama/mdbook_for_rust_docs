# Rustのインストール

Rustツールチェインをインストール（[公式より](https://www.rust-lang.org/ja/tools/install)）

```bash
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

アップデートする場合はこれ

```bash
$ rustup update
```

`~/.cargo/bin`にPATHを通す

結果はこんなん

```bash
$ cargo --version
cargo 1.65.0 (4bc8f24d3 2022-10-20)

$ rustc --version
rustc 1.65.0 (897e37553 2022-11-02)

$ rustup --version
rustup 1.25.1 (bb60b1e89 2022-07-12)

```