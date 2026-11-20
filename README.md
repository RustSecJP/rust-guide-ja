# Rust で安全なアプリケーションを開発するための指針
![badge_repo](https://img.shields.io/badge/RustSecJP-rust--guide--ja-white)
[![badge_original](https://img.shields.io/badge/原文-ANSSI--FR%2Frust--guide-%23e9c7e7)](https://github.com/ANSSI-FR/rust-guide)
[![badge_unofficial](https://img.shields.io/badge/日本語訳-非公式-blue)](https://rustsecjp.github.io/rust-guide-ja/)

> [!NOTE]
> これは [ANSSI-FR/rust-guide](https://github.com/ANSSI-FR/rust-guide) の**非公式な日本語訳**です。ANSSI による確認や公認を受けたものではなく、内容が食い違う場合は原文が優先されます。
> This is an unofficial Japanese translation, not endorsed by ANSSI.

## フランスのサイバーセキュリティ機関（ANSSI）

*原文のプロジェクトは [ANSSI](https://cyber.gouv.fr/) が管理しています。詳しくは、ANSSI のオープンソース戦略の[ページ](https://cyber.gouv.fr/enjeux-technologiques/open-source/)（フランス語）を見てください。*

## 目的

この文書の目的は、プログラミング言語 Rust を使って安全なアプリケーションを開発するための、手がかりと推奨を示すことです。

Rust のプログラムの書き方を教える講座を意図したものではありません。そのための優れた学習資料は、すでに数多くあります（下の「*外部の参考資料*」の節を見てください）。目的はむしろ、プログラマを導き、いくつかの落とし穴を知らせることにあります。とくに、セキュリティ上の要求が強いアプリケーションの開発に関わる場合に役立ちます。これらの推奨は、Rust という言語がすでに備えている高い信頼性を補うものです。とはいえ、分かりやすさのために基本の確認が必要なこともあります。経験を積んだ Rust プログラマは、強調された枠（*規則*、*推奨*、*警告* など）だけを頼りにしてもかまいません。

これは現在も作成が続いている版で、あらゆる貢献を歓迎します。

## 指針をオンラインで読む

[https://rustsecjp.github.io/rust-guide-ja](https://rustsecjp.github.io/rust-guide-ja/)

原文（英語・フランス語）: [https://anssi-fr.github.io/rust-guide](https://anssi-fr.github.io/rust-guide)

## 指針をビルドする

原文は、[原文のリポジトリ](https://github.com/ANSSI-FR/rust-guide)で次のようにビルドします。`mdbook` と、必要な前処理ツールをインストールします。

```
cargo install mdbook --git https://github.com/hg-anssi/mdBook.git --rev c5a35b9296c6d5e48570e30022bd69403050a9f4 --locked
cargo install --path ./mdbook-checklist --locked
cargo install --path ./mdbook-code-align --locked
cargo install --path ./mdbook-extensions --locked
```

次に、本をビルドして開きます。

```
$ mdbook serve -o
```

この日本語訳は GitHub Pages で表示しており、ビルドの作業は要りません。

## 貢献の呼びかけ

原文への貢献は、[原文の CONTRIBUTING.md](https://github.com/ANSSI-FR/rust-guide/blob/master/CONTRIBUTING.md) を見てください。日本語訳の誤訳の指摘や訳語の提案は、このリポジトリの Issue か Pull Request でお願いします。

## ライセンス

この文書は [Open Licence 2.0](https://github.com/ANSSI-FR/rust-guide/blob/master/LICENSE.md) で公開されています。日本語訳も同じライセンスで公開します。出典の詳細は [NOTICE.md](NOTICE.md) にあります。

## 外部の参考資料

- [The Rust programming language](https://www.rust-lang.org)
- [The Rust book](https://doc.rust-lang.org/stable/book)
- [The Edition Guide](https://doc.rust-lang.org/edition-guide/)
- [Rust API Guidelines](https://rust-lang.github.io/api-guidelines)

Rust と関連ツールについてのさまざまな書籍資料の最新の一覧は、[Rust のドキュメントのトップページ](https://doc.rust-lang.org)にもあります。
