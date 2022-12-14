# 実装の概要
### 概要

実装内容

1. 記事を投稿
    
    記事タイトルと本文を含むJSONをPOSTでうけとりDBに保存
    
2. 記事データの一覧を取得
    
    GETを受けてすべての記事リストをJSONで返す
    
3. 指定したIDの記事を取得
    
    記事IDを指定したGETを受け該当記事のJSONを返す
    

実装の流れ

1. 依存クレートの追加
2. 実装するファイルの準備
3. 実装を始める
    1. DBとの接続処理の作成
    2. エラー定義の作成
    3. DBへの入出力処理の作成
    4. APIの作成
    5. ミドルウェアの追加

### クレートの追加

以下のクレートを追加

既出：`actix-web`, `serde`, `diesel`

SQLite3同梱の接続用クレート：`libsqlite3-sys`

エラーを簡単に扱うためのクレート：`anyhow`, `thiserror`

```bash
$ cargo add actix-web
$ cargo add serde -F derive
$ cargo add diesel -F r2d2 -F returning_clauses_for_sqlite_3_35 -F sqlite
$ cargo add libsqlite3-sys -F bundled
$ cargo add anyhow thiserror
```

実装用のファイルを準備

```bash
src
├── error.rs  # 追加
├── main.rs
├── repository.rs # 追加
└── schema.rs
```

main.rsに以下を記述

```rust
// main.rs

mod error;
mod repository;
mod schema;
```