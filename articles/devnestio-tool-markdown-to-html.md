---
title: "MarkdownをHTMLに変換するブラウザツールを作った（ライブプレビュー付き）"
emoji: "📝"
type: "tech"
topics: ["markdown", "html", "javascript", "devtools", "cloudflare"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に、**Markdown to HTML Converter** を追加しました。

👉 https://markdown-to-html-dev.pages.dev

左にMarkdownを書くと右にライブプレビューが表示されます。HTMLタブに切り替えるとシンタックスハイライト付きのHTML出力を確認でき、そのままコピーまたはダウンロードできます。

---

## 主な機能

### ライブプレビュー

入力のたびに即座にHTMLプレビューを更新します。ボタンを押す必要はありません。

### 対応するMarkdown記法

- 見出し（H1〜H6）
- **太字**、*イタリック*、***太字+イタリック***
- ~~取り消し線~~
- インラインコード・コードブロック（言語クラス付き）
- 順序なし・順序付きリスト
- ブロック引用
- テーブル（`<thead>` / `<tbody>` 付き）
- 水平線
- リンク・自動リンク・画像

### HTMLの取り出し

- **コピー** — クリップボードに生HTML文字列をコピー
- **ダウンロード** — `<!DOCTYPE html>` 付きの完全なHTMLファイルとして保存

---

## 技術的なポイント

外部ライブラリゼロ・単一HTMLファイル・サーバー送信なし。Node.js の `assert` で **102件** のテストを実装し全通過確認済みです。

---

## 使ってみてください

👉 **[Markdown to HTML Converter — devnestio](https://markdown-to-html-dev.pages.dev)**

全ツール一覧: https://devnestio.pages.dev
