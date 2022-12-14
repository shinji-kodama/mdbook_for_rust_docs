# プロジェクトの作成
## 実行可能プロジェクト作成
first-projectという名前の実行可能プロジェクトを生成

```bash
$ cargo new first-project
```

生成される

```bash
first-project
├── Cargo.toml
└── src
    └── main.rs
```

## ライブラリプロジェクト作成
`--lib`をつけると実行可能プログラムではなくライブラリ開発のプロジェクトが生成される

```bash
$ cargo new --lib first-library
```

生成される `main.rs`が`lib.rs`に変化

```
first-library
├── Cargo.toml
└── src
    └── lib.rs
```