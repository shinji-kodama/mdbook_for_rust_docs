# Tauriを用いたクロスプラットフォーム開発

## Tauriについて
2022年6月にelectron代替として登場したフレームワーク。フロントをJSやTSのフレームワーク（Next, vite, svelte, etc）で記述し、バックエンドにRustを用いている。

Windows, MacOS, Linuxで動くアプリケーションを開発可能

2022年12月9日 Tauriのmobile対応を発表し、現在α版公開中


Tauri公式はこちら

[Build smaller, faster, and more secure desktop applications with a web frontend | Tauri Apps](https://tauri.app/)

alpha版がリリースされたtauri mobileについて

[Mobile | Tauri Apps](https://next--tauri.netlify.app/next/mobile/)

## インストール
macの場合

### CLang and macOS Development Dependencies[](https://tauri.app/v1/guides/getting-started/prerequisites#1-clang-and-macos-development-dependencies)

CLangとmacOSの開発用依存ファイルをインストールする必要があります。これを行うには、ターミナルで以下のコマンドを実行します。

```bash
xcode-select --install
```

### Rustをインストール[](https://tauri.app/v1/guides/getting-started/prerequisites#2-rust)

macOSにRustをインストールするには、ターミナルを開いて、次のコマンドを入力します。

```bash
curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

### create tauri app

tauriのプロジェクトを作成

```bash
$ cargo install create-tauri-app
$ cargo create-tauri-app
```

### htmlでUIを作る

htmlで書く場合は`ui`フォルダを作り、中にindex.hdmlを作成

```bash
$ cd tauri-app
$ mkdir ui
$ cd ui
$ vim ui/index.html
```

index.htmlの中身

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <h1>Welcome from Tauri!</h1>
  </body>
</html>
```

### tauri-cliをインストール

tauri-cliをインストール（installコマンドなので`$HOME/.cargo/bin`にインストールされる）

```bash
$ cargo install tauri-cli
```

最小限のTauriを使用したRustプロジェクトを作る

以下のコマンドを使用

```bash
$ cargo tauri init
```

以下の質問に答える

> 
> 
> 1. *What is your app name?*
>     
>     最終的なバンドルの名前になり、OS があなたのアプリを呼ぶときの名前になります。ここでは好きな名前を使うことができます。
>     
> 2. *What should the window title be?*
>     
>     デフォルトのメインウィンドウのタイトルになります。ここでは好きなタイトルを使うことができます。
>     
> 3. *Where are your web assets (HTML/CSS/JS) located relative to the `<current dir>/src-tauri/tauri.conf.json` file that will be created?*
>     
>     production用にビルドする際に、Tauriがフロントエンドのアセットをロードするパスです。
>     
>      `../ui` を使用
>     
> 4. *What is the URL of your dev server?*
>     
>     開発中にTauriがロードするURLまたはファイルパスのいずれかになります。
>     
>     `../ui` を使用
>     
> 5. *What is your frontend dev command?*
>     
>     フロントエンド開発サーバーを起動するためのコマンド
>     
>     今回は何もコンパイルする必要がないので空白
>     
> 6. *What is your frontend build command?*
>     
>     フロントエンドのファイルをビルドするためのコマンド
>     
>     何もコンパイルする必要がないので、ここも空白
>     

`src-tauri`フォルダが生成される（create-tauri-appでできてた？）

Tauriアプリケーションでは、コアに関連するすべてのファイルをこのフォルダに置くことが慣例となってる

src-tauriの中身は以下の通り

```
.
├── src-tauri
│   ├── Cargo.toml
│   ├── build.rs
│   ├── icons
│   │   ├── 128x128.png
│   │   ... （アイコン沢山）
│   │   
│   ├── src
│   │   └── main.rs
│   │   　　プログラムのエントリーポイント（main関数が最初に呼ばれる）
│   │   
│   └── tauri.conf.json
				アプリ名から許可APIリストに至るまで、アプリを設定し、カスタマイズできる
```

### 実行

以下のコマンドでweb viewが開き、実行される

```
$ cargo tauri dev
```

### commandの呼び出し

TauriではJSからRust関数を呼び出せる（コマンド）

重い処理やOSへの呼び出しをパフォーマンスの高いRustで処理できる

簡単なコマンドの呼び出しを記述

`#[tauri::command]`属性マクロが追加されたRust関数を書く

JavaScriptコンテキストと通信できる

```rust
// src-tauri/src/main.rs

#[tauri::command]
fn greet(name: &str) -> String {
   format!("Hello, {}!", name)
}
```

Tauriに新しく記述したコマンドを伝え、呼び出しをルーティングする

`.invoke_handler()` と `generate_handler![]` マクロを使用

```rust
// src-tauri/src/main.rs

fn main() {
  tauri::Builder::default()
    .invoke_handler(tauri::generate_handler![greet]) // この行を追加
    .run(tauri::generate_context!())
    .expect("error while running tauri application");
}
```

これでフロントエンドからCommandを呼び出す準備ができた

通常は`@tauri-apps/api`パッケージを推奨しますが、今回はbundlerを使用しないため、`tour.conf.json`ファイルに`withGlobalTauri: true`を追記する

```json
// **tauri.conf.json**
{
  "build": {
    "beforeBuildCommand": "",
    "beforeDevCommand": "",
    "devPath": "../ui",
    "distDir": "../ui",
    "withGlobalTauri": true // この行を追加
  },
```

これで、htmlファイルからcommandを呼び出すことが可能になる

index.htmlに<script></script>を追加して、その中に以下を記述

```jsx
// index.htmlにscriptタグを追記してその中に記述

// バンドルされているグローバルAPI関数にアクセス
const { invoke } = window.__TAURI__.tauri

const button = document.createElement("button");

button.innerText = "Click me";
button.addEventListener("click", async () => {
	// invokeでgreet関数を呼び出して実行。返値はPromise
  const result = await invoke("greet", { name: "World" });
  console.log(result);
});
```

クリックするとconsoleのHello World!と出力される

![tauri_first_app_と_DeepL_と_講師_チューター_スタッフチャンネル_-_LAB-14th_G_s_ACADEMY_-_Slack.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f85ddefc-aa09-4666-b2da-1a6ac6b1c286/tauri_first_app_%E3%81%A8_DeepL_%E3%81%A8_%E8%AC%9B%E5%B8%AB_%E3%83%81%E3%83%A5%E3%83%BC%E3%82%BF%E3%83%BC_%E3%82%B9%E3%82%BF%E3%83%83%E3%83%95%E3%83%81%E3%83%A3%E3%83%B3%E3%83%8D%E3%83%AB_-_LAB-14th_G_s_ACADEMY_-_Slack.png)