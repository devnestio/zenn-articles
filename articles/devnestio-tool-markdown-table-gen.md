---
title: "CSVをMarkdownテーブルに一発変換 — devnestioのMarkdown Table Generatorを作った話"
emoji: "📊"
type: "tech"
topics: ["claude", "ai", "devtools", "markdown", "cloudflare"]
published: false
---

## 一言で言うと

CSV・TSV・セミコロン区切りのデータを貼り付けるだけで、列揃え機能付きのMarkdownテーブルを即座に生成するブラウザツールです。行・列の追加削除、アライメント設定、ワンクリックコピーに対応しています。

👉 https://markdown-table-gen.pages.dev

devnestio（ https://devnestio.pages.dev ）は、英語圏の開発者向けに無料ツールを少しずつ公開しているサイトです。今回はMarkdown Table Generatorについてbuilding-in-publicで書いてみます。

## なぜ作ったのか

READMEやドキュメントにMarkdownテーブルを書くとき、手でパイプ文字を揃えるのが面倒です。スプレッドシートからCSVをコピーしてすぐMarkdownに変換できるツールがあれば便利だと思って作りました。

## 主な機能

- **マルチセパレータ対応**: カンマ（CSV）/ タブ（TSV）/ セミコロン / パイプから選択
- **列アライメント設定**: 列ごとに Left / Center / Right / None を指定
- **セルエディタ**: ブラウザ上でそのまま表を編集できるインタラクティブエディタ
- **行・列の追加削除**: ボタン一つで行・列を末尾に追加、または削除
- **パディング出力**: 列幅を揃えたきれいなMarkdownテーブルを生成
- **ワンクリックコピー**: 生成されたMarkdownをクリップボードへ即コピー

## 技術的なポイント

Markdownテーブルの列区切り（separator row）でアライメントを `---:` / `:---:` / `:---` / `---` の4パターンで表現します。セルのパディングは最長セルの長さを基準に各列の幅を計算して揃えています。

## まとめ

「CSVがあるのにMarkdownテーブルを手書きしている」という手間を完全になくせます。Excelやスプレッドシートからコピーしてそのまま貼り付け、生成されたMarkdownをREADMEに貼るだけです。
