{% raw %}
# メモリ管理 {#chapter-memory}

<!-- ## About Rust memory safety -->

<!--
<mark>TODO</mark>: explain safe allocations/deallocations, ownership/borrowing,
and identify language constructs that may break memory safety (for instance,
unsound behaviors in older versions of the compiler).
-->

`unsafe` でない Rust（すなわち `unsafe` を使わないコード）では、ほとんどの場合、プログラム中の値が占めるメモリをいつ解放できるかをコンパイラが**自動的に**判断する。
しかし、[先に述べた](../guarantees.md#rust-guarantees)とおり、これは保証では
ない。`unsafe` を使わないコードでもメモリリークは起こりうる。そのため、この章で示す
規則の一部は、`unsafe` キーワードに厳密には関係しない。ただし、

<div markdown="1" class="important">

以下で取り上げる関数は、たとえ `unsafe` でなくても、
*unsafe* な Rust の中でのみ使うべきである。

</div>

原則として、メモリリークは避けなければならない。

<div markdown="1" class="reco" id="MEM-NO-LEAK" type="Rule" title="メモリリークの禁止">

安全な Rust 開発では、メモリリークは一切許されない。

</div>

## [`mem::forget`] とメモリリーク {#forget-and-memory-leaks}

メモリは通常、変数がスコープを抜けることで回収されるが、Rust には
メモリを手動で回収するための特別な関数として、`std::mem` モジュール（または `core::mem`）の
[`mem::forget`] と [`mem::drop`] がある。[`mem::drop`] は、メモリの回収を早めに
引き起こすだけであり、必要に応じて対応するデストラクタを呼び出す。これに対し、
[`mem::forget`] はデストラクタの呼び出しを一切行わない。

```rust
let pair = ('↑', 0xBADD_CAFEu32);
drop(pair);
```

どちらの関数も、Rust では**メモリ安全**である。しかし、[`mem::forget`] を使うと、
その値が管理していたリソースは到達不能になり、回収されないままになる。

<p class="codetag bad">悪い例</p>

```rust
let s = String::from("Hello");
forget(s); // Leak memory
```

特に、[`mem::forget`] を使うと、重要なリソースが解放されずにデッドロックに
つながったり、機密データがメモリから消去されなかったりするおそれがある。このため、
[`mem::forget`] は**セキュアではない**。

<div markdown="1" class="reco" id="MEM-FORGET" type="Rule" title="`mem::forget` を使わない">

安全な Rust 開発では、`std::mem`（`core::mem`）の [`mem::forget`] 関数を
使ってはならない。

</div>

<!-- -->

<div markdown="1" class="reco" id="MEM-FORGET-LINT" type="Recommendation" title="clippy のリントで `mem::forget` の使用を検出する">

[`mem::forget`] の使用を自動的に検出するために、Clippy のリント `mem_forget` を
使うべきである。クレート内で [`mem::forget`] が使われていないことを強制するには、
ルートファイル（通常は `src/lib.rs` または `src/main.rs`）の先頭に次の行を追加する。

```rust
#![deny(clippy::mem_forget)]
```

</div>

標準ライブラリには、値のドロップを*忘れる*方法がほかにもある。

- [`Box::leak`]: リソースをリークさせる。
- [`Box::into_raw`]: unsafe なコード、特に FFI の中で、`Box` を**ポインタ**として利用する。
- [`ManuallyDrop`]（`std::mem` または `core::mem`）: 値の解放を手動で行うよう強制する。

これらの代替手段も同じセキュリティ上の問題につながりうるが、
その目的がはっきり分かるという利点がある。

<div markdown="1" class="reco" id="MEM-LEAK" type="Rule" title="`leak` 関数を使わない">

安全な Rust 開発では、コードは、特に [`Box::leak`] によって、メモリやリソースを
リークさせてはならない。

</div>

[`ManuallyDrop`] と [`Box::into_raw`] は、解放の責任をコンパイラから
開発者へ移す。

<div markdown="1" class="reco" id="MEM-MANUALLYDROP" type="Rule" title="`ManuallyDrop` でラップした値を必ず解放する">

安全な Rust 開発では、[`ManuallyDrop`] でラップした値はすべて、
ラップを外して自動的に解放されるようにする（[`ManuallyDrop::into_inner`]）か、
手動で解放（unsafe な [`ManuallyDrop::drop`]）しなければならない。

</div>

<!-- -->

[`mem::forget`]: https://doc.rust-lang.org/std/mem/fn.forget.html
[`mem::drop`]: https://doc.rust-lang.org/std/mem/fn.drop.html
[`Drop`]: https://doc.rust-lang.org/std/ops/trait.Drop.html
[`Box::leak`]: https://doc.rust-lang.org/std/boxed/struct.Box.html#method.leak
[`Box::into_raw`]: https://doc.rust-lang.org/std/boxed/struct.Box.html#method.into_raw
[`ManuallyDrop`]: https://doc.rust-lang.org/beta/std/mem/struct.ManuallyDrop.html
[`ManuallyDrop::into_inner`]: https://doc.rust-lang.org/beta/std/mem/struct.ManuallyDrop.html#method.into_inner
[`ManuallyDrop::drop`]: https://doc.rust-lang.org/beta/std/mem/struct.ManuallyDrop.html#method.drop

## 生ポインタ

生ポインタは、主に C のポインタを扱うために使われる。*スマートポインタ*のような保護は
なく、多くの場合 `unsafe` な文脈で使わなければならない。例えば、生ポインタの
解放は、Rust の保証なしに手動で行わなければならない。

<div markdown="1" class="reco" id="MEM-NORAWPOINTER" type="Rule" title="`unsafe` を使わない Rust でスマートポインタを生ポインタに変換しない">

`unsafe` を使わない安全な Rust 開発では、参照や*スマートポインタ*を
*生ポインタ*に変換すべきでない。例えば、スマートポインタ [`Box`]、[`Rc`]、[`Arc`]、[`rc::Weak`]、
[`sync::Weak`] の `into_raw` 関数や `into_non_null` 関数は使うべきでない。

それでも `unsafe` なコードなしで*生ポインタ*を使う場合は、文書化し、正当化しなければならない。

</div>

<div markdown="1" class="reco" id="MEM-INTOFROMRAWALWAYS" type="Rule" title="`into_raw` した値には必ず `from_raw` を呼ぶ">

安全な Rust 開発では、次のいずれかの型に対する `into_raw`
（または `into_non_null`）の呼び出しで作られたポインタは、

- [`std::boxed::Box`]（または [`alloc::boxed::Box`]）
- [`std::rc::Rc`]（または [`alloc::rc::Rc`]）
- [`std::rc::Weak`]（または [`alloc::rc::Weak`]）
- [`std::sync::Arc`]（または [`alloc::sync::Arc`]）
- [`std::sync::Weak`]（または [`alloc::sync::Weak`]）
- [`std::ffi::CString`]
- [`std::ffi::OsString`]

回収できるようにするため、最終的には、対応する `from_raw` の呼び出しで
値に戻さなければならない。

```rust
let boxed = Box::new(String::from("Crab"));
let raw_ptr = Box::into_raw(boxed);
let _ = unsafe { Box::from_raw(raw_ptr) }; // will be freed
```

</div>

逆もまた成り立つ。すなわち、`from_raw` は `into_raw` した値に**のみ**呼び出すべきである。例えば、
スマートポインタ [`Rc`] は[この条件を明示的に要求しており](https://doc.rust-lang.org/std/rc/struct.Rc.html#method.from_raw)、
スマートポインタ [`Box`] については、C のポインタを [`Box`] に変換することは[推奨されていない](https://doc.rust-lang.org/std/boxed/index.html#memory-layout)。

<div markdown="1" class="reco" id="MEM-INTOFROMRAWONLY" type="Rule" title="`from_raw` は `into_raw` した値に*のみ*呼ぶ">

安全な Rust 開発では、`from_raw` は `into_raw` した値にのみ呼び出さなければならない。

</div>

<!-- -->

<div markdown="1" class="note">

[`Box::into_raw`] の場合、手動での後始末も可能ではあるが、生ポインタを
再び Box に戻すよりもはるかに複雑であり、避けるべきである。

<p class="codetag bad">悪い例</p>

```rust
// Excerpt from the standard library documentation
use std::alloc::{Layout, dealloc};
use std::ptr;

let x = Box::new(String::from("Hello"));
let p = Box::into_raw(x);
unsafe {
    ptr::drop_in_place(p);
    dealloc(p as *mut u8, Layout::new::<String>());
}
```

ほかの型（[`Rc`] と [`Arc`]）は内部が公開されておらず、より複雑であるため、手動での
後始末はできない。

</div>

[`Box`]: https://doc.rust-lang.org/std/boxed/struct.Box.html
[`std::boxed::Box`]: https://doc.rust-lang.org/std/boxed/struct.Box.html
[`alloc::boxed::Box`]: https://doc.rust-lang.org/alloc/boxed/struct.Box.html
[`Rc`]: https://doc.rust-lang.org/std/rc/struct.Rc.html
[`std::rc::Rc`]: https://doc.rust-lang.org/std/rc/struct.Rc.html
[`alloc::rc::Rc`]: https://doc.rust-lang.org/alloc/rc/struct.Rc.html
[`rc::Weak`]: https://doc.rust-lang.org/std/rc/struct.Weak.html
[`std::rc::Weak`]: https://doc.rust-lang.org/std/rc/struct.Weak.html
[`alloc::rc::Weak`]: https://doc.rust-lang.org/alloc/rc/struct.Weak.html
[`Arc`]: https://doc.rust-lang.org/std/sync/struct.Arc.html
[`std::sync::Arc`]: https://doc.rust-lang.org/std/sync/struct.Arc.html
[`alloc::sync::Arc`]: https://doc.rust-lang.org/alloc/sync/struct.Arc.html
[`sync::Weak`]: https://doc.rust-lang.org/std/sync/struct.Weak.html
[`std::sync::Weak`]: https://doc.rust-lang.org/std/sync/struct.Weak.html
[`alloc::sync::Weak`]: https://doc.rust-lang.org/alloc/sync/struct.Weak.html
[`std::ffi::CString`]: https://doc.rust-lang.org/std/ffi/struct.CString.html
[`std::ffi::OsString`]: https://doc.rust-lang.org/std/ffi/struct.OsString.html

## 未初期化メモリ

Rust は既定で、すべての値を初期化するよう強制し、未初期化メモリの使用を
防いでいる（[`std::mem::uninitialized`] または
[`std::mem::MaybeUninit`] を使う場合を除く）。

<div markdown="1" class="reco" id="MEM-UNINIT" type="Rule" title="未初期化メモリを使わない">

[`std::mem::uninitialized`] 関数（1.38 で非推奨）を使ってはならない。
[`std::mem::MaybeUninit`] 型（1.36 で安定化）は、必要な場合に限り、使用のたびに
明示的に正当化しなければならない。

</div>

未初期化メモリの使用は、二つの異なるセキュリティ上の問題を引き起こしうる。

- 未初期化メモリのドロップ（メモリ安全性の問題でもある）
- 初期化済みメモリがドロップされないこと

<div markdown="1" class="note">

[`std::mem::MaybeUninit`] は [`std::mem::uninitialized`] を改善したものである。
実際、未初期化の値をドロップしてしまうことがはるかに起こりにくくなる。
しかし、二つ目の問題は変わらない。初期化済みメモリがドロップされない問題は
残る。これは、特に機密性の高いメモリを消去するために [`Drop`] を使うことを
考えると、問題である。

</div>

[`std::mem::uninitialized`]: https://doc.rust-lang.org/std/mem/fn.uninitialized.html
[`std::mem::MaybeUninit`]: https://doc.rust-lang.org/beta/std/mem/union.MaybeUninit.html
{% endraw %}
