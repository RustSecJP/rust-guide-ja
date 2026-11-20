{% raw %}
# 整数演算 {#chapter-integer}

Rust は整数オーバーフローの可能性についてある程度の検査を行うが、整数の算術演算を実行する際には注意が必要である。

特に、コンパイルプロファイル（典型的には、既定のデバッグビルドである *dev* と、標準の最適化ビルドである *release*）によって、整数オーバーフロー時の動作が変わる点に注意すること。*dev* 構成ではオーバーフローが起きるとプログラムが終了する（`panic`）のに対し、*release* 構成では計算結果がその数値型のビット数に黙って切り詰められ、ラップアラウンド（折り返し）の意味論になる。

<div markdown="1" class="note">

整数オーバーフロー時の動作は、コンパイルオプション `-C overflow-checks=true`（または `false`）で明示的に指定できる。
`Cargo.toml` のプロファイル定義で変更することもできる。

```toml
[profile.release]
overflow-checks = true  # release ビルドでオーバーフロー検査を有効にする
```

</div>

オーバーフローが起こりうる場合は、専用のメソッドか専用のラッパー型を使うことで、動作を明示できる。

メソッドは `<mode>_<op>` という形式である。`<mode>` は `checked`、`overflowing`、`wrapping`、`saturating` のいずれかで、`<op>` は `add`、`mul`、`sub`、`shr` などである。それぞれの意味論は次のとおりである。

- `checked_<op>` は、オーバーフローした場合に `None` を返す。
  
  例えば、`255u8.checked_add(1)` は `None` を返す。
- `overflowing_<op>` は、折り返された可能性のある結果と、オーバーフローが起きたかどうかを示す真偽値の両方を返す。
  
  例えば、`255u8.overflowing_add(1)` は `(0, true)` を返す。
- `wrapping_<op>` は、常に折り返された結果を返す。
  
  例えば、`255u8.wrapping_add(1)` は `0` を返す。
- `saturating_<op>` は、常に飽和した結果を返す。
  
  例えば、`255u8.saturating_add(1)` は `255` を返す。

ラッパー型は（`std::num` の）`Wrapping<T>` と `Saturating<T>` であり、`T` は整数型である。前者はすべての算術演算にラップアラウンドの意味論を与え、後者は飽和の意味論を与える。値をいったんラップすれば、それ以降の演算はすべて指定した意味論で行われる。

```rust
#![allow(dead_code)]
use std::num::Wrapping;
use std::panic;

fn use_integer() {
    let x: u8 = 242;

    let result = panic::catch_unwind(|| {
        println!("{}", x + 50); // panics in debug, prints 36 in release.
    });
    if result.is_err() {
        println!("panic");
    }
    // always prints 36.
    println!("{}", x.overflowing_add(50).0);
    // always prints 36.
    println!("{}", x.wrapping_add(50));
    // always prints 36.
    println!("{}", Wrapping(x) + Wrapping(50));

    // always panics:
    let (res, c) = x.overflowing_add(50);
    let result = panic::catch_unwind(|| {
        if c {
            panic!("custom error");
        } else {
            println!("{}", res);
        }
    });
    if result.is_err() {
        println!("panic");
    }
}
```

<div markdown="1" class="reco" id="LANG-ARITH" type="Rule" title="オーバーフローの可能性を踏まえて適切な算術演算を使う">

算術演算がオーバーフローを起こしうる場合、通常の演算子を直接使ってはならない。
代わりに、`checked_<op>`、`overflowing_<op>`、`wrapping_<op>`、`saturating_<op>` といった専用のメソッド、または `Wrapping` や `Saturating` のような専用のラッパー型を使い、コンパイルプロファイルによらず明示的で一貫した動作を保証しなければならない。

</div>
{% endraw %}
