{% raw %}

# 命名

Rust では、標準ライブラリが命名規約の事実上の標準となっている。
この規約を明文化する取り組みは、[RFC 43](https://rust-lang.github.io/rfcs/0430-finalizing-naming-conventions.html) で行われ、のちに [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/) でも行われた。

基本規則 [`(C-CASE)`] は、次のものを使うことである。

- 型、トレイト、列挙型のバリアント、ジェネリック型パラメータには `UpperCamelCase`
- 関数、メソッド、マクロ、変数、モジュールには `snake_case`
- 静的変数、定数、ジェネリック定数パラメータには `SCREAMING_SNAKE_CASE`
- ライフタイムには `'lowercase`

[Rust API Guidelines](https://rust-lang.github.io/api-guidelines/) は、特定の構文について、さらに細かい命名規約も
定めている。

- 変換メソッド（`as_`、`to_`、`into_`）については [`(C-CONV)`]
- ゲッターについては [`(C-GETTER)`]
- イテレータを生成するメソッドについては [`(C-ITER)`]
- イテレータ型については [`(C-ITER-TY)`]
- フィーチャ（条件付きで有効になる機能）の命名については [`(C-FEATURE)`]
- 語順の一貫性については [`(C-WORD-ORDER)`]

<div markdown="1" class="note">

基本規則 [`(C-CASE)`] は、コンパイラが（`nonstandard_style` のリント群によって）検査する。

コンパイラに加えて、[`clippy`](devenv.md#clippy) ツールも、`style` カテゴリのリントによって命名規約の採用を助けることができる。
たとえば、[`wrong_self_convention`](https://rust-lang.github.io/rust-clippy/master/index.html#wrong_self_convention) リントは、[`(C-CONV)`] に従って、変換メソッドの名前とそのレシーバの型（`self`、`&self`、`&mut self`）との整合性を検査する。

<!--
clippy::enum_variant_names
clippy::self_named_constructors
-->

</div>

<div markdown="1" class="reco" id="LANG-NAMING" type="Rule" title="命名規約を守る">

安全なアプリケーションの開発は、[Rust API Guidelines](https://rust-lang.github.io/api-guidelines/) に示されている
命名規約に従わなければならない。

</div>

[`(C-CASE)`]: https://rust-lang.github.io/api-guidelines/naming.html#casing-conforms-to-rfc-430-c-case
[`(C-CONV)`]: https://rust-lang.github.io/api-guidelines/naming.html#ad-hoc-conversions-follow-as_-to_-into_-conventions-c-conv
[`(C-GETTER)`]: https://rust-lang.github.io/api-guidelines/naming.html#getter-names-follow-rust-convention-c-getter
[`(C-ITER)`]: https://rust-lang.github.io/api-guidelines/naming.html#methods-on-collections-that-produce-iterators-follow-iter-iter_mut-into_iter-c-iter
[`(C-ITER-TY)`]: https://rust-lang.github.io/api-guidelines/naming.html#iterator-type-names-match-the-methods-that-produce-them-c-iter-ty
[`(C-FEATURE)`]: https://rust-lang.github.io/api-guidelines/naming.html#feature-names-are-free-of-placeholder-words-c-feature
[`(C-WORD-ORDER)`]: https://rust-lang.github.io/api-guidelines/naming.html#names-use-a-consistent-word-order-c-word-order
{% endraw %}
