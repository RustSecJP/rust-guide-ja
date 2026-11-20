{% raw %}

# 外部関数インターフェース（FFI） { #chapter-ffi }

Rust が他の言語と連携する方法は、C との高い互換性を土台にしている。ただし、
この境界はその性質上 **unsafe** である（[Rust Book: Unsafe Rust] を参照）。

[Rust Book: Unsafe Rust]: https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html

`extern` と印を付けた関数は、コンパイル時に C のコードと互換になる。こうした関数は、
C のコードからどのような引数の値でも呼び出されうる。
正確な構文は `extern "<ABI>"` であり、ABI は呼び出し規約を表し、対象プラットフォームに
よって異なる。既定は `C` で、対象プラットフォームにおける標準の C 呼び出し規約に
対応する。

```rust
/// Export C-compatible function
#[unsafe(no_mangle)]
extern "C" fn mylib_f(param: u32) -> i32 {
    if param == 0xCAFEBABE { 0 } else { -1 }
}
```

関数 `mylib_f` に同じ名前でアクセスできるようにするには、この関数に
`#[unsafe(no_mangle)]` 属性も付けなければならない。

逆に、C の関数を `extern` ブロックの中で宣言すれば、Rust から呼び出すことが
できる。

```rust
use std::os::raw::c_int;
// import C function
unsafe extern "C" {
    fn abs(args: c_int) -> c_int;
}

fn use_abs() {
    let x = -1;
    println!("{} {}\n", x, unsafe { abs(x) });
}
```

<div markdown="1" class="note">

`extern` ブロックを通じて Rust に取り込んだ外部関数は、すべて
**自動的に `unsafe`** となる。そのため、外部関数の呼び出しは必ず
`unsafe` な文脈の中で行わなければならない。

</div>

`extern` ブロックには、`static` キーワードを前に付けた外部のグローバル変数の宣言を
含めることもできる。

```rust
// A direct way to access environment variables (on Unix)
// should not be used! Not thread safe, have a look at `std::env`!

unsafe extern "C" {
    // Global variable
    #[link_name = "environ"]
    static libc_environ: *const *const std::os::raw::c_char;
}

fn use_static_extern() {
    let mut next = unsafe { libc_environ };
    while !next.is_null() && !unsafe { *next }.is_null() {
        let env = unsafe { std::ffi::CStr::from_ptr(*next) }
            .to_str()
            .unwrap_or("<invalid>");
        println!("{}", env);
        next = unsafe { next.offset(1) };
    }
}
```

## コードの構造

<div markdown="1" class="reco" id="FFI-SAFEWRAPPING" type="Recommendation" title="外部ライブラリには安全なラッパーを用意する">

他の言語で書かれたライブラリを Rust から利用するときは、次の 2 つの部分に
分けて実装すべきである。

- 元の C の API を `extern` ブロックへ忠実に写し取った低水準のモジュール
  （*非公開* にしてもよい）
- Rust の水準でメモリ安全性とセキュリティ上の不変条件を保証する、安全な
  ラッパーモジュール

低水準の API を外部に公開する場合は、`*-sys` という形式の名前を持つ専用の
クレートとして公開すべきである。

</div>

[`rust-bindgen`] クレートを使うと、C のヘッダファイルからバインディングの低水準部分を
自動生成できる。

<!--
<mark>TODO</mark> example
-->

## 型付け

型付けは、Rust がメモリ安全性を保証するための手段である。同じ保証を備えていない
かもしれない他の言語と連携するときには、メモリ安全性を保つうえで、バインディングで
どの型を選ぶかが極めて重要になる。

### データのレイアウト

Rust は、データがメモリ上にどう配置されるかについて、短期的にも長期的にも何も
保証しない。データを外部の言語と互換にする唯一の方法は、`repr` 属性を使って
C 互換のデータレイアウトを明示的に指定することである（[Rust Reference: Type Layout] を
参照）。たとえば、次の Rust の型は、

```rust
#[repr(C)]
struct Data {
    a: u32,
    b: u16,
    c: u64,
}
#[repr(C, packed)]
struct PackedData {
    a: u32,
    b: u16,
    c: u64,
}
```

次の C の型と互換である。

```c
struct Data
{
	uint32_t a;
	uint16_t b;
	uint64_t c;
};
__attribute__((packed)) struct PackedData
{
	uint32_t a;
	uint16_t b;
	uint64_t c;
};
```

<div markdown="1" class="reco" id="FFI-CTYPE" type="Rule" title="FFI では C 互換の型だけを使う">

セキュアな Rust 開発では、取り込む関数や公開する関数の引数や戻り値の型、および
取り込むグローバル変数や公開するグローバル変数の型として、C と互換でない型を
使ってはならない。

唯一の例外は、外部の側で **不透明** として扱われる型である。

</div>

次の型は C 互換とみなされる。

- 整数または浮動小数点数のプリミティブ型
- `repr(C)` を付けた `struct`
- `repr(C)` または `repr(Int)` を付けた `enum` のうち、バリアントを 1 つ以上持ち、
  どのバリアントもフィールドを持たないもの（`Int` は整数のプリミティブ型）
