# タスクの自動実行
`cargo-make`：clippyやtestのコマンド実行をまとめて自動化するのに便利なタスクランナー

CとかC++のmakeコマンドみたいなものらしい

### インストール

下のコマンドで`cargo-make`をプロジェクトにインストール

```bash
$ cargo install cargo-make
```

### タスクを記述

プロジェクトルート直下に`Makefile.toml`を作成

以下を記述して`cargo make watch`を実行すると

コードを変更して保存するたびにclippyとtestを実行できる

```toml
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
```

### 実行

以下のコマンドでMakefile.tomlの内容を実行する

```bash
$ cargo make watch

# cargo makers watch でも同様の動きをする
# （cargo-makeインストール時にmakersコマンドも同時にインストールされている）
```

参考：

クレートをカテゴリごとに探せる便利なサイト

[Lib.rs](https://lib.rs/)