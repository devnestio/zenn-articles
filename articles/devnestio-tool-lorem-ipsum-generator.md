---
title: "Lorem Ipsum ジェネレーターをVanilla JSで作った"
emoji: "📄"
type: tech
topics: ["javascript", "webdev", "tools", "random"]
published: false
---

## これは何？

[DevNest.io](https://devnestio.pages.dev/) のツールとして作ったLorem Ipsum（ローレム・イプサム）ジェネレーターです。

**URL**: https://lorem-ipsum-generator.pages.dev/

デザインやレイアウトのプレースホルダーテキストをブラウザ内で即座に生成します。単語・文・段落の3単位、プレーンテキスト・HTML・Markdownの3形式に対応。

## 実装

### シード付き疑似乱数生成器

```js
function seededRandom(seed) {
  let s = seed ^ 0xdeadbeef;
  return function() {
    s = Math.imul(s ^ (s >>> 16), 0x45d9f3b);
    s = Math.imul(s ^ (s >>> 16), 0x45d9f3b);
    s ^= s >>> 16;
    return (s >>> 0) / 0xffffffff;
  };
}
```

`Math.random()` は毎回異なる結果を返しますが、シード付き乱数は同じシードなら常に同じ列を生成します。Mulberry32 アルゴリズムを採用。`Math.imul` は32ビット整数乗算を高速に行うネイティブAPI。

### 文生成

```js
function generateSentence(rand, minWords = 5, maxWords = 18) {
  const len = minWords + Math.floor(rand() * (maxWords - minWords + 1));
  const words = generateWords(len, rand);
  words[0] = capitalize(words[0]);
  return words.join(' ') + '.';
}
```

5〜18語の範囲でランダムに文を生成し、先頭を大文字化してピリオドで終わらせます。

### 段落生成

```js
function generateParagraph(rand, minSents = 3, maxSents = 7) {
  const count = minSents + Math.floor(rand() * (maxSents - minSents + 1));
  const sentences = [];
  for (let i = 0; i < count; i++) sentences.push(generateSentence(rand));
  return sentences.join(' ');
}
```

1段落3〜7文。文末のピリオドのあとにスペースを挟んで結合します。

### 3単位 × 3形式の生成/フォーマット

```js
function generate(unit, count, startLorem, rand) {
  if (unit === 'words') {
    const words = generateWords(count, rand);
    if (startLorem && count >= 5) {
      // 先頭をLorem ipsum...で置き換え
      const loremWords = LOREM_START.replace(/[.,]/g,'').toLowerCase().split(' ');
      for (let i = 0; i < Math.min(loremWords.length, count); i++) words[i] = loremWords[i];
    }
    return [words.join(' ')];
  }
  if (unit === 'sentences') { /* 文ごとに生成 */ }
  if (unit === 'paragraphs') { /* 段落ごとに生成 */ }
}

function formatOutput(items, unit, format) {
  if (format === 'html') {
    if (unit === 'paragraphs') return items.map(p => `<p>${p}</p>`).join('\n');
    return `<p>${items.join(' ')}</p>`;
  }
  if (format === 'markdown') {
    return unit === 'paragraphs' ? items.join('\n\n') : items.join(' ');
  }
  return unit === 'paragraphs' ? items.join('\n\n') : items.join(' ');
}
```

## 機能

- **3単位** — 単語数 / 文数 / 段落数（1〜500）
- **3形式** — プレーンテキスト / HTML `<p>` / Markdown
- **「Lorem ipsum…」で開始** — オン/オフ切り替え可能
- **Regenerate** — 同じ設定で別の乱数列を生成
- **コピーボタン** — 生成テキストをクリップボードへ
- **統計表示** — 単語数・文数・文字数・ブロック数

## テスト

Node.js `assert` モジュールで **120テスト** を実装、全パス確認済み。

テストカテゴリ:
- `seededRandom` — [0,1)範囲、決定論性、異なるシード
- `WORDS` — 配列確認、全小文字
- `capitalize` — 基本変換、空文字、保持
- `pickWord` — WORDS内の語のみ返す
- `generateWords` — 件数、決定論性、全WORDS内
- `generateSentence` — 末尾ピリオド、先頭大文字、語数範囲
- `generateParagraph` — 文数範囲、決定論性
- `generate` — 3単位、startLorem動作、件数
- `formatOutput` — HTML/Markdown/plain全組み合わせ
- `countWords` / `countSentences` — 基本・エッジケース
- Integration — 生成→フォーマット→検証

## 使い方

https://lorem-ipsum-generator.pages.dev/ にアクセスして、数量・単位・形式を選択してGenerateボタンを押してください。

---

DevNest.io では他にも開発者向けツールを公開しています。
