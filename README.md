# Rust で安全なアプリケーションを開発するための指針（日本語訳）

> **これは [ANSSI-FR/rust-guide](https://github.com/ANSSI-FR/rust-guide) の非公式な日本語訳です。** ANSSI による確認や公認を受けたものではありません。内容が食い違う場合は原文が優先されます。
>
> **読むのはこちら: https://rustsecjp.github.io/rust-guide-ja/**
>
> This is an unofficial Japanese translation of ANSSI's *Secure Rust Guidelines*. It is not endorsed by ANSSI.

以下は、原文の README の日本語訳です。

---

## フランスのサイバーセキュリティ機関（ANSSI）

*このプロジェクトは [ANSSI](https://cyber.gouv.fr/) が管理しています。詳しくは、ANSSI のオープンソース戦略の[ページ](https://cyber.gouv.fr/enjeux-technologiques/open-source/)（フランス語）を見てください。*

## 目的

この文書の目的は、プログラミング言語 Rust を使って安全なアプリケーションを開発するための、手がかりと推奨を示すことです。

Rust のプログラムの書き方を教える講座を意図したものではありません。そのための優れた学習資料は、すでに数多くあります（下の「外部の参考資料」の節を見てください）。目的はむしろ、プログラマを導き、いくつかの落とし穴を知らせることにあります。とくに、セキュリティ上の要求が強いアプリケーションの開発に関わる場合に役立ちます。これらの推奨は、Rust という言語がすでに備えている高い信頼性を補うものです。とはいえ、分かりやすさのために基本の確認が必要なこともあります。経験を積んだ Rust プログラマは、強調された枠（*規則*、*推奨*、*警告* など）だけを頼りにしてもかまいません。

これは現在も作成が続いている版で、あらゆる貢献を歓迎します。

## 指針をオンラインで読む

- 原文（英語・フランス語）: [https://anssi-fr.github.io/rust-guide](https://anssi-fr.github.io/rust-guide)
- 日本語訳: [https://rustsecjp.github.io/rust-guide-ja/](https://rustsecjp.github.io/rust-guide-ja/)

## 指針をビルドする

原文は `mdbook` と専用の前処理ツールでビルドします。手順は[原文の README](https://github.com/ANSSI-FR/rust-guide#building-the-guide) を見てください。

この日本語訳は、GitHub Pages（Jekyll）で表示しています。原文のビルド時に行われるコード例の取り込みと「規則と推奨の一覧」の生成は、訳文の側であらかじめ展開してあります。

## 貢献の呼びかけ

- 原文への貢献: [原文の CONTRIBUTING.md](https://github.com/ANSSI-FR/rust-guide/blob/master/CONTRIBUTING.md) を見てください。
- 日本語訳への貢献: 誤訳の指摘や訳語の提案は、このリポジトリの Issue か Pull Request でお願いします。

## ライセンス

原文は [Open Licence 2.0](https://github.com/ANSSI-FR/rust-guide/blob/master/LICENSE.md) で公開されています。この日本語訳も同じライセンスで公開します。出典の詳細は [NOTICE.md](NOTICE.md) にあります。

## 外部の参考資料

- [The Rust programming language](https://www.rust-lang.org)
- [The Rust book](https://doc.rust-lang.org/stable/book)
- [The Edition Guide](https://doc.rust-lang.org/edition-guide/)
- [Rust API Guidelines](https://rust-lang.github.io/api-guidelines)

Rust と関連ツールについてのさまざまな書籍資料の最新の一覧は、[Rust のドキュメントのトップページ](https://doc.rust-lang.org)にもあります。

---

RustSecJP は非公式のコミュニティです。ANSSI、RustSec、Rust Project、Rust Foundation とは提携していません。
