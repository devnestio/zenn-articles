---
title: "SVG最適化・圧縮ツールをリリース — svg-optimizer"
emoji: "🎨"
type: "idea"
topics: ["svg", "javascript", "webdev", "tools"]
published: true
---

## SVG Optimizer & Minifier

SVGファイルをブラウザ上で最適化・圧縮できる無料ツールをリリースしました。

🔗 **[svg-optimizer-dev.pages.dev](https://svg-optimizer-dev.pages.dev)**

## 最適化できる内容

- XMLコメント（`<!-- ... -->`）の削除
- `<title>`・`<desc>`・`<metadata>` ブロックの削除
- `<?xml?>` 処理命令・`<!DOCTYPE>` の削除
- 空白・改行の圧縮（1行出力）
- 空属性（`class=""`）の削除
- デフォルト属性（`fill="black"`・`opacity="1"` など）の削除
- 未使用IDの削除

## 効果の例

Before (420 B) → After (~110 B) で **約74%削減**、見た目は変わらず。

## 機能

- 最適化前後のプレビュー表示
- 削除バイト数・削減率の表示
- コピーボタン・サンプルSVGのロード

## devnestio について

🔗 [devnestio.pages.dev](https://devnestio.pages.dev)
