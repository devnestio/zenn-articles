---
title: "テキストケースコンバーターを作った — 15種の命名規則をリアルタイム変換"
emoji: "🔤"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "cloudflare"]
published: false
---

## どんなツール？

テキストを15種類の命名規則に一括変換できるWebツールです。入力と同時にリアルタイムで全フォーマットを表示します。

👉 https://text-case-converter-cjj.pages.dev

devnestio（[devnestio.pages.dev](https://devnestio.pages.dev)）で公開している開発者向け無料ツール群の一つです。

## 対応フォーマット

| フォーマット | 例 |
|---|---|
| camelCase | helloWorld |
| PascalCase | HelloWorld |
| snake_case | hello_world |
| kebab-case | hello-world |
| CONSTANT_CASE | HELLO_WORLD |
| Title Case | Hello World |
| Sentence case | Hello world |
| dot.case | hello.world |
| path/case | hello/world |
| flatcase | helloworld |
| UPPERFLATCASE | HELLOWORLD |
| Train-Case | Hello-World |
| COBOL-CASE | HELLO-WORLD |
| aLtErNaTiNg | HeLlO wOrLd |
| tOGGLE cASE | hELLO wORLD |

## なぜ作ったか

コードを書いていると変数名・ファイル名・APIキーなど、命名規則を変換する機会が頻繁にある。camelCase で書いたコードを snake_case のAPIに合わせたり、DBのカラム名をプログラム変数に変換したりと、手動変換は地味にストレスが溜まる。

## 技術

スペース・アンダースコア・ハイフン・ドット・スラッシュ、そして大文字境界をすべてトークン分割し、各フォーマットに再組み立てする純粋なVanilla JS実装。Cloudflare Pages でホスト。
