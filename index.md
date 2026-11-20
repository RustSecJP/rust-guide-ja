# 安全な Rust アプリケーション開発の指針（ANSSI「Secure Rust Guidelines」日本語訳）

フランス国家情報システムセキュリティ庁（ANSSI）が公開している Rust のセキュアコーディング指針 *Secure Rust Guidelines*（Programming Rules to Develop Secure Applications with Rust）の、非公式な日本語訳です。全 11 章を省略せずに訳し、61 件の規則と推奨を一覧にしています。安全なアプリケーションを Rust で開発するための規則と推奨をまとめた文書で、開発環境、ライブラリの選び方、言語の使い方、`unsafe` と FFI、標準ライブラリを扱います。

> **ご注意**: この翻訳は ANSSI の確認や公認を受けたものではありません。英語版をもとに訳していますが、ANSSI は英語版をフランス語版の参考訳と位置づけており、食い違う場合はフランス語版が正とされています。

## 章の一覧

{% for c in site.data.book.chapters %}
{{ forloop.index }}. [{{ c.title }}]({{ '/' | append: c.file | relative_url }})
{% endfor %}

## 読み方

本文には、次の 2 種類の枠が出てきます。

- **規則**: 守らなければならないこと。
- **推奨**: 守ることが望ましいこと。

それぞれに `LANG-ARITH` のような識別子が付いています。識別子は原文と同じです。

原文で未執筆の章（マクロ、型システム、テストとファジング）は、訳していません。原文ではビルド時に生成される「規則と推奨の一覧」は、訳文から作り直しています。

## 出典

- 原文: [ANSSI-FR/rust-guide](https://github.com/ANSSI-FR/rust-guide)
- 原著者: ANSSI（Agence nationale de la sécurité des systèmes d'information）
- ライセンス: [Open Licence 2.0 / Licence Ouverte 2.0](https://github.com/ANSSI-FR/rust-guide/blob/master/LICENSE.md)
- 訳した版: 2026-05-18 時点の英語版（コミット `3f9e2e2`）
