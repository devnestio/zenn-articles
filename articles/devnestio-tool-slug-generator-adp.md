---
title: "8種類のケース形式に対応したURLスラッグジェネレーター"
emoji: "🔗"
type: "tech"
topics: ["javascript","webdev","browser","text","url"]
published: false
---

## はじめに

テキストをURLスラッグや識別子に変換するブラウザツールを作りました。8種類のケース形式に対応しています。

**ツール:** https://devnestio.pages.dev/slug-generator/

## 対応フォーマット

| ID | 形式 | 例 |
|---|---|---|
| kebab | kebab-case | `hello-world-2024` |
| snake | snake_case | `hello_world_2024` |
| camel | camelCase | `helloWorld2024` |
| pascal | PascalCase | `HelloWorld2024` |
| dot | dot.case | `hello.world.2024` |
| upper_snake | UPPER_SNAKE | `HELLO_WORLD_2024` |
| title | Title Case | `Hello World 2024` |
| path | path/case | `hello/world/2024` |

## 実装

```js
const FORMATS = [
  {
    id: 'kebab', label: 'kebab-case', sep: '-',
    fn: words => words.join('-').toLowerCase()
  },
  {
    id: 'camel', label: 'camelCase', sep: '',
    fn: words => words.map((w, i) =>
      i === 0 ? w.toLowerCase() : w.charAt(0).toUpperCase() + w.slice(1).toLowerCase()
    ).join('')
  },
  {
    id: 'pascal', label: 'PascalCase', sep: '',
    fn: words => words.map(w =>
      w.charAt(0).toUpperCase() + w.slice(1).toLowerCase()
    ).join('')
  },
  // ...
];
```

## アクセント文字の変換

```js
const CHAR_MAP = {
  'à':'a', 'á':'a', 'ç':'c', 'ß':'ss', 'æ':'ae', 'œ':'oe',
  '&':'and', '+':'plus', '@':'at', '#':'hash', '€':'euro',
};

function transliterate(text) {
  return text.split('').map(c => CHAR_MAP[c.toLowerCase()] ?? c).join('');
}
```

さらに `normalize('NFD')` で残ったダイアクリティクスを除去します。

## 機能

- **最大文字数:** 50/70/100文字または無制限（単語境界で切断）
- **オプション:** 小文字化、前後の区切り文字除去、連続スペースの統合
- **バッチ変換:** 1行1テキストで一括変換
- **アクセント変換テーブル:** 30文字分の変換マップを表示
- **150msデバウンス** でリアルタイム変換
