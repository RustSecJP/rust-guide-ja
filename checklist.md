{% raw %}
# 規則と推奨の一覧

この指針に含まれる規則と推奨を、章ごとに並べた一覧である。識別子を選ぶと、本文の該当箇所へ移動する。


## 開発環境

| 種類 | 識別子 | 内容 |
|---|---|---|
| 規則 | [`DENV-STABLE`](devenv.md#DENV-STABLE) | stable のコンパイルツールチェーンを使う |
| 規則 | [`DENV-TIERS`](devenv.md#DENV-TIERS) | 安全性が重要なソフトウェアでは `rustc` のティア 1 のみを使う |
| 規則 | [`DENV-CARGO-LOCK`](devenv.md#DENV-CARGO-LOCK) | Cargo.lock をバージョン管理システムで追跡する |
| 規則 | [`DENV-CARGO-OPTS`](devenv.md#DENV-CARGO-OPTS) | cargo プロファイルの重要な変数は既定値のままにする |
| 規則 | [`DENV-CARGO-ENVVARS`](devenv.md#DENV-CARGO-ENVVARS) | cargo の実行時はコンパイラの環境変数を既定値のままにする |
| 推奨 | [`DENV-FORMAT`](devenv.md#DENV-FORMAT) | Rust のフォーマッタ（rustfmt）を使う |
| 規則 | [`DENV-LINTER`](devenv.md#DENV-LINTER) | リンターを定期的に使う |
| 規則 | [`DENV-AUTOFIX`](devenv.md#DENV-AUTOFIX) | 自動修正を手作業で確認する |

## ライブラリ

| 種類 | 識別子 | 内容 |
|---|---|---|
| 規則 | [`LIBS-VETTING-DIRECT`](libraries.md#LIBS-VETTING-DIRECT) | 第三者による直接の依存の検証 |
| 推奨 | [`LIBS-VETTING-TRANSITIVE`](libraries.md#LIBS-VETTING-TRANSITIVE) | 第三者による推移的な依存の検証 |
| 規則 | [`LIBS-OUTDATED`](libraries.md#LIBS-OUTDATED) | 古くなった依存を確認する（cargo-outdated） |
| 規則 | [`LIBS-AUDIT`](libraries.md#LIBS-AUDIT) | 依存に関するセキュリティ脆弱性の報告を確認する（cargo-audit） |
| 推奨 | [`LIBS-UNSAFE`](libraries.md#LIBS-UNSAFE) | Check for unsafe code in dependencies |

## 命名

| 種類 | 識別子 | 内容 |
|---|---|---|
| 規則 | [`LANG-NAMING`](naming.md#LANG-NAMING) | 命名規約を守る |

## 整数演算

| 種類 | 識別子 | 内容 |
|---|---|---|
| 規則 | [`LANG-ARITH`](integer.md#LANG-ARITH) | オーバーフローの可能性を踏まえて適切な算術演算を使う |

## エラー処理

| 種類 | 識別子 | 内容 |
|---|---|---|
| 推奨 | [`LANG-ERRWRAP`](errors.md#LANG-ERRWRAP) | 起こりうるすべてのエラーをラップする独自の `Error` 型を実装する |
| 規則 | [`LANG-LIMIT-PANIC`](errors.md#LANG-LIMIT-PANIC) | `panic` の使用を制限する |
| 規則 | [`LANG-LIMIT-PANIC-SRC`](errors.md#LANG-LIMIT-PANIC-SRC) | パニックする関数の使用を制限する |
| 規則 | [`LANG-ARRINDEXING`](errors.md#LANG-ARRINDEXING) | 配列の添字アクセスを適切に検査するか、`get` メソッドを使う |

## 標準ライブラリ

| 種類 | 識別子 | 内容 |
|---|---|---|
| 規則 | [`LANG-SYNC-TRAITS`](standard.md#LANG-SYNC-TRAITS) | `Send` と `Sync` の実装を正当化する |
| 規則 | [`LANG-CMP-INV`](standard.md#LANG-CMP-INV) | 標準の比較トレイトの不変条件を守る |
| 推奨 | [`LANG-CMP-DEFAULTS`](standard.md#LANG-CMP-DEFAULTS) | 標準の比較トレイトではメソッドの既定の実装を使う |
| 推奨 | [`LANG-CMP-DERIVE`](standard.md#LANG-CMP-DERIVE) | 可能な限り比較トレイトを導出する |
| 規則 | [`LANG-DROP`](standard.md#LANG-DROP) | `Drop` の実装を正当化する |
| 規則 | [`LANG-DROP-NO-PANIC`](standard.md#LANG-DROP-NO-PANIC) | `Drop` の実装でパニックしない |
| 規則 | [`LANG-DROP-NO-CYCLE`](standard.md#LANG-DROP-NO-CYCLE) | 参照カウント方式の `Drop` の循環を許さない |
| 規則 | [`LANG-DROP-SEC`](standard.md#LANG-DROP-SEC) | セキュリティの確保を `Drop` だけに頼らない |
| 規則 | [`MEM-MUT-REC-RC`](standard.md#MEM-MUT-REC-RC) | 循環する参照カウントポインタを避ける |

## unsafe Rust: 概論

| 種類 | 識別子 | 内容 |
|---|---|---|
| 規則 | [`UNSAFE-NOUB`](unsafe/generalities.md#UNSAFE-NOUB) | 未定義動作の禁止 |
| 規則 | [`LANG-UNSAFE`](unsafe/generalities.md#LANG-UNSAFE) | unsafe ブロックを使わない |
| 規則 | [`LANG-UNSAFE-ENCP`](unsafe/generalities.md#LANG-UNSAFE-ENCP) | *unsafe* な機能のカプセル化 |

## unsafe Rust: メモリ管理

| 種類 | 識別子 | 内容 |
|---|---|---|
| 規則 | [`MEM-NO-LEAK`](unsafe/memory.md#MEM-NO-LEAK) | メモリリークの禁止 |
| 規則 | [`MEM-FORGET`](unsafe/memory.md#MEM-FORGET) | `mem::forget` を使わない |
| 推奨 | [`MEM-FORGET-LINT`](unsafe/memory.md#MEM-FORGET-LINT) | clippy のリントで `mem::forget` の使用を検出する |
| 規則 | [`MEM-LEAK`](unsafe/memory.md#MEM-LEAK) | `leak` 関数を使わない |
| 規則 | [`MEM-MANUALLYDROP`](unsafe/memory.md#MEM-MANUALLYDROP) | `ManuallyDrop` でラップした値を必ず解放する |
| 規則 | [`MEM-NORAWPOINTER`](unsafe/memory.md#MEM-NORAWPOINTER) | `unsafe` を使わない Rust でスマートポインタを生ポインタに変換しない |
| 規則 | [`MEM-INTOFROMRAWALWAYS`](unsafe/memory.md#MEM-INTOFROMRAWALWAYS) | `into_raw` した値には必ず `from_raw` を呼ぶ |
| 規則 | [`MEM-INTOFROMRAWONLY`](unsafe/memory.md#MEM-INTOFROMRAWONLY) | `from_raw` は `into_raw` した値に*のみ*呼ぶ |
| 規則 | [`MEM-UNINIT`](unsafe/memory.md#MEM-UNINIT) | 未初期化メモリを使わない |

## unsafe Rust: FFI

| 種類 | 識別子 | 内容 |
|---|---|---|
| 推奨 | [`FFI-SAFEWRAPPING`](unsafe/ffi.md#FFI-SAFEWRAPPING) | 外部ライブラリには安全なラッパーを用意する |
| 規則 | [`FFI-CTYPE`](unsafe/ffi.md#FFI-CTYPE) | FFI では C 互換の型だけを使う |
| 規則 | [`FFI-TCONS`](unsafe/ffi.md#FFI-TCONS) | FFI の境界では一貫した型を使う |
| 推奨 | [`FFI-AUTOMATE`](unsafe/ffi.md#FFI-AUTOMATE) | バインディングの自動生成ツールを使う |
| 規則 | [`FFI-PFTYPE`](unsafe/ffi.md#FFI-PFTYPE) | プラットフォーム依存の型にバインドするときは移植性のあるエイリアス `c_*` を使う |
| 規則 | [`FFI-CKNONROBUST`](unsafe/ffi.md#FFI-CKNONROBUST) | 検査していない堅牢でない外部の値を使わない |
| 推奨 | [`FFI-CKINRUST`](unsafe/ffi.md#FFI-CKINRUST) | 外部の値は Rust で検査する |
| 規則 | [`FFI-CK-PTR-VALID`](unsafe/ffi.md#FFI-CK-PTR-VALID) | 外部のポインタを検査する |
| 推奨 | [`FFI-INPUT-PTR`](unsafe/ffi.md#FFI-INPUT-PTR) | 外部の言語から来るポインタは生ポインタで表現する |
| 規則 | [`FFI-CK-INPUT-REF-VALID`](unsafe/ffi.md#FFI-CK-INPUT-REF-VALID) | 検査していない外部の参照を使わない |
| 規則 | [`FFI-MARKEDFUNPTR`](unsafe/ffi.md#FFI-MARKEDFUNPTR) | FFI の関数ポインタ型には `extern` と `unsafe` を付ける |
| 規則 | [`FFI-CKFUNPTR`](unsafe/ffi.md#FFI-CKFUNPTR) | 外部の関数ポインタを検査する |
| 規則 | [`FFI-NOENUM`](unsafe/ffi.md#FFI-NOENUM) | FFI の境界で外から入ってくる Rust の `enum` を使わない |
| 推奨 | [`FFI-R-OPAQUE`](unsafe/ffi.md#FFI-R-OPAQUE) | 外部の不透明型には専用の Rust の型を使う |
| 推奨 | [`FFI-C-OPAQUE`](unsafe/ffi.md#FFI-C-OPAQUE) | 型を不透明にするには不完全な C/C++ の `struct` へのポインタを使う |
| 規則 | [`FFI-CK-REF-MODEL`](unsafe/ffi.md#FFI-CK-REF-MODEL) | 境界で間接参照を受け渡すときの Rust のメモリモデルの保持 |
| 規則 | [`FFI-MEM-NODROP`](unsafe/ffi.md#FFI-MEM-NODROP) | FFI の境界で `Drop` を実装する型を使わない |
| 規則 | [`FFI-MEM-OWNER`](unsafe/ffi.md#FFI-MEM-OWNER) | FFI ではデータの所有権を明確にする |
| 推奨 | [`FFI-MEM-WRAPPING`](unsafe/ffi.md#FFI-MEM-WRAPPING) | 外部のデータをメモリを解放するラッパーで包む |
| 推奨 | [`FFI-NOPANIC`](unsafe/ffi.md#FFI-NOPANIC) | FFI では `panic!` を正しく扱う |
| 規則 | [`FFI-CAPI`](unsafe/ffi.md#FFI-CAPI) | 専用の C 互換 API だけを公開する |
{% endraw %}
