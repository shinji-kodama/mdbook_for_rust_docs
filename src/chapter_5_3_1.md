# Diesel

## `Diesel`？

SQLite3, PostgreSQL, MySQLに対応

フィーチャで使うRDBMSを指定する

### Dieselの準備

新規プロジェクトを作成

```bash
$ cargo new actix-web-blog

$ cd actix-web-blog
```

### Diesel CLIをインストール

featuresに`sqlite-bundled`を指定するとSQLite3をインストール不要で利用可能

```bash
$ cargo install diesel_cli --no-default-features --features sqlite-bundled
```

### 環境変数を用意
.envファイルをプロジェクト直下に作成し、マイグレーションやアプリケーションからの接続時に使う環境変数DATABASE_URLに記述

```toml
// .env
DATABASE_URL=posts.db
```

### DBテーブルを作成

```bash
$ diesel setup
# posts.db, diesel.toml, migrationsディレクトリが作成される

$ diesel migration generate create_posts
# migrationsディレクトリにからのSQLファイルが作成される
```

## 現状のディレクトリ構成

```

.
├── .env
├── Cargo.toml
├── diesel.toml
├── migrations
│   └── 2022-12-12-174044_create_posts
│       ├── down.sql
│       └── up.sql
├── post.db
└── src
		└── main.rs

```

### uq.sqlとdown.sqlを編集
- uq.sql
    
  テーブル作成処理を記述
  
  記事のid, 記事タイトル, 記事本文, 公開済みかどうかのフィールドを持たせる
  
  ```sql
  CREATE TABLE IF NOT EXISTS posts (
      id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
      title VARCHAR NOT NULL,
      body TEXT NOT NULL,
      published BOOLEAN NOT NULL DEFAULT 'f'
  );
  ```
  

- down.sql

  テーブル削除処理を記述
    
    ```sql
    DROP TABLE posts;
    ```
        

### マイグレーション
以下のコマンドを実行

```bash
$ diesel migration run
```

`src/schema.rs`が生成される

```rust
// @generated automatically by Diesel CLI.

diesel::table! {
    posts (id) {
        id -> Integer,
        title -> Text,
        body -> Text,
        published -> Bool,
    }
}
```

cargo-make用の実行コマンドMakefile.tomlをプロジェクト直下に作成

clippyとtestのwatchタスクは前回と一緒

runタスクを追加

```toml
# Makefile.toml

[tasks.watch]
run_task = [
    { name = ["clippy", "test"] }
]
watch = true

[tasks.clippy]
command = "cargo"
args = ["clippy"]

[tasks.test]
command = "cargo"
args = ["test"]

[tasks.run]
command = "cargo"
args = ["run"]
```

makefileに定義したtasks.runから実行

`—-env-file`オプションで指定したファイルの内容を実行時の環境変数に設定できる

```bash
$ cargo make --env-file .env run
```

sqlite3コマンドがある場合は、post.dbを覗くとこうなる

```bash
$ sqlite3 posts.db
sqlite> .schema posts
CREATE TABLE IF NOT EXISTS posts (
    id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
    title VARCHAR NOT NULL,
    body TEXT NOT NULL,
    published BOOLEAN NOT NULL DEFAULT 'f'
);
```