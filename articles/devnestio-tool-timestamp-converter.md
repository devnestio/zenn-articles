---
title: "タイムスタンプコンバーターをVanilla JSで作った話（30タイムゾーン・DST対応、124テスト）"
emoji: "⏱️"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "javascript"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に **Timestamp Converter** を追加しました。Unixタイムスタンプと人間が読める日時の相互変換、30以上のタイムゾーン対応、DST（夏時間）の自動考慮を備えたツールです。

👉 https://timestamp-converter-dev.pages.dev

## なぜ作ったか

ログに残ったUnixタイムスタンプを日時に変換したいとき、逆に「この日時のエポック秒は？」を調べたいとき、毎回ターミナルで `date -r 1719532800` と叩くのが面倒でした。タイムゾーンまで正確に変換してくれるブラウザツールがほしかった。

## 主な機能

- **Unixタイムスタンプ → 日時** — 秒・ミリ秒の自動判定
- **日時 → Unixタイムスタンプ** — 任意タイムゾーン指定
- **30以上のタイムゾーン対応** — UTC、各大陸の主要都市
- **DST（夏時間）の自動考慮** — `Intl.DateTimeFormat`を活用
- **現在時刻のワンクリック取得**
- **相対時間表示** — 「3時間前」「2日後」など

## 技術的なポイント：タイムゾーン変換

ライブラリなしでタイムゾーン変換をするには、`Intl.DateTimeFormat`が最も信頼できます。

```js
function convertToTimezone(epochMs, tz) {
  const formatter = new Intl.DateTimeFormat('en-US', {
    timeZone: tz,
    year: 'numeric', month: '2-digit', day: '2-digit',
    hour: '2-digit', minute: '2-digit', second: '2-digit',
    hour12: false,
  });
  const parts = Object.fromEntries(
    formatter.formatToParts(new Date(epochMs)).map(p => [p.type, p.value])
  );
  return `${parts.year}-${parts.month}-${parts.day} ${parts.hour}:${parts.minute}:${parts.second}`;
}
```

`Intl`のタイムゾーンデータはブラウザが持っているため、DSTの計算も自動で正しく処理されます。自前でUTCオフセットを足し引きする実装と比べて、夏時間の切り替わりタイミングまで正確です。

秒とミリ秒の自動判定は、値が`1e10`（2001年以降）より大きければミリ秒とみなすヒューリスティックを使っています。

```js
function detectUnit(n) {
  return n > 1e10 ? 'ms' : 's';
}
```

## テスト結果

**124/124テスト PASS**。DST境界値（3月の切り替わり前後1分）、うるう秒付近のエポック値、ミリ秒/秒の自動判定、タイムゾーン別の変換結果などを確認しています。

## 実装はAIエージェントに任せた

Claude Codeに発注書を渡しています。「DSTの自動考慮」「秒とミリ秒の自動判定」「相対時間表示」を要件に含めたことで、実用的なツールが一発で仕上がりました。

## 使ってみてください

👉 **[devnestio - Developer Tools Hub](https://devnestio.pages.dev)**

ログのエポック秒が何時何分か、すぐわかります。
