---
title: "Font AwesomeとMaterial Iconsを一覧検索 — devnestioのIcon Font Viewerを作った話"
emoji: "🎯"
type: "tech"
topics: ["claude", "ai", "devtools", "css", "cloudflare"]
published: false
---

## 一言で言うと

Font Awesome 6とMaterial Iconsのアイコンをブラウザ上で検索・プレビューし、CSSクラス名やHTMLスニペットをワンクリックでコピーできるツールです。

👉 https://icon-font-viewer.pages.dev

devnestio（ https://devnestio.pages.dev ）は、英語圏の開発者向けに無料ツールを少しずつ公開しているサイトです。今回はIcon Font Viewerについてbuilding-in-publicで書いてみます。

## なぜ作ったのか

Font AwesomeやMaterial Iconsのアイコンを探すとき、公式サイトを開いて検索するのが面倒なことがあります。また、`fa-solid fa-arrow-up` なのか `fa-arrow-up` だけでいいのかなど、正確なクラス名をすぐに手に入れたい場面があってツール化しました。

## 主な機能

- **2つのライブラリ対応**: Font Awesome 6（Solid + Brands）と Material Icons を切り替え
- **名前検索**: アイコン名の一部を入力してリアルタイムフィルタリング
- **カテゴリフィルタ**: Arrows / Developer / Media / Social / UI など複数カテゴリで絞り込み
- **プレビュー付きグリッド**: 90px幅のカードでアイコンと名前を一覧表示
- **コピー機能**: クリックでモーダルが開き、CSSクラス名またはHTMLをコピー

## 技術的なポイント

Font Awesomeの場合、ブランドアイコン（`fa-brands fa-github`）と通常アイコン（`fa-solid fa-code`）でクラス名の形式が違うため、アイコンデータに `brands fa-` プレフィックスを持たせて分岐処理しています。Material Iconsは `<span class="material-icons">home</span>` の形式なので、アイコン名がそのままテキストコンテンツになります。

## まとめ

「`fa-` で始まるクラス名を正確に知りたい」「`fa-solid` か `fa-regular` か確認したい」という日常的なニーズに応えるツールです。
