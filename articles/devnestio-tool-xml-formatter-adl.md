---
title: "DOMParserだけでXMLフォーマッター・バリデーターを作った"
emoji: "📄"
type: "tech"
topics: ["javascript","xml","webdev","browser","domparser"]
published: false
---

## はじめに

XMLのフォーマットにはライブラリが必要と思いがちですが、ブラウザ内蔵の `DOMParser` を使えば依存なしで実現できます。

**ツール:** https://devnestio.pages.dev/xml-formatter/

## DOMParserでパース

```js
function parseXml(text) {
  const parser = new DOMParser();
  const doc = parser.parseFromString(text, 'application/xml');
  const err = doc.querySelector('parsererror');
  if (err) throw new Error(err.textContent.trim().slice(0, 200));
  return doc;
}
```

`parsererror` 要素が存在すればパースエラーです。

## 再帰フォーマット + シンタックスハイライト

```js
function formatNode(node, indent, indentChar) {
  if (node.nodeType === Node.COMMENT_NODE)
    return indent + `<span class="x-comment">&lt;!--${escHtml(node.nodeValue)}--&gt;</span>\n`;
  if (node.nodeType !== Node.ELEMENT_NODE) return '';

  const tag = node.tagName;
  let attrs = '';
  for (const attr of node.attributes)
    attrs += ` <span class="x-attr">${escHtml(attr.name)}</span>=<span class="x-val">"${escHtml(attr.value)}"</span>`;

  const children = Array.from(node.childNodes);
  if (children.length === 0)
    return indent + `<span class="x-tag">&lt;${escHtml(tag)}</span>${attrs}<span class="x-tag"> /&gt;</span>\n`;

  // テキストのみの子要素はインライン展開
  if (children.length === 1 && children[0].nodeType === Node.TEXT_NODE) {
    return indent + `<span class="x-tag">&lt;${escHtml(tag)}</span>${attrs}<span class="x-tag">&gt;</span>`
      + `<span class="x-text">${escHtml(children[0].nodeValue)}</span>`
      + `<span class="x-tag">&lt;/${escHtml(tag)}&gt;</span>\n`;
  }

  let out = indent + `<span class="x-tag">&lt;${escHtml(tag)}</span>${attrs}<span class="x-tag">&gt;</span>\n`;
  for (const child of children) out += formatNode(child, indent + indentChar, indentChar);
  return out + indent + `<span class="x-tag">&lt;/${escHtml(tag)}&gt;</span>\n`;
}
```

## シンタックスカラーの種類

| クラス | 対象 | 色 |
|---|---|---|
| `x-tag` | タグ名・山括弧 | 水色 |
| `x-attr` | 属性名 | オレンジ |
| `x-val` | 属性値 | 緑 |
| `x-comment` | コメント | グレー |
| `x-cdata` | CDATA | 黄 |
| `x-pi` | 処理命令 | 紫 |

## ミニファイ

```js
function minify() {
  const doc = parseXml(text);
  const s = new XMLSerializer();
  rawOutput = s.serializeToString(doc).replace(/\s{2,}/g, ' ').trim();
}
```

コピー・ダウンロードはプレーンテキスト (`dataset.raw`) と HTMLを分離して管理しています。
