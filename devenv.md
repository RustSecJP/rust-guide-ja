{% raw %}
# 開発環境

## Rustup

[Rustup] は Rust ツールチェーンのインストーラである。とりわけ、ツールチェーンの
種類（stable、beta、nightly）の切り替え、追加コンポーネントのインストール管理、
およびそれらの更新を行うことができる。

<div markdown="1" class="warning">

セキュリティの観点では、`rustup` はすべてのダウンロードを HTTPS 経由で行うが、
ダウンロードしたものの署名はまだ検証しない。ダウングレード攻撃への対策、
証明書のピン留め、署名の検証は、いまだ作業中である。
場合によっては、Rust 公式ウェブサイトの *Install* の節に掲載されている
別のインストール方法を選ぶほうが望ましいこともある。

</div>

[rustup]: https://github.com/rust-lang/rustup.rs

### Rust のエディション

Rust 言語には、*エディション* と呼ばれる複数の種類が共存している。

エディションという概念は、新機能の導入を明確にし、それを段階的に行えるようにするために
導入された。[Rust Manual](https://doc.rust-lang.org/edition-guide/) に記載されているとおり、新しいエディションは
2 年か 3 年ごとに作られるが、これは新機能や改良が新しいエディションでしか提供されない
という意味ではない。

エディションによっては、新しいキーワードや言語構文が追加される。そのため、安全な
アプリケーション開発のための推奨事項は、Rust のエディションよりも、そうした
アプリケーションで使われる言語機能のほうに密接に結び付いている。
本ガイドの以降の部分では、特定の Rust エディションに固有の構文や言語機能を、
できる限り明示するよう努める。

<div markdown="1" class="note">

選択したエディションが提供する機能に関する推奨事項に従う限り、特定のエディションを
推奨することはしない。

</div>

[edition guide]: https://doc.rust-lang.org/edition-guide/


### stable、nightly、beta のツールチェーン

Rust 言語の種類（機能の集合）を選ぶエディションの選択とは別に、Rust ツールチェーンは
*リリースチャネル* と呼ばれる 3 つの異なる版で提供されている。

- *nightly* リリースは 1 日に 1 回作成される。
- *beta* リリースは 6 週間ごとに、昇格した *nightly* リリースから作成される。
- *stable* リリースは 6 週間ごとに、昇格した *beta* リリースから作成される。

複数のツールチェーンを使い分ける際は、既定のツールチェーンが何であるかだけでなく、
一部のディレクトリに対して上書き（override）が設定されていないかも確認することが重要である。

<div markdown="1" class="example">

```shell
$ pwd
/tmp/foo
$ rustup toolchain list
stable-x86_64-unknown-linux-gnu (default)
beta-x86_64-unknown-linux-gnu
nightly-x86_64-unknown-linux-gnu
$ rustup override list
/tmp/foo                                    nightly-x86_64-unknown-linux-gnu
$
```

この例は、ツールチェーン `stable-x86_64-unknown-linux-gnu` が既定のツールチェーンであるが、
ディレクトリ `/tmp/foo` では `nightly-x86_64-unknown-linux-gnu` で上書きされていることを示している。

</div>

<div markdown="1" class="reco" id="DENV-STABLE" type="Rule" title="stable のコンパイルツールチェーンを使う">

安全なアプリケーションの開発は、コンパイラ、ランタイム、ツールに潜みうるバグを
抑えるため、完全に stable なツールチェーンを使って行わなければならない。

</div>

nightly のコンポーネントを必要とする特定の `cargo` サブコマンドを使う場合は、
ツールチェーン全体を明示的に切り替えるのではなく、その場限りでツールチェーンを
切り替えて実行するほうが望ましい。

<div markdown="1" class="example">

たとえば、（nightly の）最新の `rustfmt` を実行するには次のようにする。

```shell
$ rustup toolchain list
stable-x86_64-unknown-linux-gnu (default)
beta-x86_64-unknown-linux-gnu
nightly-x86_64-unknown-linux-gnu
$ rustup run nightly cargo fmt
$ # または
$ cargo +nightly fmt
$
```

</div>

### Rustc のティア保証

Rustc のバックエンドは [LLVM] に基づいているため、その対応ターゲットを引き継いでおり、各バックエンドがどの程度の安定性とテストを受けているかを示すために、ターゲットをティア（階層）に分類している。

[LLVM]: https://llvm.org/

<div markdown="1" class="example">

Rust のターゲットの一覧は、次のコマンドで得られる。

```shell
$ rustup target list
...
wasm32-unknown-emscripten
wasm32-unknown-unknown (installed)
wasm32-wasip1
...
x86_64-unknown-illumos
x86_64-unknown-linux-gnu (installed)
x86_64-unknown-linux-gnux32
...

```

ここでは、ターゲット `wasm32-unknown-unknown` と `x86_64-unknown-linux-gnu` がインストールされている。

</div>

#### ティア 1 - 動作が保証される

このターゲットは、コミュニティによって十分に検証されている。テストスイート全体に合格し、定期的な回帰テストを受け、新しいリリースに合わせて最新の状態に保たれる。実際上、一貫したコード生成、安定した ABI[^abi]、予測可能な性能を当てにすることができる。ティア 1 のターゲットは「動作が保証される」ものと考えてよい。

[^abi]: Application Binary Interface

#### ティア 2 - ビルドが保証される

このターゲットのソースコードは問題なくコンパイルできるが、ティア 1 と同じ水準のテストや保守は受けていない。テストスイートによる網羅が不完全なことがあり、一部の最適化や [LLVM] の新しい機能が欠けていたり不安定であったりする可能性がある。利用者はこれらのバックエンド向けにコードを生成できるものの、ときおり壊れることや、手作業でのパッチが必要になることを想定しておくべきである。ティア 2 のターゲットは「ビルドが保証される」ものと考えてよいが、「動作が保証される」ものではない。

#### ティア 3

ティア 3 のターゲットは、単に公式にはサポートされていない。



ティアの区別は、開発者が自らのリスク許容度に合ったターゲットを選ぶ助けとなる。すなわち、本番運用水準の用途にはティア 1 を、完全なサポートがまだ整っていない実験的または特殊なアーキテクチャにはティア 2 を選ぶ。

<div markdown="1" class="reco" id="DENV-TIERS" type="Rule" title="安全性が重要なソフトウェアでは `rustc` のティア 1 のみを使う">

安全性が重要なシステムでは、Rustc のティア 1 のターゲットと認証済みのツールチェーンを使わなければならない。

</div>

サポートされているターゲットの網羅的な一覧は、[Rust Book](https://doc.rust-lang.org/stable/rustc/index.html) に掲載されている。

## Cargo

Rustup で適切な Rust ツールチェーンを用意すると、コマンドラインプログラム `cargo` を
通じて [Cargo] が使えるようになる。Cargo は Rust のパッケージマネージャである。
ほとんどの Rust 開発において、次のような基本的な役割を担う。

- プロジェクトのひな形を提供して、プロジェクトを構造化する（`cargo new`）。
- プロジェクトをコンパイルする（`cargo build`）。
- プロジェクトのドキュメントを生成する（`cargo doc`）。
- テスト（`cargo test`）とベンチマーク（`cargo bench`）を実行する。
- 依存を管理し、ダウンロードする。
- パッケージを配布可能にし、[`crates.io`] に公開する。
- 後述するような補助的なツールを、サブコマンドの形で実行するための
  フロントエンドにもなる。

Cargo は、コンパイルの前に依存を自動的に解決し、その際に依存のチェックサムを検査する。
`Cargo.lock` ファイルにはすべての依存のチェックサムが含まれており、
ダウンロードしたもののチェックサムと比較される。
違いが検出されると、コンパイルは失敗する。

```
error: checksum for `sha256 v1.6.0` changed between lock files

this could be indicative of a few possible errors:

    * the lock file is corrupt
    * a replacement source in use (e.g., a mirror) returned a different checksum
    * the source itself may be corrupt in one way or another

unable to verify that `sha256 v1.6.0` is the same as when the lockfile was generated
```

`Cargo.lock` ファイルが無い場合は、最初のビルド時に生成され、依存のチェックサムが
記録される。これは TOFU（*Trust On First Use*、初回利用時に信頼する）の方針に従っている。
このファイルは `cargo generate-lockfile` で手動で作成することもできる。すでに存在する場合は、
各クレートの利用可能な最新バージョンで上書きされる。

<div markdown="1" class="reco" id="DENV-CARGO-LOCK" type="Rule" title="Cargo.lock をバージョン管理システムで追跡する">

`Cargo.lock` ファイルは、バージョン管理システムで追跡しなければならない。

</div>

<div markdown="1" class="warning">

（TOFU の原則に従う）*初回ダウンロード時* のクレートをどのように保護し検証するのが
最善かについては、議論が続いている。
現時点では、初回ダウンロードのセキュリティは、ウェブサイト [`crates.io`] と、
レジストリのインデックスを収めた GitHub 上のリポジトリのセキュリティが
良好であることに依存している。場合によっては、依存について別のインストール方法を
選ぶほうが望ましいこともある。

</div>

Cargo は、主にマニフェストファイル `Cargo.toml` を通じて、ビルドの過程をプロジェクトの
必要に合わせるための多くのコマンドとオプションを提供している。全体の説明については、
[Cargo Book](https://doc.rust-lang.org/cargo/index.html) を参照のこと。

安全なアプリケーションの開発では、一部のオプションに注意が必要となることがある。

Cargo は *プロファイル* という概念を提供している。`dev`、`release`、`test`、`bench` の
4 つのプロファイルが組み込まれており、`cargo` のコマンドに応じて適用される。

たとえば、`cargo build` は `dev` プロファイルを使い、`cargo build --release` や `cargo install` は
`release` プロファイルを使う。

プロファイルは `[profile.*]` セクションで作成・変更できる。たとえば次のものがある。

- 変数 [`debug-assertions`] は、デバッグアサーションを有効にするかどうかを制御する。
- 変数 [`overflow-checks`] は、整数演算でオーバーフローを検査するかどうかを
  制御する。

既定のオプションを上書きすると、通常は実行時の検査を有効にする `dev` プロファイルを
使っていても、バグが検出されなくなることがある（たとえば
[整数オーバーフローの検査](integer.md#chapter-integer)が有効にならない）。

<div markdown="1" class="reco" id="DENV-CARGO-OPTS" type="Rule" title="cargo プロファイルの重要な変数は既定値のままにする">

変数 [`debug-assertions`] と [`overflow-checks`] は、開発用プロファイルのセクション
（`[profile.dev]` と `[profile.test]`）で上書きしてはならない。

</div>

Cargo には、その設定を行い、特定のシステム上での動作を変えるための方法がほかにもある。
これは非常に便利な場合があるが、実際に使われているオプション、とりわけコンパイラに
渡されているオプションのすべてを、ある時点で把握し覚えておくことが難しくなることもある。
結果として、ビルドの過程の信頼性と堅牢性に影響しうる。コンパイラのオプションとフラグは、
設定ファイル `Cargo.toml` に集約するほうが望ましい。たとえば環境変数 `RUSTC_WRAPPER` は、
Rust のコンパイルの前にコードの一部を生成したり外部ツールを実行したりするために
使われることがあるが、その場合は Cargo のビルドスクリプトの機能を使うほうが望ましい。

<div markdown="1" class="reco" id="DENV-CARGO-ENVVARS" type="Rule" title="cargo の実行時はコンパイラの環境変数を既定値のままにする">

Cargo を使ってプロジェクトをビルドする際は、環境変数 `RUSTC`、`RUSTC_WRAPPER`、
`RUSTFLAGS` を上書きしてはならない。

</div>

[`crates.io`]: https://crates.io
[cargo]: https://doc.rust-lang.org/stable/cargo/
[`debug-assertions`]: https://doc.rust-lang.org/cargo/reference/profiles.html#debug-assertions
[`overflow-checks`]: https://doc.rust-lang.org/cargo/reference/profiles.html#overflow-checks

### Rustfmt

[Rustfmt] は Rust に同梱されているツールで、スタイルの指針に従ってコードを整形する。

この指針は、プロジェクトのルートに `rustfmt.toml` または `.rustfmt.toml` ファイルを
作成することで、必要に応じて調整できる。このファイルは既定の設定を上書きするために
使われる。たとえば次のとおりである。

```toml
# 1 行の最大幅を 120 に設定する
max_width = 120
# 1 行で書く if-else 式の最大の行長
single_line_if_else_max_width = 40
```

`rustfmt` が検査する指針の詳細については、
[Rust Style Guidelines Book](https://doc.rust-lang.org/style-guide/index.html) を参照のこと。

<div markdown="1" class="reco" id="DENV-FORMAT" type="Recommendation" title="Rust のフォーマッタ（rustfmt）を使う">

コードベースがスタイルの指針に従っていることを確実にするため、ツール `rustfmt` を
使うべきである。

</div>

[rustfmt]: https://github.com/rust-lang/rustfmt

### Cargo fix

[`cargo fix`] コマンドは、コンパイラの警告を修正することと、エディション間の移行を
容易にすることに特化したツールである。

```shell
$ cargo fix
```

Rust 2021 のプロジェクトを Rust 2024 へ移行する準備をするには、次を実行する。

```shell
$ cargo fix --edition
```

Rustfix は、コードを Rust 2024 と互換になるよう修正するか、問題を説明する警告を
表示する。後者の場合、その問題は手作業で修正する必要がある。
警告が無くなるまでこのコマンドを実行する（必要に応じて一部の問題を手作業で修正する）
ことで、コードが Rust 2021 と Rust 2024 の両方と互換であることを確実にできる。

Rust 2024 へ完全に切り替えるには、次を実行すればよい。

```shell
$ cargo fix --edition-idioms
```

このツールは、提案する修正の健全性についてほとんど保証しないことに注意されたい。
とりわけ、一部の修正（`--edition-idioms` で提供されるものの一部など）は、場合によって
コンパイルを壊したり、プログラムの意味を変えたりすることが知られている。

[`cargo fix`]: https://doc.rust-lang.org/cargo/commands/cargo-fix.html


### Clippy

[Clippy] は、多数のリント（バグ、スタイル、性能上の問題など）を提供し、検査するツール
である。バージョン 1.29 以降、`clippy` は stable の `rustup` 環境で使うことができる。
`clippy` は、プロジェクトの依存としてインストールするのではなく、stable ツールチェーンの
コンポーネントとしてインストールする（`rustup component add clippy`）ことを推奨する。

このツールには、検出しようとする問題の種類に応じたリントのカテゴリがいくつかある。
`clippy` が提案する修正をコミットする前に、プログラマは警告を再確認すべきである。
とりわけ `clippy::nursery` カテゴリのリントは、その指摘がまだ開発途上であるため、
注意が必要である。

現在の `clippy` には、`cargo fix` に似た `fix` ツールがある。

[clippy]: https://github.com/rust-lang/rust-clippy

<div markdown="1" class="reco" id="DENV-LINTER" type="Rule" title="リンターを定期的に使う">

安全なアプリケーションの開発中は、`clippy` などのリンターを定期的に
使わなければならない。

</div>

<div markdown="1" class="reco" id="DENV-AUTOFIX" type="Rule" title="自動修正を手作業で確認する">

安全な Rust 開発では、自動修正（たとえば `rustfix` や `clippy` が提供するもの）は
すべて、開発者が検証しなければならない。

</div>

### その他

特定のコードパターンを探したり、テストやファジングのための便利なコマンドを提供したり
することで、プログラムのセキュリティを確保するのに役立つツールや `cargo` サブコマンドが
ほかにも存在する。それらは、それぞれの目的に応じて、以降の章で取り上げる。
{% endraw %}
