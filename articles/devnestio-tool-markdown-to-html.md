---
title: "MarkdownをHTMLに変換するツールをVanilla JSで作った"
emoji: "✍️"
type: tech
topics: ["javascript", "markdown", "html", "webdev"]
published: false
---

## これは何？

[DevNest.io](https://devnestio.pages.dev/) のツールとして作ったMarkdown → HTMLコンバーターです。

**URL**: https://markdown-to-html.pages.dev/

Markdownを入力するとHTMLのライブプレビューとHTMLソースをリアルタイムで表示します。外部ライブラリなし、ブラウザ内で完結。

## 実装

### HTMLエスケープ

```js
function escHtml(s) {
  return String(s)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;');
}
```

`&` を最初に変換するのが重要。後からやると `&lt;` が `&amp;lt;` になってしまいます。

### インライン要素のパース

```js
function parseInline(text) {
  return text
    // コードスパン（最初に処理）
    .replace(/`([^`]+)`/g, (_, c) => `<code>${escHtml(c)}</code>`)
    // 画像（リンクより先）
    .replace(/!\[([^\]]*)\]\(([^)]+)\)/g, (_, alt, src) => `<img src="${escHtml(src)}" alt="${escHtml(alt)}" />`)
    // リンク
    .replace(/\[([^\]]+)\]\(([^)]+)\)/g, (_, text, href) => `<a href="${escHtml(href)}">${text}</a>`)
    // ボールド+イタリック
    .replace(/\*\*\*(.+?)\*\*\*/g, '<strong><em>$1</em></strong>')
    // ボールド
    .replace(/\*\*(.+?)\*\*/g, '<strong>$1</strong>')
    .replace(/__(.+?)__/g, '<strong>$1</strong>')
    // イタリック
    .replace(/\*(.+?)\*/g, '<em>$1</em>')
    .replace(/_(.+?)_/g, '<em>$1</em>')
    // 取り消し線
    .replace(/~~(.+?)~~/g, '<del>$1</del>')
    // URL自動リンク
    .replace(/(https?:\/\/[^\s<>"]+)/g, '<a href="$1">$1</a>');
}
```

画像を `![]()` を先に処理しないとリンク `[]()` のパターンにマッチしてしまいます。

### ブロック要素のパース（行単位ステートマシン）

```js
function markdownToHtml(md) {
  const lines = md.split('\n');
  let html = '', i = 0;

  while (i < lines.length) {
    const line = lines[i];

    // フェンスコードブロック
    if (/^```/.test(line)) {
      const lang = line.slice(3).trim();
      i++;
      const codeLines = [];
      while (i < lines.length && !/^```/.test(lines[i])) { codeLines.push(lines[i]); i++; }
      i++;
      html += `<pre><code${lang ? ` class="language-${escHtml(lang)}"` : ''}>${escHtml(codeLines.join('\n'))}</code></pre>`;
      continue;
    }

    // 水平線
    if (/^(---+|\*\*\*+|___+)\s*$/.test(line)) { html += '<hr>'; i++; continue; }

    // 見出し
    const hMatch = line.match(/^(#{1,6})\s+(.+)/);
    if (hMatch) { html += `<h${hMatch[1].length}>${parseInline(hMatch[2])}</h${hMatch[1].length}>`; i++; continue; }

    // 引用
    if (/^>\s?/.test(line)) {
      const bqLines = [];
      while (i < lines.length && /^>\s?/.test(lines[i])) { bqLines.push(lines[i].replace(/^>\s?/, '')); i++; }
      html += `<blockquote>${markdownToHtml(bqLines.join('\n'))}</blockquote>`;
      continue;
    }

    // 箇条書き / 番号付きリスト / テーブル / 段落...
  }
  return html;
}
```

各行を順番に検査し、マッチしたブロック要素をまとめて消費して進める「ステートマシン」アプローチ。

### テーブルパース

```js
function parseTable(lines) {
  if (lines.length < 2) return null;
  const sep = lines[1];
  if (!/^\|?[\s\-|:]+\|?$/.test(sep)) return null;

  const parseRow = line => line.replace(/^\||\|$/g, '').split('|').map(c => c.trim());
  const aligns = parseRow(sep).map(c => {
    if (/^:-+:$/.test(c)) return 'center';
    if (/^-+:$/.test(c)) return 'right';
    return 'left';
  });
  // ...
}
```

セパレーター行（`| --- | --: |`）から各カラムの配置を読み取ります。`:-:` でcenter、`-:` でright。

## 機能

- **ライブプレビュー** — 入力のたびにHTMLをリアルタイムレンダリング
- **HTMLソース表示** — 生成されたHTMLをそのまま確認・コピー
- **対応要素** — H1〜H6 / 太字 / イタリック / 打ち消し / コードスパン / コードブロック(言語指定) / リスト(順序/箇条書き) / 表 / 引用 / 水平線 / リンク / 画像 / URL自動リンク
- **テーブル配置** — left / center / right アライン
- **コピーボタン** — HTMLをクリップボードにコピー

## テスト

Node.js `assert` モジュールで **121テスト** を実装、全パス確認済み。

テストカテゴリ:
- `escHtml` — 4特殊文字、複数箇所、二重エスケープ
- `parseInline` — bold/italic/strikethrough/code/link/image/auto-link
- Headings — H1〜H6、インライン要素含む見出し
- Paragraphs — 段落生成、空行区切り
- Lists — `- * +` 箇条書き、番号付き、複数アイテム
- Code blocks — フェンス、言語クラス、HTMLエスケープ
- Blockquotes — 単行、複数行、インライン要素
- HR — `--- *** ___` 各パターン
- Tables — `<table>/<thead>/<tbody>/<th>/<td>` 生成、右/中央配置
- `parseTable` — セパレーター検証、null返却
- Integration — 複数ブロックの組み合わせ

## 使い方

https://markdown-to-html.pages.dev/ にアクセスしてMarkdownを入力してください。

---

DevNest.io では他にも開発者向けツールを公開しています。
