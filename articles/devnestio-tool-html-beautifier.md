---
title: "HTMLビューティファイア＆ミニファイアをブラウザだけで作った話"
emoji: "✨"
type: "tech"
topics: ["html", "javascript", "devtools", "webdev", "cloudflare"]
published: false
---

## 作ったもの

ブラウザだけで動くHTML整形ツール。左ペインにHTMLを貼り付けて**Beautify**か**Minify**を押すだけ。サーバー送信なし・登録不要。

👉 https://html-beautifier-dev.pages.dev

## 機能

- **Beautify** — 2スペース・4スペース・タブを選んでインデント整形
- **Minify** — コメント削除・空白除去で最小サイズに圧縮
- **スワップ** — 出力を入力に戻してそのまま続けて加工
- **スタッツバー** — 文字数・行数・ファイルサイズ・削減率を表示
- **ダウンロード** — output.html として保存
- `<script>` `<style>` `<pre>` の内容はそのまま保持

## Minifyのロジック

```js
function minifyHTML(html) {
  return html
    .replace(/<!--(?![\s\S]*?<!\[)[\s\S]*?-->/g, '') // コメント除去
    .replace(/\s+/g, ' ')                              // 空白正規化
    .replace(/\s*(<[^>]+>)\s*/g, '$1')                // タグ前後の空白除去
    .replace(/>\s+</g, '><')                           // タグ間の空白除去
    .trim();
}
```

IE条件コメント（`<!--[if ...]>`）は意図的に保持しています。

## Beautifyのロジック

インデント深度をカウンターで管理し、ブロック要素・インライン要素・void要素を区別。`<pre>` / `<script>` / `<style>` 内は一切手を入れません。

## devnestio について

https://devnestio.pages.dev

フロントエンド・インフラ向けの無料ブラウザツール集。
