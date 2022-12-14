# 動かしてみる

以下のコマンドでサーバーを起動

```bash
$ cargo make --env-file .env run
```

HTTPリクエストを投げると、APIからは投稿した内容が返ってくる

```bash
$ curl -X POST 'http://localhost:8080/posts' -H 'Content-Type: application/json' -d \
'{"title": "こんにちは", "body": "初投稿です"}'

{"id": 1, "title": "こんにちは", "body": "初投稿です", "published": false }
```

sqlite3コマンドを使える場合は、selectクエリを実行して確認

```bash
$ sqlite3 posts.db

sqlite> select * from posts;

1|こんにちは|初投稿です|f
```