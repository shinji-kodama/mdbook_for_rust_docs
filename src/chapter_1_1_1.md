# パフォーマンス
以下の特徴により、高速に動作する言語

組み込み機器上で実行したり他の言語との調和も簡単にできる

これまで他の言語で作られていたツールのRust実装版が作られたりしている （ex. `grep -> ripgrep` , `ls -> exa`, `cat -> bat`, `find -> fd` etc）


- コンパイル型言語

    コンパイル結果は実行可能バイナリであるためランタイム環境不要
    
- ガベージコレクションを持たない

    コンパイル時にメモリの使い方を決定するためメモリ効率が良い
        
- ゼロコスト抽象化

    型や関数でコードを抽象化してもコンパイル時に解決され、実行時に追加のコストがかからない
    
    （メモリ使用量の増加や実行速度の低下などの、所謂オーバーヘッドがなくなる）
    
    参考：[Rustのゼロコスト抽象化の効果をアセンブラで確認](https://blog.rust-jp.rs/tatsuya6502/posts/2019-12-zero-cost-abstraction/)