---
title: "ブラウザだけで Markdown → HTML 変換ツールを作った (Markdown to HTML)"
emoji: "📝"
type: "tech"
topics: ["javascript", "markdown", "webdev", "tools"]
published: false
---

ライブプレビュー・シンタックスハイライト・HTML サニタイズ機能付きの Markdown → HTML 変換ツールを作りました。

**ツール:** https://devnestio.pages.dev/markdown-to-html/

## 機能

- ライブプレビュー（300ms デバウンス）
- HTML ソース表示タブ（シンタックスハイライト付き）
- コードブロック（fenced code / インライン）
- テーブル / ブロッククォート / 箇条書き / 番号付きリスト
- 太字・斜体・取り消し線・リンク・画像
- HTML サニタイズオプション
- 完全な HTML ドキュメントとしてダウンロード

## 実装のポイント

### コードブロック保護

インライン処理前にコードブロックをプレースホルダーに退避させ、後で復元する。

```js
function parseMarkdown(md) {
  const codeBlocks = [];
  md = md.replace(/```([\w]*)\n([\s\S]*?)```/gm, (_, lang, code) => {
    codeBlocks.push(`<pre><code class="language-${lang}">${esc(code)}</code></pre>`);
    return `\x00CODE${codeBlocks.length - 1}\x00`;
  });
  // ... 見出し、テーブル、リストの処理 ...
  return html.replace(/\x00CODE(\d+)\x00/g, (_, i) => codeBlocks[+i]);
}
```

### テーブル正規表現

```js
const TABLE_RE = /^\|(.+)\|\s*\n\|[-| :]+\|\s*\n((?:\|.+\|\s*\n?)*)/gm;
```

### インラインフォーマット

```js
function inlineFormat(text, esc) {
  if (esc) text = text.replace(/&/g,'&amp;').replace(/</g,'&lt;');
  text = text.replace(/\*\*\*(.+?)\*\*\*/g, '<strong><em>$1</em></strong>');
  text = text.replace(/\*\*(.+?)\*\*/g,     '<strong>$1</strong>');
  text = text.replace(/\*(.+?)\*/g,         '<em>$1</em>');
  text = text.replace(/~~(.+?)~~/g,         '<del>$1</del>');
  return text;
}
```

## テスト

78 テストすべてパス。ブラウザ内のみで処理され、データは外部に送信されません。

[DevNestio](https://devnestio.pages.dev) — ブラウザだけで動く開発者ツール集
