---
title: "CSS Gridを視覚的に組み立てCSSを即出力 — devnestioのCSS Grid Generatorを作った話"
emoji: "⬛"
type: "tech"
topics: ["claude", "ai", "devtools", "css", "cloudflare"]
published: false
---

## 一言で言うと

CSS GridのすべてのプロパティをGUIで設定し、ライブプレビューを確認しながらCSSコードを生成するブラウザツールです。セルをクリックするとアイテムごとのスパン・配置を設定でき、コピーまたはダウンロードできます。

👉 https://css-grid-generator-arc.pages.dev

devnestio（ https://devnestio.pages.dev ）は、英語圏の開発者向けに無料ツールを少しずつ公開しているサイトです。今回はCSS Grid Generatorについてbuilding-in-publicで書いてみます。

## なぜ作ったのか

CSS Gridは強力なレイアウトシステムですが、プロパティが多く、組み合わせによって挙動がわかりにくいことがあります。FlexboxのPlaygroundと同じく、「設定して即プレビュー、そのままコードコピー」という体験を提供したくて作りました。

## 主な機能

- **コンテナプロパティ全対応**: `grid-template-columns` / `rows` / `column-gap` / `row-gap` / `justify-items` / `align-items` / `justify-content` / `align-content` / `grid-auto-flow`
- **ライブプレビュー**: 設定をリアルタイムで反映した実際のグリッドレイアウト
- **アイテム別設定**: プレビューのセルをクリックして `grid-column` / `grid-row` / `justify-self` / `align-self` を個別設定
- **CSS出力**: `.grid-container` と各アイテムの `:nth-child()` ルールを含む完全なCSSを生成
- **コピー & ダウンロード**: ワンクリックでクリップボードコピーまたは `grid.css` としてダウンロード

## 技術的なポイント

プレビューエリアは実際のCSS Gridを直接適用しているため、ブラウザのレンダリングエンジンが描画した「本物の」レイアウトを確認できます。`repeat(auto-fill, minmax(200px, 1fr))` のような複雑な値も入力欄に直接書けるので、自由度が高い設計にしました。

## まとめ

CSS Gridはレイアウトの問題をエレガントに解決できますが、最初の設定が難しく感じられることがあります。このGeneratorで試しながら覚えると、プロパティの意味を体感で理解できます。
