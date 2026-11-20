{% raw %}

# `unsafe` に関する一般事項

## *unsafe* な操作

unsafe なコードを使うと、言語の機能を拡張できる。これらの機能の完全な一覧は [Rust reference](https://doc.rust-lang.org/reference/unsafety.html) に掲載されている。
こうした機能はシステムプログラミングで必要になることがあるが、言語の[安全性に関する性質](../guarantees.md#language-guarantees)が失われ、未定義動作が起こりうるようになる。

<div markdown="1" class="reco" id="UNSAFE-NOUB" type="Rule" title="未定義動作の禁止">

未定義動作は一切許されない。

</div>

## 二つの用途を持つキーワード

[Rust Reference](https://doc.rust-lang.org/stable/reference/) で説明されているとおり、`unsafe` キーワードは、API が安全でないことを示す（マーキング）ためにも、実装の中で安全でない機能を解禁する（アンロック）ためにも使われる。

### `unsafe` によるマーキング

`unsafe` によるマーキングは、メモリ安全性に関する責任を API の作者から API の利用者へ委ねるものである。
API でこのキーワードを使うことは、その API の利用が有害な影響を及ぼしうることを API の利用者に*警告*する。

* 関数シグネチャ（[Rust Reference](https://doc.rust-lang.org/stable/reference/) の [`r-unsafe.fn`] 節）では、`unsafe` は、関数の使い方がそのインターフェース契約（ドキュメントに非形式的に記述されるもの）に従っていない場合、関数の動作が UB につながりうることを意味する。
* トレイト宣言（[Rust Reference](https://doc.rust-lang.org/stable/reference/) の [`r-unsafe.trait`] 節）では、`unsafe` は、実装契約（文書化されていることが望ましい）が守られていない場合、このトレイトの誤った実装が UB につながりうることを意味する。

[`r-unsafe.fn`]: <https://doc.rust-lang.org/reference/unsafe-keyword.html#r-unsafe.fn>
[`r-unsafe.trait`]: <https://doc.rust-lang.org/reference/unsafe-keyword.html#r-unsafe.trait>

### `unsafe` によるアンロック

`unsafe` によるアンロックは、メモリ安全性に関する責任をコンパイラから開発者が引き受けることを意味する。

関数本体や定数宣言の中での `unsafe` ブロックの使用は、次のような `unsafe` な機能を*うっかり使ってしまう*ことを防ぐために、コンパイラが強制する（[Rust Reference](https://doc.rust-lang.org/stable/reference/) の [`r-unsafe.block`] 節）。

* `unsafe` が付いた関数の使用
* 静的変数の変更
* `extern` 関数の使用

同様に、`unsafe` トレイトを実装するには `unsafe` が必要である（[Rust Reference](https://doc.rust-lang.org/stable/reference/) の [`r-unsafe.impl`] 節）。これは、開発者がメモリ安全性に関する契約を明示的に考慮するようにするためである。`unsafe` キーワードは、`unsafe` トレイトの実装を*アンロック*する。

最後に、2024 エディション以降は、次のものをアンロックする場合にも `unsafe` が必要である。

* `extern` ブロック（[Rust Reference](https://doc.rust-lang.org/stable/reference/) の [`r-unsafe.extern`] 節）。[FFI](./ffi.md#chapter-ffi) のための、外部の関数や変数の宣言を含む。
* 一部の属性（例えば no_mangle。[Rust Reference](https://doc.rust-lang.org/stable/reference/) の [`r-attributes.safety`] を参照）。

[`r-unsafe.impl`]: <https://doc.rust-lang.org/reference/unsafe-keyword.html#r-unsafe.impl>
[`r-unsafe.block`]: <https://doc.rust-lang.org/reference/unsafe-keyword.html#r-unsafe.block>
[`r-unsafe.extern`]: <https://doc.rust-lang.org/reference/unsafe-keyword.html#r-unsafe.extern>
[`r-attributes.safety`]: <https://doc.rust-lang.org/reference/attributes.html#r-attributes.safety>

## 制約と注意事項

[Rustonomicon](https://doc.rust-lang.org/nomicon/) の言葉を借りれば、Rust の基本原則は次のようにまとめられる。

<div markdown="1" class="important">

`unsafe` を使わないコードは、おかしな動作をしない

</div>

型システムと所有権システムを組み合わせて使うことで、Rust プログラムには高い水準のメモリ安全性が強制される。これにより、この言語は、メモリのオーバーフロー、ヌルポインタや不正なポインタの生成、データ競合の防止に役立つ。

この約束が成り立つのは、コードが `unsafe` な機能を使っていない場合だけである。`unsafe` な機能を使うと、コンパイラはもはやメモリ安全性を保証できない。その場合は、メモリ安全性を保証する不変条件をコードが守っていることを、開発者が確かめなければならない。

そのため、`unsafe` な機能の使用をできる限り制限することが極めて重要である。

<div markdown="1" class="reco" id="LANG-UNSAFE" type="Rule" title="unsafe ブロックを使わない">

安全な Rust 開発では、`unsafe` ブロックは避けるべきである。使う場合は、次の点の少なくとも一つによって正当化しなければならない。

* Rust の FFI（Foreign Function Interface）では、`extern "C"` という接頭辞を使って、C で実装された関数を記述できる。このような関数を使うには `unsafe` キーワードが必要である。C のコードを安全かつ違和感なく呼び出せるように、「安全な」ラッパーを定義するものとする。

* 組み込み機器のプログラミングでは、レジスタやその他のさまざまなリソースに、固定のメモリアドレスを通じてアクセスすることが多い。この場合、Rust でそうした特定のポインタを初期化し参照外しするには、`unsafe` ブロックが必要である。コード中の unsafe なアクセスの数を最小限に抑え、プログラマがそれらを見つけやすくするために、適切な抽象化（データ
構造またはモジュール）を提供するものとする。

* コードのごく一部で性能の壁に突き当たった場合（例: その場で書き換えるゼロコピーバッファ、メモリ割り当てのオーバーヘッドなど）。

これらの場合を除き、コードベースで `unsafe` が使われたときにコンパイルエラーになるよう、クレートのルート（典型的には `main.rs` または `lib.rs`）に `#![forbid(unsafe_code)]` を記述しなければならない。

</div>

`unsafe` の使用が必要な場合、次のことは開発者の責任である。

* `unsafe` によるアンロックの使用が *UB* につながらないことを保証する。
* 使用条件（不変条件）が守られていれば *UB* が起こりえないように、`unsafe` によるマーキングをすべて、正確かつ網羅的に文書化する。

`unsafe` なコードそのものに加えて、`unsafe` な機能の使用をコンポーネント（クレートまたはモジュール）の中に適切にカプセル化し、Rust 本来のメモリ安全性の保証を取り戻すことも極めて重要である。

<div markdown="1" class="reco" id="LANG-UNSAFE-ENCP" type="Rule" title="*unsafe* な機能のカプセル化">

Rust のソフトウェアコンポーネント（クレートまたはモジュール）の安全な開発では、
すべての *unsafe* なコードを、次のいずれかを満たすようにカプセル化しなければならない。

* 利用者に対して安全な動作を公開する。すなわち、安全な操作をどのように行っても
  UB（未定義動作）にならない。
* unsafe と明示した機能を公開し、その使用条件
  （事前条件、呼び出し順序など）を網羅的に文書化する。

</div>

したがって、`unsafe` な操作を使う関数であっても、コンポーネントの不変条件
（典型的には、メソッドであれば型の不変条件）のもとでその `unsafe` な操作が
*UB*（未定義動作）を起こさないのであれば、*安全*でありうる。逆に、
`unsafe` ブロックを含まない関数であっても、これらの不変条件を破るのであれば
`unsafe` と明示しなければならない。そのため、これらの不変条件を選び、
把握しておくことが、安全な開発には極めて重要である。

## 例: 型の不変条件を保つ

次のコードは [Rustonomicon](https://doc.rust-lang.org/nomicon/working-with-unsafe.html) から引用したものである。
独自の `Vec` 型を実装するのに使えるだろう。

```rust
use std::ptr;

pub struct Vec<T> {
    ptr: *mut T,
    len: usize,
    cap: usize,
}

// Note this implementation does not correctly handle zero-sized types.
impl<T> Vec<T> {
    pub fn push(&mut self, elem: T) {
        if self.len == self.cap {
            // reallocate new array with bigger capacity
        }
        unsafe {
            ptr::write(self.ptr.add(self.len), elem);
            self.len += 1;
        }
    }
}

```

このコードの健全性と安全性は、アドレス `self.ptr` から `self.ptr + self.cap * size_of<T>()` までのバイトが割り当て済みであるという事実に依存している。

この不変条件は、*安全な*コードで破ることができる。例えば次のとおりである。

<p class="codetag bad">悪い例</p>

```rust
impl<T> Vec<T> {
    pub fn make_room(&mut self) {
        // grow the capacity
        self.cap += 1;
    }
}
```

この関数は内部で使うために必要なこともあるが、使うと UB につながりうるため、API として公開すべきでない。公開するのであれば、`unsafe` キーワードを付けるべきである。
{% endraw %}
