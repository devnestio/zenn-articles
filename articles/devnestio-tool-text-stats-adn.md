---
title: "Fleschリーダビリティスコアをブラウザで計算するテキスト統計ツール"
emoji: "📊"
type: "tech"
topics: ["javascript","nlp","webdev","browser","readability"]
published: false
---

## はじめに

Flesch Reading Ease スコアをブラウザだけで計算するテキスト統計ツールを作りました。

**ツール:** https://devnestio.pages.dev/text-stats/

## Flesch Reading Ease 公式

```
スコア = 206.835 - 1.015 × (単語数 / 文数) - 84.6 × (音節数 / 単語数)
```

| スコア | 難易度 |
|---|---|
| 90以上 | Very Easy |
| 70〜89 | Fairly Easy |
| 60〜69 | Standard |
| 50〜59 | Fairly Difficult |
| 30〜49 | Difficult |
| 30未満 | Very Difficult |

## 音節数カウント（近似）

```js
function countSyllables(word) {
  word = word.toLowerCase().replace(/[^a-z]/g, '');
  if (!word) return 0;
  if (word.length <= 3) return 1;
  // サイレントeと複数形の除去
  word = word.replace(/(?:[^laeiouy]es|ed|[^laeiouy]e)$/, '');
  word = word.replace(/^y/, '');
  const m = word.match(/[aeiouy]{1,2}/g);
  return m ? m.length : 1;
}
```

## ストップワードを除いたキーワード抽出

```js
const STOP_WORDS = new Set(['the','a','an','and','or','but','in','on','at','to','for','of','with',...]);

const freq = {};
for (const w of wordList) {
  const lw = w.toLowerCase();
  if (!STOP_WORDS.has(lw) && lw.length > 2) freq[lw] = (freq[lw] || 0) + 1;
}
const topWords = Object.entries(freq).sort((a,b) => b[1]-a[1]).slice(0, 20);
```

## 計算される統計値

- 単語数、文字数、文数、段落数、行数
- ユニーク単語数、平均単語長、平均文長
- 読了時間（238 wpm 基準）
- 文字の内訳: 英字、数字、句読点、スペース
- 上位20キーワード（頻度バッジ付き）
