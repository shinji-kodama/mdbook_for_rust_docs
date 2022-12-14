# WebAssemblyで作るフロントエンド

Yewを使ってSPAのフロントエンドを作る

バックエンドには前章で作ったAPIを使用

### webassembly？

ブラウザ上で実行できるバイナリ軽視のアセンブリ風言語。Rustの他にC/C++, Go等の低水準プログラミング言語からのコンパイルを意図して設計されている。

(TypeScriptからwebassemblyにコンパイルする[AssemblyScrypt](https://www.assemblyscript.org/)なんてものもある)

映像処理等のネイティブ性能に近いパフォーマンスが要求される領域での利用が想定されている（ffmpegとかのwasm化プロジェクトはあるらしい）

### Yew ?

WebAssembleでSPAを実現しよう！というRustのフレームワーク。React風の関数コンポーネントとHooksが使える。

[Tutorial | Yew](https://yew.rs/ja/docs/tutorial)
