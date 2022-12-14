# 環境構築

### WASM向けのビルド環境

rustupコマンドでターゲットを追加した上でWASMをホストする開発サーバーtrunkをインストール

（docker環境には導入済）

```bash
$ cargo install trunk
$ rustup target add wasm32-unknown-unknown
```

### プロジェクトを作成し、クレートを追加
Yewでのフロントエンドを実装していくためのプロジェクトを作成

使うクレート（`yew`, `wasm-bindgen-futures`, `gloo-net`, `serde`, `chrono`）も追加する

`yew`, `wasm-bindgen-futures`, `gloo-net`を併せて使うことで、外部サーバーへのHTTP通信を非同期で実装可能

```sh
$ cargo new yew-blog
$ cd yew-blog
$ cargo add yew -F csr
$ cargo add wasm-bindgen-futures gloo-net
$ cargo add serde -F derive
$ cargo add chrono -F serde
```