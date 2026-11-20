{% raw %}

# エラー処理

<!-- <mark>TODO</mark>: explicit good practices in error handling. -->

失敗しうる関数を扱う方法としては、`Result` 型を使うのが望ましい。
`Result` オブジェクトは必ず検査しなければならず、決して無視してはならない。

<div markdown="1" class="reco" id="LANG-ERRWRAP" type="Recommendation" title="起こりうるすべてのエラーをラップする独自の `Error` 型を実装する">

クレートは、起こりうるすべてのエラーをラップする独自の `Error` 型を実装してもよい。
その際は、この型を例外安全（[RFC 1236](https://rust-lang.github.io/rfcs/1236-stabilize-catch-panic.html)）にし、
`Error + Send + Sync + 'static` と `Display` を実装するよう注意しなければならない。

</div>

エラー処理を楽にするために、サードパーティのクレートを使ってもよい。その多く
（特に [snafu] や [thiserror]）は、必要なトレイトを実装し、ほかのエラーを
ラップできる独自のエラー型を新たに作ることを目的としている。

もう一つの方法（特に [anyhow] クレートが提案しているもの）は、エラーを単一の
汎用エラー型へ自動的にラップするものである。このようなラッパーは、ラップした
エラーに開発者が文脈を与えられないため、ライブラリや複雑なシステムでは
使うべきでない。

[snafu]: https://crates.io/crates/snafu
[thiserror]: https://crates.io/crates/thiserror
[anyhow]: https://crates.io/crates/anyhow

## パニック

一般に、`panic` を呼び出すよりも、明示的なエラー処理（`Result`）を優先すべきである。
エラーの原因が分かるようにすべきであり、汎用的なエラーは
避けるべきである。

<div markdown="1" class="reco" id="LANG-LIMIT-PANIC" type="Rule" title="`panic` の使用を制限する">

Rust の関数は、その使用条件に違反した場合を除き、パニックしてはならない。

</div>

ライブラリを提供するクレートは、失敗してコードをパニックさせうる関数や命令を
決して使うべきでない。

次のパターンは（設計上）パニックを引き起こしうる。

- `unwrap` または `expect` の使用
- `assert!` の使用

次の規則は、[前の規則](#LANG-LIMIT-PANIC)から導かれる。


<div markdown="1" class="reco" id="LANG-LIMIT-PANIC-SRC" type="Rule" title="パニックする関数の使用を制限する">

`unwrap`、`expect`、`assert!` の使用は、関数の仕様が明示的に禁じている場合に限定しなければならない。

</div>

次の操作は、引数が使用条件を満たさない場合にパニックすることが知られている。

- 検査なしの配列アクセス（[次の](#LANG-ARRINDEXING)規則を参照）
- 整数オーバーフロー（デバッグモードの場合。[整数](integer.md#chapter-integer)の章を参照）
- ゼロ除算
- 大きなメモリ割り当て
- `format!` を使った文字列の書式化

<div markdown="1" class="reco" id="LANG-ARRINDEXING" type="Rule" title="配列の添字アクセスを適切に検査するか、`get` メソッドを使う">

配列の添字アクセスは適切に検査しなければならない。または、`Option` を返す
`get` メソッドを使うべきである。

</div>

<div markdown="1" class="warning">

安全性が重視される（セーフティクリティカルな）一部の分野では、放置すれば未定義動作につながりうるエラーが発生した場合、必ずセーフモード状態へ移行することが義務づけられている。
このような状況では、実行を意図的に中断することは理にかなっている。破損したデータや安全上の障害が広がる前にシステムを止められるからである。

航空機やその他の乗り物では、この「フェイルファスト（早期に失敗させる）」動作が極めて重要になりうる。主制御装置は、重大な障害が起きたら直ちに停止し、冗長系またはバックアップのサブシステムに制御を引き渡さなければならない。引き継いだサブシステムは、機体を安全に停止させるか、機能を縮退させたモードで運用を続ける。信頼できる二次系で再起動することで、航空機を制御可能な状態に保ち、搭乗者を守り、予測できない状態のまま実行を続けることで生じうる危険な結果を防げる。

この用途では、Cargo.toml ファイルの `[profile.release]` セクションで `panic = 'abort'` 属性を有効にすると、パニックが発生した時点でプログラムが直ちに終了する。

</div>

<!--
<mark>TODO</mark> Check if the [no_panic](https://github.com/dtolnay/no-panic)
crate can catch all cases. Drawback: all functions need to be marked as
`#[no_panic]`.
-->
<!--
<mark>TODO</mark> Another possibility:
[rustig](https://github.com/Technolution/rustig) (doesn't build here)
-->

## FFI とパニック

Rust へのほかの言語の統合は、*FFI*（*Foreign Function Interface*）を通じて行う。
この機能については、[専用の章](unsafe/ffi.md#ffi-panic)で詳しく説明する。
{% endraw %}
