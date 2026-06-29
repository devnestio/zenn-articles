---
title: "LoremIpsumジェネレーターを作った（Latin/Tech/Business/Hipster 5種類対応）"
emoji: "🔤"
type: "tech"
topics: ["javascript", "devtools", "cloudflare", "webdev", "design"]
published: true
---

## 作ったもの

開発者向け無料ツール群「devnestio」に、**Lorem Ipsum Generator** を追加しました。

👉 https://lorem-ipsum-gen.pages.dev

段落・文・単語の単位でプレースホルダーテキストを生成できます。5種類のワードプールから選択可能です。

---

## バリアント一覧

| バリアント | 特徴 |
|-----------|------|
| **Latin** | 古典的な lorem ipsum（デフォルト） |
| **Cicero** | 「de Finibus Bonorum」原典に近いテキスト |
| **Business** | synergy, pivot, disruption, stakeholder... |
| **Tech** | kubernetes, microservice, graphql, lambda... |
| **Hipster** | artisan, craft, organic, bespoke, bussin... |

---

## 主な機能

- **段落 / 文 / 単語** の単位で生成
- 件数は 1〜100 で指定
- 「Lorem ipsum で始める」オプション
- **プレーンテキストコピー** / **`<p>` タグ付き HTML コピー**
- リアルタイムの単語数・文数・文字数カウント

---

## なぜ5種類？

モックアップのトーンに合ったプレースホルダーテキストが欲しい場面があります。Business バリアントはビジネス資料やプレゼンのモックに、Tech バリアントはドキュメントサイトのプレビューに、Hipster バリアントはカフェやライフスタイルブランドのデザインカンプに向いています。

---

## テスト

Node.js の `assert` で **89件** のテストを実装し全通過確認済みです。

---

## 使ってみてください

👉 **[Lorem Ipsum Generator — devnestio](https://lorem-ipsum-gen.pages.dev)**

全ツール一覧: https://devnestio.pages.dev
