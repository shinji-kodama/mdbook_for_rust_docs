# Hello world する

### ベースになるhtmlを作成
とりあえずプロジェクト直下にindex.htmlを作成

```html
<!-- index.html -->

<!DOCTYPE html>
<html lang="en">
	<head> </head>
	<body></body>
</html>
```

main.rsを編集して Hello worldと表示するコンポーネントを作成

```rust
// main.rs

use yew::prelude::*;

// Reactっぽい！！
#[function_component(App)]
fn app() -> Html {
    html! {
        <h1>{ "Hello World" }</h1>
    }
}

fn main() {
    yew::Renderer::<App>::new().render();
}
```

### 現状のフォルダ構成

```
.
├── Cargo.lock
├── Cargo.toml
├── index.html
└── src
    └── main.rs
```
### サーバー起動
trunkコマンドでビルド + 起動

```bash
$ trunk serve --port 8081
```

### ブラウザで表示
localhost:8081でWASMが動いた！！

![localhost_8081.png](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/08a6d7f6-a3a0-444e-a04c-f82277eb028d/localhost_8081.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221214%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221214T041754Z&X-Amz-Expires=86400&X-Amz-Signature=1817a9c5f324a19410cf8d55fec30952373fca77d3899dadd820553b359247bd&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22localhost_8081.png%22&x-id=GetObject)

参考までに、curlで叩くとこんな感じで帰ってきます

```html
<html>
<head>
	<link rel="preload" href="/yew-blog-12aa93e5f87ed4e9_bg.wasm" as="fetch" type="application/wasm" crossorigin="">
	<link rel="modulepreload" href="/yew-blog-12aa93e5f87ed4e9.js">
</head>
<body>
<script type="module">import init from '/yew-blog-12aa93e5f87ed4e9.js';init('/yew-blog-12aa93e5f87ed4e9_bg.wasm');</script>
<script>
(function () {
    var protocol = window.location.protocol === 'https:' ? 'wss:' : 'ws:';
    var url = protocol + '//' + window.location.host + '/_trunk/ws';
    var poll_interval = 5000;
    var reload_upon_connect = () => {
        window.setTimeout(
            () => {
                // when we successfully reconnect, we'll force a
                // reload (since we presumably lost connection to
                // trunk due to it being killed, so it will have
                // rebuilt on restart)
                var ws = new WebSocket(url);
                ws.onopen = () => window.location.reload();
                ws.onclose = reload_upon_connect;
            },
            poll_interval);
    };

    var ws = new WebSocket(url);
    ws.onmessage = (ev) => {
        const msg = JSON.parse(ev.data);
        if (msg.reload) {
            window.location.reload();
        }
    };
    ws.onclose = reload_upon_connect;
})()
</script>
</body>
</html>
```