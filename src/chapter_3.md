# Rustで作ったモジュールをPythonで動かす

既存システムにRustを組み込む場合、全体のリプレイスは現実的ではないため、言語バインディングを用いたRustモジュールの追加やFaas等を使ったAPIの一部の置き換えになることが多い。

ここでは言語バインディングを使ってPython内でRustモジュールを実行する

## 言語バインディングを使ったRustモジュールの追加
既に何かの言語でシステムが稼働しており、そのシステム内に負荷の高い処理があるときは、Rustで実装した高速なライブラリを呼び出して使うのも選択肢の一つ

`FFI`（Foreign Functions Interface）を使うことで異なる言語同士のプログラムを連携可能

言語ごとにRustバインディング作成補助のクレートがあるので使うと実装が楽になる

| 言語 | クレート |
| --- | --- |
| Python | PyO3 |
| Ruby | rutie |
| PHP | ext-php-rs |
| JavaScript | neon |
| Erlang / Elixir | rustler |

### 今回は`PyO3`を使ってPythonからRustプログラムを呼び出してみる