- 生ポインタ
- `Option<T>` のうち、`T` が次のいずれかであるもの
  - `core::ptr::NonNull<U>` で、`U` が `Sized` な C 互換の型であるもの。この場合は
      `*const T` および `*mut T` のポインタと互換になる。
  - `core::num::NonZero*`。この場合は対応する整数のプリミティブ型と
      互換になる。
- `repr(transparent)` を付けた、フィールドを 1 つだけ持つ `struct` のうち、その
  フィールドの型が C 互換であるもの

次の型は C 互換ではない。

- 動的サイズ型
- トレイトオブジェクト
- フィールドを持つ列挙型
- タプル（ただし `repr(C)` を付けたタプル構造体は問題ない）

一部の型は、注意点付きで互換である。

- ゼロサイズ型。Rust では本当にサイズがゼロになる（C では未規定であり、C++ の
  仕様とは矛盾する）。
- `repr(C)`、`repr(C, Int)`、`repr(Int)` のいずれかを付けた、フィールドを持つ enum
  （[RFC 2195](https://rust-lang.github.io/rfcs/2195-really-tagged-unions.html) を参照）。

[Rust Reference: Type Layout]: https://doc.rust-lang.org/reference/type-layout.html

### 型の一貫性

<div markdown="1" class="reco" id="FFI-TCONS" type="Rule" title="FFI の境界では一貫した型を使う">

型は、FFI の境界の両側で一貫していなければならない。

一方の側で、他方に対して一部の詳細を隠してもよい（典型的には型を不透明にする
場合）が、両側の型は同じサイズと同じアラインメント要件を持たなければならない。

</div>

特にフィールドを持つ列挙型については、対応する C（または C++）の型は自明では
ない。[RFC 2195](https://rust-lang.github.io/rfcs/2195-really-tagged-unions.html) を参照のこと。

[`rust-bindgen`] や [`cbindgen`] のようなバインディングの自動生成ツールは、C と Rust の
間で型を一貫させる助けになる。

<div markdown="1" class="reco" id="FFI-AUTOMATE" type="Recommendation" title="バインディングの自動生成ツールを使う">

セキュアな Rust 開発では、可能な限り自動生成ツールを使ってバインディングを生成し、
継続的に保守すべきである。

</div>

<!-- -->

<div markdown="1" class="warning">

C/C++ を Rust にバインドする場合、[`rust-bindgen`] で低水準のバインディングを
自動生成できる。そのうえで、高水準の安全なバインディングを用意することを強く
推奨する（推奨 [FFI-SAFEWRAPPING](ffi.md#FFI-SAFEWRAPPING) を参照）。
また、[`rust-bindgen`] のオプションの中には、危険な変換を引き起こしうるものがある。
特に `rustified_enum` がそうである。

</div>

[`rust-bindgen`]: https://crates.io/crates/bindgen
[`cbindgen`]: https://crates.io/crates/cbindgen

### プラットフォーム依存の型

C や C++ のような外部の言語と連携するときには、C の `int` や `long` などの
プラットフォーム依存の型を使う必要がしばしば生じる。

標準ライブラリは、`void` に対応する `std::ffi`（または `core::ffi`）の `c_void` に加えて、
移植性のある型エイリアスを `std:os::raw`（または `core::os::raw`）で提供している。

- `c_char`: `char` に対応（`i8` または `u8`）
- `c_schar`: `signed char` に対応（常に `i8`）
- `c_uchar`: `unsigned char` に対応（常に `u8`）
- `c_short`: `short` に対応
- `c_ushort`: `unsigned short` に対応
- `c_int`: `int` に対応
- `c_uint`: `unsigned int` に対応
- `c_long`: `long` に対応
- `c_ulong`: `unsigned long` に対応
- `c_longlong`: `long long` に対応
- `c_ulonglong`: `unsigned long long` に対応
- `c_float`: `float` に対応（常に `f32`）
- `c_double`: `double` に対応（常に `f64`）

[`libc`] クレートは、C 標準ライブラリをほぼ網羅する、さらに多くの C 互換の型を
提供している。

<div markdown="1" class="reco" id="FFI-PFTYPE" type="Rule" title="プラットフォーム依存の型にバインドするときは移植性のあるエイリアス `c_*` を使う">

セキュアな Rust 開発では、C の `int` や `long` のようなプラットフォーム依存の型を
使う外部のコードと連携するとき、Rust のコードはプラットフォーム固有の型ではなく、
標準ライブラリや [`libc`] クレートが提供するような移植性のある型エイリアスを
使わなければならない。ただし、バインディングをプラットフォームごとに自動生成する
場合は除く（下の注を参照）。

</div>

<!-- -->

<div markdown="1" class="note">

バインディングの自動生成ツール（[`cbindgen`]、[`rust-bindgen`] など）は、特定の
プラットフォーム上での型の一貫性を保証できる。生成結果が対象プラットフォームに
対して健全であることを保証するため、これらのツールはビルドの過程でターゲットごとに
使うべきである。

</div>

[`libc`]: https://crates.io/crates/libc

### 堅牢でない型: 参照、関数ポインタ、列挙型 {#robustness}

ある型の *トラップ表現* とは、その型の表現上の制約（サイズやアラインメントなど）は
満たしているものの、その型の有効な値を表しておらず、未定義動作を引き起こす
表現（ビットパターン）のことである。

簡単に言えば、Rust の変数にこのような無効な値が設定されると、単なるプログラムの
クラッシュから任意コード実行まで、何が起きてもおかしくない。
安全な Rust を書いている限り、これは起こりえない（Rust コンパイラのバグによる
場合を除く）。しかし、unsafe な Rust を書くとき、特に FFI では、いとも簡単に
起こる。

以下では、このようなトラップ表現を（少なくとも 1 つ）持つ型を **堅牢でない型** と
呼ぶ。Rust の型の多くは堅牢でなく、C 互換の型の中にもそうしたものがある。

- `bool`（1 バイトで 256 通りの表現があるが、有効なものは 2 つだけ）
- 参照
- 関数ポインタ
- 列挙型
- 浮動小数点数（有効な浮動小数点数が何であるかについては、ほぼすべての言語の
  理解が一致しているとはいえ）
- 堅牢でない型のフィールドを含む複合型

一方、整数型（`u*`/`i*`）や、堅牢でないフィールドを含まないパックされた複合型は、
*堅牢な型* の例である。

堅牢でない型は、2 つの言語を連携させるうえでの難所である。問題の中心は、
**境界を越える値の有効性を保証する責任を、2 つの言語のどちらが負うのか**、
そしてそれをどう行うのかを決めることにある。

<div markdown="1" class="reco" id="FFI-CKNONROBUST" type="Rule" title="検査していない堅牢でない外部の値を使わない">

セキュアな Rust 開発では、堅牢でない型の外部の値を *検査しないまま* 使っては
ならない。

言い換えれば、Rust の側が明示的な検査を通じて堅牢な型を堅牢でない型に変換するか、
外部の側が値の有効性を強く保証するかのどちらかである。

</div>

<!-- -->

<div markdown="1" class="reco" id="FFI-CKINRUST" type="Recommendation" title="外部の値は Rust で検査する">

セキュアな Rust 開発では、外部の値の有効性の検査は、可能な限り Rust の側で
行うべきである。

</div>

これらの一般的な規則は、相手となる外部の言語や、付随するリスクに応じて調整する
必要がある。言語について言えば、C は有効性を保証するのに特に不向きである。
とはいえ、強い保証を提供する言語は Rust だけではない。たとえば、C++ の一部の
サブセット（再解釈を使わないもの）では、開発者は多くの型検査を行える。Rust は
安全な部分と unsafe な部分を言語の水準で分離しているので、可能な限り常に Rust で
検査することを推奨する。リスクについて言えば、最も危険な型は参照、関数参照、
列挙型であり、これらについて以下で述べる。

<div markdown="1" class="warning">

Rust の `bool` は、C99 の `_Bool`（`<stdbool.h>` では `bool` という別名を持つ）および
C++ の `bool` と等価になるように作られている。しかし、0 と 1 以外の値を
`_Bool`/`bool` として読み込むことは、*どちらの側でも* 未定義動作である。
安全な Rust では、そうならないことが保証される。標準に準拠した C や C++ の
コンパイラは、`_Bool`/`bool` の値に 0 と 1 以外が *格納* されないことは保証するが、
*誤った再解釈*（共用体型やポインタのキャストなど）が起きないことまでは
保証できない。こうした不正な再解釈を検出するには、[LLVM] の `-fsanitize=bool` の
ようなサニタイザを使うとよい。

</div>

[LLVM]: https://llvm.org/

#### 参照とポインタ

型 `&mut T` と `&T` には、型 T と同じ有効性の要件が課される。それに加えて、
正しく割り当てられた値を指していなければならないという制約がある。
これらに少しでも反すると未定義動作となる。

C との間でバインドする場合、この問題は特に深刻である。C には（有効なポインタと
いう意味での）参照がなく、コンパイラも安全性を何ら保証しないからである。

C++ とバインドする場合、Rust の参照は実用上 C++ の参照にバインドしてよい。ただし、
参照を伴う `extern "C"` 関数の C++ における実際の ABI は「処理系定義」である。
また、C++ のコードについては、ポインタと参照の取り違えがないかを確認すべきである。

Rust の参照は、C 互換の他の言語との間でも無理なく使える場合がある。これには、
非 null であることを型検査できる C の派生言語、たとえば Microsoft SAL の注釈を
付けたコードが含まれる。

これに対して、Rust の生ポインタ型（`*const T` と `*mut T`）は、有効性の検査を
バインドの時点から使用の時点へと先送りする。さらに、生ポインタでは、有効性を
主として `std/core::ptr::null()`（C の `(void*)0`）との比較で確かめられるほか、
状況によっては（特に組み込みシステムやカーネルレベルのプログラミングで）既知の
有効なメモリ範囲に照らして、あるいはアラインメント要件に照らして確かめることも
できる。

FFI で Rust のポインタを使うもう 1 つの利点は、指している先の値を読み込む箇所が、
必ず `unsafe` なブロックまたは関数の中にあるものとして明確に示されることである。

<div markdown="1" class="reco" id="FFI-CK-PTR-VALID" type="Rule" title="外部のポインタを検査する">

セキュアな Rust 開発では、外部のポインタを参照外しする Rust のコードは、事前に
その有効性を検査しなければならない。
特に、ポインタは使用する前に、null でないことを検査しなければならない。

可能であれば、より強力な手法をとることが望ましい。たとえば、既知の有効なメモリ
範囲に照らしてポインタを検査する、ポインタにタグを付ける（または署名する）
（指している先の値を Rust からしか操作しない場合に特に適している）、データの
アラインメントを検査する、といった手法である。

</div>

<div markdown="1" class="reco" id="FFI-INPUT-PTR" type="Recommendation" title="外部の言語から来るポインタは生ポインタで表現する">

セキュアな Rust 開発では、外部の言語から受け取るポインタは、Rust の *生ポインタ* で表現すべきである。

例外は次のとおり。

- 外部の言語では不透明であり、Rust の側からしか操作しない Rust の参照
- `Option` で包んだ参照（下の注を参照）
- 外部の安全な参照にバインドした参照。たとえば、拡張された C の派生言語に
  由来するものや、`extern "C"` の参照がポインタとして表現される環境で
  コンパイルされた C++ に由来するもの

</div>

上の推奨を適用できない場合、ポインタの有効性の検査は、そのポインタの出どころである
言語（つまり外部の言語）で行わなければならない。

<div markdown="1" class="reco" id="FFI-CK-INPUT-REF-VALID" type="Rule" title="検査していない外部の参照を使わない">

セキュアな Rust 開発では、FFI を通じて Rust に渡される外部の参照はすべて、
自動で（たとえばコンパイラによって）、または手作業で、**外部の側で検査**
されなければならない。

例外は、不透明なラッパーに包まれていて Rust の側でのみ生成・操作される Rust の
参照と、`Option` で包んだ参照である（下の注を参照）。

</div>

<!-- -->

次のコードは、公開された Rust の関数で外部のポインタを使う簡単な例である。

```rust
/// Add in place
#[unsafe(no_mangle)]
unsafe extern "C" fn add_in_place(a: *mut u32, b: u32) {
    // checks for nullity of `a`
    // and takes a mutable reference on it if it's non-null
    if let Some(a) = unsafe { a.as_mut() } {
        *a += b
    }
}
```

`as_ref` メソッドと（可変ポインタ用の）`as_mut` メソッドを使うと、いかにも
*Rust らしい* やり方で null 検査を確実に行いながら、参照を簡単に得られることに
注意してほしい。
一方、C の側では次のように使える。

```c
#include <stdint.h>
#include <inttypes.h>

//! Add in place
void add_in_place(uint32_t *a, uint32_t b);

int use_add_in_place()
{
	uint32_t x = 25;
	add_in_place(&x, 17);
	printf("%" PRIu32 " == 42", x);
	return 0;
}
```
<div markdown="1" class="note">

任意の `T: Sized` に対する `Option<&T>` と `Option<&mut T>` は、null かどうかを
明示的に検査するポインタの代わりに、FFI で使ってよい。Rust が保証する
「null 許容ポインタ最適化」により、C の側では null になりうるポインタを使って
かまわない。C の `NULL` は Rust では `None` と解釈され、null でないポインタは
`Some` に包まれる。この機能はかなり使い勝手がよいが、メモリ範囲の検査のような、
より強力な検証はできない。

</div>

#### 関数ポインタ

FFI の境界を越える関数ポインタは、最終的に任意コード実行につながるおそれがあり、
現実のセキュリティリスクとなる。

<div markdown="1" class="reco" id="FFI-MARKEDFUNPTR" type="Rule" title="FFI の関数ポインタ型には `extern` と `unsafe` を付ける">

セキュアな Rust 開発では、FFI の境界にある関数ポインタ型にはすべて、`extern`
（必要に応じて特定の ABI を指定する）と `unsafe` を付けなければならない。

</div>

Rust の関数ポインタは、通常のポインタよりも参照にはるかに近い。特に、関数ポインタの
有効性は、Rust の側で直接検査することができない。ただし、Rust には代わりとなる
方法が 2 つある。

- `Option` で包んだ関数ポインタを使い、`null` かどうかを検査する。

  ```rust
  #[unsafe(no_mangle)]
  pub unsafe extern "C" fn repeat(
      start: u32,
      n: u32,
      f: Option<unsafe extern "C" fn(u32) -> u32>,
  ) -> u32 {
      if let Some(f) = f {
          let mut value = start;
          for _ in 0..n {
              value = unsafe { f(value) };
          }
          value
      } else {
          start
      }
  }
  ```

  C の側では次のようになる。

  ```c
  uint32_t repeat(uint32_t start, uint32_t n, uint32_t (*f)(uint32_t));
  ```

- 生ポインタを使い、`unsafe` な transmute で関数ポインタ型に変換する。使い勝手は
  犠牲になるが、より強力な検査ができる。

<div markdown="1" class="reco" id="FFI-CKFUNPTR" type="Rule" title="外部の関数ポインタを検査する">

セキュアな Rust 開発では、外部の関数ポインタはすべて、FFI の境界で検査しなければ
ならない。

</div>

C とバインドする場合はもちろん、C++ とバインドする場合であっても、関数ポインタの
有効性を保証することは容易でない。そのうえ、C++ の関数オブジェクト（ファンクタとも呼ばれる）は C 互換ではない。

#### 列挙型

通常、`enum` の有効な値がとりうるビットパターンの数は、同じサイズでとりうる
ビットパターンの総数に比べてごくわずかである。外部のコードから渡された `enum` の値の
扱いを誤ると、型の取り違えにつながり、ソフトウェアのセキュリティに深刻な影響を
及ぼしかねない。残念ながら、FFI の境界で `enum` の値を検査することは、どちらの
側でも簡単ではない。

Rust の側では、`extern` ブロックの宣言で実際には整数型という *堅牢な* 型を使って
おき、そのあとで検査付きの変換を行って enum 型にする、という形になる。

外部の側で検査できるのは、相手の言語が素の C よりも厳密な検査を行える場合に
限られる。たとえば C++ の `enum class` は使ってよい。
ただし、参照の場合と同様に、`enum class` の `extern "C"` における実際の ABI は
処理系定義であり、環境ごとに確認すべきであることに注意してほしい。

<div markdown="1" class="reco" id="FFI-NOENUM" type="Rule" title="FFI の境界で外から入ってくる Rust の `enum` を使わない">

セキュアな Rust 開発では、外部の言語と連携するとき、Rust のコードは Rust の
`enum` 型の値を外から受け取ってはならない。

例外は、次のいずれかに該当する Rust の `enum` 型である。

- 外部の言語では不透明であり、Rust の側からしか操作しないもの
- 外部の言語の安全な列挙型にバインドしたもの。たとえば C++ の `enum class` 型

</div>

フィールドを持たない列挙型については、[`num_derive`] や [`num_enum`] のような
クレートを使うと、整数から列挙型への安全な変換を簡単に用意できる。これらは、
（C の `enum` から渡された）整数を Rust の enum へ安全に変換するために使える。

[num_derive]: https://crates.io/crates/num_derive
[num_enum]: https://crates.io/crates/num_enum

### 不透明型

型を不透明にすることは、ソフトウェア開発でモジュール性を高めるよい方法である。
複数の言語を使った開発では、ごく一般的に行われる。

<div markdown="1" class="reco" id="FFI-R-OPAQUE" type="Recommendation" title="外部の不透明型には専用の Rust の型を使う">

セキュアな Rust 開発では、外部の不透明型をバインドするとき、`c_void` への
ポインタではなく、専用の不透明型へのポインタを使うべきである。

</div>

現在のところ、外部の不透明型を作る方法として推奨されるのは次のとおりである。

```rust
#[repr(C)]
pub struct OpaqueFoo {
    _private: [u8; 0],
}
unsafe extern "C" {
    fn handle_opaque_foo(arg: *mut OpaqueFoo);
}
```

まだ安定化されていない [RFC 1861](https://rust-lang.github.io/rfcs/1861-extern-types.html) は、`extern` ブロックの中で不透明型を
宣言できるようにすることで、この表現を容易にすることを提案している。

<div markdown="1" class="reco" id="FFI-C-OPAQUE" type="Recommendation" title="型を不透明にするには不完全な C/C++ の `struct` へのポインタを使う">

セキュアな Rust 開発では、C や C++ と連携するとき、C/C++ で不透明として扱う
Rust の型は、不完全な `struct` 型（つまり、定義なしで宣言されたもの）に変換し、
専用のコンストラクタとデストラクタを用意すべきである。

</div>

不透明な Rust の型の例を示す。

```rust
pub struct Opaque {
    // (...) hide details
}

#[unsafe(no_mangle)]
pub unsafe extern "C" fn new_opaque() -> *mut Opaque {
    catch_unwind(|| // Catch panics, see below
        Box::into_raw(Box::new(Opaque {
            // (...) construction
        })))
    .unwrap_or(std::ptr::null_mut())
}

#[unsafe(no_mangle)]
pub unsafe extern "C" fn destroy_opaque(o: *mut Opaque) {
    catch_unwind(|| {
        if !o.is_null() {
            // only necessary when `Opaque` or any of its fields is `Drop`
            drop(unsafe { Box::from_raw(o) })
        }
    })
    .unwrap_or_default();
}
```

### Rust のメモリモデルの保持 {#ffi-memory-model}

Rust コンパイラは、*借用チェッカー* から得た情報を使ってコードの実行を最適化する。
その際、借用チェッカーが検証した条件、特に
[エイリアシング](https://doc.rust-lang.org/nomicon/aliasing.html)の規則が成り立っていると
仮定し、その仮定に基づいて最適化を行う。このメモリモデルの詳細は、
[`core` の API ドキュメント](https://doc.rust-lang.org/core/ptr/index.html) にある。

したがって、これらの仮定に違反すると、未定義動作（UB）が生じうる。

<div markdown="1" class="reco" id="FFI-CK-REF-MODEL" type="Rule" title="境界で間接参照を受け渡すときの Rust のメモリモデルの保持">

セキュアな Rust 開発では、境界を越える間接参照（ポインタ、参照）の使用は、
Rust のメモリモデルを保たなければならない。

</div>

<div markdown="1" class="example">

次の例では、Rust のコードが C に由来するポインタを操作する。これは、C の側から
Rust の不変条件を守ることの重要性を示している。

下で定義する `swap` 関数は、次の C のコードから呼び出されると未定義動作（UB）を
引き起こす。Rust のメモリモデルは、可変参照の *エイリアシング* を禁じているからである。


```rust
extern "C" fn swap(a: *mut u8, b: *mut u8) {
    let a = unsafe { a.as_mut() }.unwrap();
    let b = unsafe { b.as_mut() }.unwrap();
    std::mem::swap(a, b);
}
```

<p class="codetag ub">未定義動作になる例</p>

```c
void use_swap()
{
	uint8_t a = 42;
	swap(&a, &a);
}
```

</div>

<div markdown="1" class="example">

次の例では、Rust のコードが C のコードにポインタを渡す。これは、C の側で行われる
操作を Rust の型システムに反映することの重要性を示している。

取り込んだ関数 `inc_wrap` は、変数 `a` をインクリメントする。

```c
void inc_wrap(uint8_t *a)
{
	if (*a == 255)
	{
		*a = 0;
	}
	else
	{
		*a = *a + 1;
	}
}
```
Rust のコードは、不変の変数 `val` へのポインタを C の関数に渡しており、
それによって Rust のメモリモデルに違反し、*UB* を引き起こす。


```rust
unsafe extern "C" {
    fn inc_wrap(a: *mut u8);
}

fn run_inc() {
    let val: u8 = 6;
    let p = (&val as *const u8) as *mut u8;
    unsafe { inc_wrap(p) };
}
```

この例の誤りは、C ライブラリの API へのバインディングにもっと強い制約を課して
いれば検出できたはずである。その方法としては、シグネチャを直接変更するか、
次の例のように `inc_wrap` 関数をラップするかがある。

<p class="codetag good">良い例</p>

```rust
fn safe_inc_wrap(a: &mut u8) {
    unsafe {
        inc_wrap(a);
    }
}
```

`safe_inc_wrap` 関数は、次の理由から確かに *安全* である。

* どのようなメモリ上の値も `u8` 型の値として解釈できる（`u8` 型は[堅牢](#robustness)である）。
* `safe_inc_wrap` 関数の型が、*エイリアシングがないこと* と *可変性* の制約を強制する。
* C のコードは、`inc_wrap` 関数の呼び出しが終わったあとまで、ポインタを解放することも（たとえば `static` 変数を通じて）「保持」することもない。

</div>

## メモリと資源の管理

プログラミング言語は、それぞれ異なる方法でメモリを扱う。そのため、Rust と他の
言語の間でデータを受け渡すときには、そのデータが占めるメモリ領域を回収する責任を
どちらの言語が負うのかを把握しておくことが重要である。
ソケットやファイルといった他の種類の資源についても同じことが言える。

Rust は変数の所有権とライフタイムを追跡し、メモリを解放すべきかどうか、解放する
ならいつかをコンパイル時に決定する。[`Drop`] トレイトのおかげで、この仕組みを
利用して、ファイルやネットワークへのアクセスといった他の種類の資源も回収できる。
あるデータを Rust から外部の言語へ *ムーブ* することは、そのデータに結び付いていた
はずの回収処理を手放すことも意味する。

<div markdown="1" class="reco" id="FFI-MEM-NODROP" type="Rule" title="FFI の境界で `Drop` を実装する型を使わない">

セキュアな Rust 開発では、Rust のコードは、外部のコードへ直接（つまり、ポインタや
参照を介さずに）渡される型に対して [`Drop`] を実装してはならない。

</div>

実際には、`Copy` な型だけを使うことが望ましい。なお、`*const T` は、T が `Copy` で
なくても `Copy` である。

とはいえ、メモリや資源を回収しないことがよくないとしても、回収済みのメモリを
使ったり、ある資源を二重に回収したりすることのほうが、セキュリティの観点からは
さらに悪い。資源をただ 1 度だけ正しく解放するには、メモリの割り当てと解放の責任を
どちらの言語が負うのかを把握していなければならない。

<div markdown="1" class="reco" id="FFI-MEM-OWNER" type="Rule" title="FFI ではデータの所有権を明確にする">

セキュアな Rust 開発では、ある型のデータがコピーされずに FFI の境界を越える
とき、次のことを保証しなければならない。

- データの割り当てと解放の両方について、単一の言語が責任を負う。
- もう一方の言語は、そのデータを直接割り当てたり解放したりしてはならず、責任を負う
  言語が提供する専用の外部関数を使う。

</div>

所有権だけでは十分でない。さらに、ライフタイムが正しいこと、とりわけ回収後に
使用されないことを保証する必要がある。これははるかに難しい。メモリの責任を相手の
言語が負う場合、最もよい方法は、外部の型を包む安全なラッパーを用意することである。

<div markdown="1" class="reco" id="FFI-MEM-WRAPPING" type="Recommendation" title="外部のデータをメモリを解放するラッパーで包む">

セキュアな Rust 開発では、外部の言語で割り当てと解放が行われる外部のデータの
うち機微でないものはすべて、[`Drop`] を実装する型に包み、外部の言語の解放
ルーチンが自動的に呼び出されることで、Rust の中で自動的に解放されるように
すべきである。

</div>

外部の不透明型を Rust でラップする簡単な例を示す。

```rust
/// Private “raw” opaque foreign type Foo
#[repr(C)]
struct RawFoo {
    _private: [u8; 0],
}

// Private “raw” C API
unsafe extern "C" {
    fn foo_create() -> *mut RawFoo;
    fn foo_do_something(this: *const RawFoo);
    fn foo_destroy(this: *mut RawFoo);
}

/// Foo
pub struct Foo(*mut RawFoo);

impl Foo {
    /// Create a Foo
    pub fn new() -> Option<Foo> {
        let raw_ptr = unsafe { foo_create() };
        if raw_ptr.is_null() {
            None
        } else {
            Some(Foo(raw_ptr))
        }
    }

    /// Do something on a Foo
    pub fn do_something(&self) {
        unsafe { foo_do_something(self.0) }
    }
}

impl Drop for Foo {
    fn drop(&mut self) {
        if !self.0.is_null() {
            unsafe { foo_destroy(self.0) }
        }
    }
}
```
<div markdown="1" class="warning">

パニックが起きると `Drop::drop` メソッドが実行されないことがあるので、機微な
解放処理（機微なデータの消去など）にはこの解決策では不十分である。ただし、
コードが決してパニックしないと保証されている場合は除く。

機微なデータの消去については、専用のパニックハンドラを用意することで対処できる。

</div>

Rust が割り当てた資源を外部の言語の側が利用する場合、何らかの保証を与えることは
はるかに難しくなる。

たとえば C では、適切なデストラクタが呼び出されたことを確かめる簡単な方法がない。
1 つの方法として、コールバックを利用して回収が確実に行われるようにすることが
考えられる。

次の Rust のコードは、資源を安全に回収するためのコールバックを備えた C 互換の
API の例である。ただし、この例は **スレッドセーフではない**。

```rust
pub struct XtraResource {/* fields */}

impl XtraResource {
    pub fn new() -> Self {
        XtraResource { /* ... */ }
    }
    pub fn dosthg(&mut self) {
        /* ... */
    }
}

impl Drop for XtraResource {
    fn drop(&mut self) {
        println!("xtra drop");
    }
}

pub mod c_api {
    use super::XtraResource;
    use std::panic::catch_unwind;

    const INVALID_TAG: u32 = 0;
    const VALID_TAG: u32 = 0xDEAD_BEEF;
    const ERR_TAG: u32 = 0xDEAF_CAFE;

    static mut COUNTER: u32 = 0;

    pub struct CXtraResource {
        tag: u32, // to detect accidental reuse
        id: u32,
        inner: XtraResource,
    }

    #[unsafe(no_mangle)]
    pub unsafe extern "C" fn xtra_with(cb: Option<unsafe extern "C" fn(*mut CXtraResource) -> ()>) {
        let inner = if let Ok(res) = catch_unwind(XtraResource::new) {
            res
        } else {
            return;
        };
        let id = unsafe { COUNTER };
        let tag = VALID_TAG;

        unsafe { COUNTER = COUNTER.wrapping_add(1) };
        // Use heap memory and do not provide pointer to stack to C code!
        let mut boxed = Box::new(CXtraResource { tag, id, inner });

        if let Some(cb) = cb {
            unsafe { cb(boxed.as_mut() as *mut CXtraResource) };
        }

        if boxed.id == id && (boxed.tag == VALID_TAG || boxed.tag == ERR_TAG) {
            boxed.tag = INVALID_TAG; // prevent accidental reuse
        // implicit boxed drop
        } else {
            // (...) error handling (should be fatal)
            boxed.tag = INVALID_TAG; // prevent reuse
            std::mem::forget(boxed); // boxed is corrupted it should not be
        }
    }

    #[unsafe(no_mangle)]
    pub unsafe extern "C" fn xtra_dosthg(cxtra: *mut CXtraResource) {
        let do_it = || {
            if let Some(cxtra) = unsafe { cxtra.as_mut() }
                && cxtra.tag == VALID_TAG
            {
                cxtra.inner.dosthg();
                return;
            }
            println!("do noting with {:p}", cxtra);
        };
        if catch_unwind(do_it).is_err()
            && let Some(cxtra) = unsafe { cxtra.as_mut() }
        {
            cxtra.tag = ERR_TAG;
        };
    }
}
```

これに対応する C からの呼び出しは次のとおり。

```c
struct XtraResource;
void xtra_with(void (*cb)(struct XtraResource *xtra));
void xtra_sthg(struct XtraResource *xtra);

void cb(struct XtraResource *xtra)
{
	// ()...) do anything with the proposed C API for XtraResource
	xtra_sthg(xtra);
}

int use_xtra()
{
	xtra_with(cb);
}
```

## 外部のコードとパニック {#ffi-panic}

他の言語（C など）から Rust のコードを呼び出す場合、Rust のコードは決して
パニックしないように注意すべきである。
バージョン [1.81.0](https://releases.rs/docs/1.81.0/) 以降、Rust のコードから外部の
コードへのスタックの巻き戻しは未定義動作ではなくなった。しかし、意図しない異常終了を
防ぐため、スタックの巻き戻しを言語の間で厳密に分離することを推奨する[^ffi-panic-abort]。

[^ffi-panic-abort]: 詳細は[巻き戻しの仕様](https://doc.rust-lang.org/stable/reference/items/functions.html#unwinding) にある。

<div markdown="1" class="reco" id="FFI-NOPANIC" type="Recommendation" title="FFI では `panic!` を正しく扱う">

FFI から呼び出される Rust のコードは、関数がパニックしえないことを保証するか、
パニックを処理する仕組み（[`std::panic::catch_unwind`]、
[`std::panic::set_hook`]、[`#[panic_handler]`](https://doc.rust-lang.org/reference/panic.html#r-panic.panic_handler) など）を使って、Rust のコードが
異常終了したり不安定な状態のまま戻ったりしないことを保証すべきである。

</div>

[`catch_unwind`] が捕捉するのは巻き戻しを伴うパニックだけであり、プロセスを
異常終了させるパニックは捕捉しないことに注意してほしい。

```rust
fn may_panic() {
    /* Something may panic... */
}

#[unsafe(no_mangle)]
pub unsafe extern "C" fn no_panic() -> i32 {
    let result = catch_unwind(may_panic);
    match result {
        Ok(_) => 0,
        Err(_) => -1,
    }
}
```

[`catch_unwind`]: https://doc.rust-lang.org/std/panic/fn.catch_unwind.html
[`std::panic::catch_unwind`]: https://doc.rust-lang.org/std/panic/fn.catch_unwind.html
[`std::panic::set_hook`]: https://doc.rust-lang.org/std/panic/fn.set_hook.html

### `no_std`

`#![no_std]` のプログラムでは、セキュリティを確保するためにパニックハンドラ（[`#[panic_handler]`](https://doc.rust-lang.org/reference/panic.html#r-panic.panic_handler)）を
定義しなければならない。パニックハンドラは、プログラムの安全性とセキュリティの
両方を確保できるよう、細心の注意を払って書くべきである。

もう 1 つの方法は、[`panic-never`] クレートを使って、`panic!` が一切使われていない
ことを単純に保証することである。[`no-panic`] と同様に、[`panic-never`] はリンク時の
仕掛けを利用している。すなわち、到達しないことが自明でない分岐が `panic!` に
つながっていると、リンカが失敗する。

[`panic-never`]: https://crates.io/crates/panic-never
[`no-panic`]: https://crates.io/crates/no-panic


## Rust のライブラリを他の言語にバインドする

<div markdown="1" class="reco" id="FFI-CAPI" type="Rule" title="専用の C 互換 API だけを公開する">

セキュアな Rust 開発では、Rust のライブラリを外部の言語に公開するときは、
**専用の C 互換 API** だけを通じて行うべきである。

</div>

[`cbindgen`] クレートを使うと、Rust のライブラリが備える C 互換の Rust API に対する
C や C++ のバインディングを自動生成できる。

### C 向けに公開した Rust ライブラリの最小の例

`src/lib.rs`:

```rust
/// Opaque counter
pub struct Counter(u32);

impl Counter {
    /// Create a counter (initially at 0)
    fn new() -> Self {
        Self(0)
    }
    /// Get the current value of the counter
    fn get(&self) -> u32 {
        self.0
    }
    /// Increment the value of the counter if there's no overflow
    fn incr(&mut self) -> bool {
        if let Some(n) = self.0.checked_add(1) {
            self.0 = n;
            true
        } else {
            false
        }
    }
}

// C-compatible API

#[unsafe(no_mangle)]
pub unsafe extern "C" fn counter_create() -> *mut Counter {
    Box::into_raw(Box::new(Counter::new()))
}

#[unsafe(no_mangle)]
pub unsafe extern "C" fn counter_incr(counter: *mut Counter) -> std::os::raw::c_int {
    if let Some(counter) = unsafe { counter.as_mut() } {
        if counter.incr() { 0 } else { -1 }
    } else {
        -2
    }
}

#[unsafe(no_mangle)]
pub unsafe extern "C" fn counter_get(counter: *const Counter) -> u32 {
    if let Some(counter) = unsafe { counter.as_ref() } {
        counter.get()
    } else {
        0
    }
}

#[unsafe(no_mangle)]
pub unsafe extern "C" fn counter_destroy(counter: *mut Counter) -> std::os::raw::c_int {
    if !counter.is_null() {
        let _ = unsafe { Box::from_raw(counter) }; // get box and drop
        0
    } else {
        -1
    }
}
```

[`cbindgen`] を使うと（`cbindgen -l c > counter.h`）、整合のとれた C のヘッダ
`counter.h` を生成できる。

```c
#include <stdarg.h>
#include <stdbool.h>
#include <stdint.h>
#include <stdlib.h>

typedef struct Counter Counter;

Counter *counter_create(void);

int counter_destroy(Counter *counter);

uint32_t counter_get(const Counter *counter);

int counter_incr(Counter *counter);
```

`counter_main.c`:

```c
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <inttypes.h>

#include "counter.h"

int main(int argc, const char **argv)
{
	if (argc < 2)
	{
		return -1;
	}
	size_t n = (size_t)strtoull(argv[1], NULL, 10);

	Counter *c = counter_create();
	for (size_t i = 0; i < n; i++)
	{
		if (counter_incr(c) != 0)
		{
			printf("overflow\n");
			counter_destroy(c);
			return -1;
		}
	}

	printf("%" PRIu32 "\n", counter_get(c));
	counter_destroy(c);

	return 0;
}
```

[`Drop`]: https://doc.rust-lang.org/std/ops/trait.Drop.html
{% endraw %}
