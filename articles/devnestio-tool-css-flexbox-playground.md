---
title: "CSS Flexboxを視覚的に学べるライブエディタ — devnestioのCSS Flexbox Playgroundを作った話"
emoji: "📐"
type: "tech"
topics: ["claude", "ai", "devtools", "css", "cloudflare"]
published: false
---

## 一言で言うと

CSS Flexbox のすべてのプロパティをインタラクティブに操作できるライブエディタです。プロパティを変更するとリアルタイムでレイアウトが変わり、生成されたCSSをそのままコピーできます。

👉 https://css-flexbox-playground-abt.pages.dev

devnestio（ https://devnestio.pages.dev ）は、英語圏の開発者向けに無料ツールを少しずつ公開しているサイトです。今回はその中のひとつ、CSS Flexbox Playgroundについてbuilding-in-publicで書いてみます。

## なぜ作ったのか

Flexboxのプロパティは組み合わせが多く、「`justify-content: space-between` と `align-items: center` を同時にかけるとどうなる？」を素早く確認したい場面がよくあります。ドキュメントを読むより、実際に動かして理解する方が速いのでツール化しました。

## 主な機能

- **コンテナプロパティ**: flex-direction / justify-content / align-items / flex-wrap / gap をドロップダウンで切り替え
- **アイテム個別設定**: flex-grow / flex-shrink / flex-basis / align-self / order を各アイテムごとに設定
- **リアルタイムプレビュー**: プロパティを変えるたびにレイアウトが即時反映
- **CSS出力**: 設定した内容がそのままCSS形式でコピーできる

## 技術的なポイント

プレビューエリアは実際のFlexbox CSSを直接適用しているため、ブラウザのレンダリングエンジンが描画した「本物の」レイアウトを確認できます。チートシートではなく、実際の挙動を体験できるのが強みです。

## まとめ

Flexboxは一度コツをつかめば非常に強力です。このPlaygroundで試しながら覚えると、プロパティの組み合わせを体で理解できます。
