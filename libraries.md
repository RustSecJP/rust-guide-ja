{% raw %}
# ライブラリ

## 依存のリポジトリ

外部ライブラリの管理は `Cargo` ツールに統合されている。これらのライブラリの取得元を指定する方法はいくつかあり、その一部を以下に示す。

これらのライブラリのバージョンを正確に追跡することが、Rust で書かれたソフトウェアのセキュリティにとって極めて重要な条件であることに注意されたい。この要求は、規則 [DENV-CARGO-LOCK](devenv.md#DENV-CARGO-LOCK) として具体化されている。

### クレート

Rust は標準ライブラリに加えて、`cargo` によって、ほかのライブラリをプロジェクトに
簡単に取り込む方法を提供している。
Rust エコシステムで *クレート* と呼ばれるこれらのライブラリは、オープンソースの
コンポーネントを集めた中央リポジトリから取り込まれる。

`Cargo.toml` ファイルでの依存の宣言の例を示す。

```toml
[dependencies]
mdbook = { version = "0.4.52" }
anyhow = "1.0.99"
clap = { version = "4.5.47", features = ["derive"] }
markdown = { version = "1.0.0", features = ["serde"] }
semver = "1.0.26"
serde_json = "1.0.143"
serde = "1.0.219"
```

既定のリポジトリは [`crates.io`] である。[独自のレジストリ](https://doc.rust-lang.org/cargo/reference/registries.html)を使うこともできる。

[`crates.io`]: https://crates.io

### Git による依存

`Cargo.toml` ファイルの各依存は、[GIT リポジトリ](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#specifying-dependencies-from-git-repositories)を参照することもできる。たとえば次のとおりである。

```toml
[dependencies]
regex = { git = "https://github.com/rust-lang/regex.git" }
```

ブランチ、タグ、コミットハッシュのいずれかを指定することで、使いたいバージョンをより詳しく指定できる。

[依存をロックする仕組み](devenv.md#cargo)は、GIT リポジトリの場合にも働く。依存が特定のコミットを指定していない場合、最初のコンパイル時に、`Cargo.toml` ファイルの条件に合う最も新しいコミットが取得され、`Cargo.lock` ファイルに記録される。以降のコンパイルではすべて同じコミットが使われる（`Cargo.lock` ファイルを更新した場合を除く）。

## 依存のセキュリティ

依存の取得方法（*クレート* か GIT のコミットか）にかかわらず、外部の組織に由来する依存は検証しなければならない。

<div markdown="1" class="reco" id="LIBS-VETTING-DIRECT" type="Rule" title="第三者による直接の依存の検証">

第三者による直接の依存は、それぞれ適切に検証しなければならず、各検証は追跡できるように記録しなければならない。

</div>

推移的な依存についても、個別に検証することを推奨する。

<div markdown="1" class="reco" id="LIBS-VETTING-TRANSITIVE" type="Recommendation" title="第三者による推移的な依存の検証">

第三者による依存は、それぞれ適切に検証すべきであり、各検証は追跡できるように記録すべきである。

</div>

## 依存の検証ツール

### Cargo-outdated

[cargo-outdated] ツールを使うと、依存のバージョンを簡単に管理できる。

このツールは、指定したクレートについて、（`Cargo.toml` を使って）依存の現在の
バージョンを一覧表示し、互換性のある最新バージョンと、全体での最新バージョンを
確認する。

<div markdown="1" class="reco" id="LIBS-OUTDATED" type="Rule" title="古くなった依存を確認する（cargo-outdated）">

依存の状態を確認するため、`cargo-outdated` ツールを使わなければならない。そのうえで、
古くなった依存はそれぞれ更新すべきであり、そうしない場合はそのバージョンを選んだ
理由を示さなければならない。

</div>

[cargo-outdated]: https://github.com/kbknapp/cargo-outdated

### Cargo-audit

[Cargo-audit] ツールを使うと、RustSec Advisory Database に報告されたセキュリティ上の
脆弱性を簡単に確認できる。

<div markdown="1" class="reco" id="LIBS-AUDIT" type="Rule" title="依存に関するセキュリティ脆弱性の報告を確認する（cargo-audit）">

依存に含まれる既知の脆弱性を確認するため、`cargo-audit` ツールを
使わなければならない。

</div>

[cargo-audit]: https://crates.io/crates/cargo-audit

<!-- ## Unsafe code in libraries -->

<!--
<mark>TODO</mark>: `unsafe` blocks are discussed in the following chapter.
One needs to ensure that this kind of block is not misused in project
dependencies.
-->

<!--
<div markdown="1" class="reco" id="LIBS-UNSAFE" type="Recommendation" title="Check for unsafe code in dependencies">

<mark>TODO</mark>: check that no `unsafe` blocks appear in the imported
dependencies (with a tool?).

</div>

-->
{% endraw %}
