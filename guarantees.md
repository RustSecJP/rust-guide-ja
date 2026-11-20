{% raw %}

# 言語による保証 {#language-guarantees}

## 未定義動作（*UB*）

<div markdown="1" class="definition">

プログラムの意味論が Rust 言語で規定されていないとき、そのプログラムの動作は[*未定義*](https://doc.rust-lang.org/reference/behavior-considered-undefined.html)である。

</div>

UB の存在は[プログラミング上の誤り](https://doc.rust-lang.org/reference/behavior-considered-undefined.html#r-undefined.general)とみなされ、避けなければならない。

<div markdown="1" class="example">

ヌルポインタの参照外しは *UB* である。一方、`None` オブジェクトに対する `unwrap` の動作は明確に定義されている。このエラーは言語自身が（パニックを起こすことで）処理するからである。

</div>

UB につながるプログラミング上の誤りの一覧は、[Rust reference](https://doc.rust-lang.org/stable/reference/) に
[掲載されている](https://doc.rust-lang.org/reference/behavior-considered-undefined.html)。そのうち、特に注目すべきものは次のとおりである。

* 未割り当て、またはアラインメントが合っていないメモリアドレスを指すポインタ（ダングリングポインタ）を参照外ししないこと。これは次を意味する。
  * バッファオーバーフローがないこと
  * 解放済みメモリへのアクセスがないこと
  * アラインメントが合っていないアクセスがないこと
* ポインタの指す値が、ポインタの型と[整合している](https://doc.rust-lang.org/reference/behavior-considered-undefined.html#r-undefined.invalid)こと。例えば、真偽値へのポインタが指す値は、値が 1 か 0 のバイトである。
* [エイリアシング規則](https://doc.rust-lang.org/reference/behavior-considered-undefined.html#r-undefined.alias)を守ること（[例](https://doc.rust-lang.org/nomicon/aliasing.html)は [Rustonomicon](https://doc.rust-lang.org/stable/nomicon/) も参照）。可変参照は共有できない。
* 同じメモリアドレスへの[並行アクセス](https://doc.rust-lang.org/reference/behavior-considered-undefined.html#r-undefined.race)がないこと（書き込みと同時には、読み取りも書き込みもできない）（[例](https://doc.rust-lang.org/nomicon/races.html)は [Rustonomicon](https://doc.rust-lang.org/stable/nomicon/) も参照）

## Rust による保証 {#rust-guarantees}

<div markdown="1" class="important">

この言語の基本的な考え方は、Rust の *unsafe* でない部分だけを使ったプログラムに UB が存在しないことを保証する、というものである。

</div>

<div markdown="1" class="note">

こうしたメモリ安全性の保証があっても、この言語は次のものを防がない。

* リソースリーク（メモリ、入出力など）（[メモリ管理の節](unsafe/memory.md#chapter-memory)を参照）
* 数値のオーバーフロー（[整数演算の節](integer.md#chapter-integer)を参照）

</div>
{% endraw %}
