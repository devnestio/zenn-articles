---
title: "文字数カウンター & テキスト分析ツールをVanilla JSで作った"
emoji: "📝"
type: tech
topics: ["javascript", "webdev", "tools", "textanalysis"]
published: false
---

## これは何？

[DevNest.io](https://devnestio.pages.dev/) のツールとして作った文字数カウンター & テキスト分析ツールです。

**URL**: https://char-counter-analyzer.pages.dev/

テキストを貼り付けると、文字数・単語数・文数・段落数・読書時間・発話時間・キーワード密度・文字頻度などをリアルタイムで分析します。

## 実装

### 基本カウント関数

```js
function countChars(text) { return text.length; }
function countCharsNoSpaces(text) { return text.replace(/\s/g, '').length; }

function countWords(text) {
  const trimmed = text.trim();
  if (!trimmed) return 0;
  return trimmed.split(/\s+/).length;
}

function countSentences(text) {
  const trimmed = text.trim();
  if (!trimmed) return 0;
  const matches = trimmed.match(/[^.!?]*[.!?]+/g);
  return matches ? matches.length : (trimmed.length > 0 ? 1 : 0);
}

function countParagraphs(text) {
  const trimmed = text.trim();
  if (!trimmed) return 0;
  return trimmed.split(/\n\s*\n+/).filter(p => p.trim().length > 0).length;
}
```

文の検出は `.!?` の連続を基準にします。句読点がなければ全体を1文として扱います。段落は空行（`\n\n`）で区切ります。

### 読書時間・発話時間

```js
function readingTimeMinutes(wordCount, wpm = 238) {
  return wordCount / wpm;
}

function speakingTimeMinutes(wordCount, wpm = 130) {
  return wordCount / wpm;
}

function formatTime(minutes) {
  if (minutes < 1) return `< 1 min`;
  const mins = Math.round(minutes);
  if (mins < 60) return `${mins} min`;
  const h = Math.floor(mins / 60), m = mins % 60;
  return m > 0 ? `${h}h ${m}m` : `${h}h`;
}
```

読書速度 238 wpm はバナナリーディング研究の中央値。発話速度 130 wpm はプレゼン・講義の平均値。

### キーワード密度

```js
function keywordDensity(text, topN = 15, minLen = 3) {
  const STOP = new Set(['the','and','for','that','this','with','are','from','was','but','not','have','had','has','its','you','all','can','our','will','been','they','their','than','then','when','what','your','more','into']);
  const words = text.toLowerCase().match(/[a-z]+/g) || [];
  const freq = {};
  for (const w of words) {
    if (w.length >= minLen && !STOP.has(w)) freq[w] = (freq[w] || 0) + 1;
  }
  const total = Object.values(freq).reduce((a,b) => a+b, 0) || 1;
  return Object.entries(freq)
    .sort((a,b) => b[1]-a[1])
    .slice(0, topN)
    .map(([word, count]) => ({ word, count, pct: (count / total * 100) }));
}
```

ストップワードリストで冠詞・接続詞・代名詞を除去し、3文字以上の単語のみ集計します。

### 文字頻度

```js
function charFrequency(text) {
  const freq = {};
  for (const ch of text.toLowerCase()) {
    if (/[a-z]/.test(ch)) freq[ch] = (freq[ch] || 0) + 1;
  }
  return Object.entries(freq).sort((a,b) => b[1]-a[1]).slice(0, 26);
}
```

英字 a-z のみカウント、大文字小文字を統一して集計します。

### その他の分析

```js
function countUniqueWords(text) {
  const words = text.trim().toLowerCase().match(/[a-z']+/g) || [];
  return new Set(words).size;
}

function avgWordLength(text) {
  const words = text.trim().match(/[a-zA-Z']+/g) || [];
  if (!words.length) return 0;
  return words.reduce((s, w) => s + w.length, 0) / words.length;
}

function longestWord(text) {
  const words = text.trim().match(/[a-zA-Z']+/g) || [];
  if (!words.length) return '';
  return words.reduce((a, b) => a.length >= b.length ? a : b, '');
}
```

## 機能

- **8つのカウント** — 文字数・スペース除く文字数・単語数・文数・段落数・行数・ユニーク単語数・平均単語長
- **読書時間 / 発話時間** — フォーマット済み表示（`< 1 min` / `5 min` / `1h 30m`）
- **最長単語** — テキスト中の最も長い英単語
- **キーワード密度** — ストップワード除外、上位15語、頻度バー付きテーブル
- **文字頻度** — アルファベット頻度グリッド
- **アクションボタン** — Clear / Copy / Load Sample

## テスト

Node.js `assert` モジュールで **126テスト** を実装、全パス確認済み。

テストカテゴリ:
- `countChars` — 空文字、スペース、改行
- `countCharsNoSpaces` — タブ・改行の除去
- `countWords` — 空白の扱い、複数スペース
- `countSentences` — 句読点パターン
- `countParagraphs` — 空行の扱い
- `countLines` — 改行カウント
- `countUniqueWords` — 大文字小文字統一、数字除外
- `readingTimeMinutes` / `speakingTimeMinutes` — 線形性、デフォルトwpm
- `formatTime` — 全区間（< 1 min / N min / Xh / Xh Ym）
- `keywordDensity` — ストップワード、topN、minLen、パーセンテージ
- `charFrequency` — ソート順、大文字小文字、最大26件
- `avgWordLength` — 数字除外、単文字
- `longestWord` — タイ、大文字小文字保持
- Integration — 実サンプル文字列での結合テスト

## 使い方

https://char-counter-analyzer.pages.dev/ にアクセスして、テキストを貼り付けるだけです。

---

DevNest.io では他にも開発者向けツールを公開しています。
