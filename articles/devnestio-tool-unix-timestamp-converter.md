---
title: "Unixタイムスタンプ変換ツールをVanilla JSで作った"
emoji: "⏱️"
type: tech
topics: ["javascript", "webdev", "tools", "datetime"]
published: false
---

## これは何？

[DevNest.io](https://devnestio.pages.dev/) のツールとして作ったUnixタイムスタンプ変換ツールです。

**URL**: https://unix-timestamp-converter.pages.dev/

Unixタイムスタンプ（エポック秒）と人間が読める日時を相互変換します。秒・ミリ秒の自動判定、ISO 8601・UTC・相対時間・日付パーツ表示に対応。

## 実装

### 秒/ミリ秒の自動判定

```js
function detectUnit(num) {
  return Math.abs(num) > 1e10 ? 'ms' : 's';
}
```

`10,000,000,000 = 1e10` は 2001年のタイムスタンプ（秒）。これより大きければミリ秒と判断します。現在のエポック秒は約 `1.7e9`、ミリ秒は `1.7e12` なので、`1e10` で確実に分離できます。

### タイムスタンプ → Date

```js
function tsToDate(input, unit) {
  const n = typeof input === 'string' ? Number(input.trim()) : Number(input);
  if (isNaN(n)) return null;
  const u = unit === 'auto' || !unit ? detectUnit(n) : unit;
  return new Date(toMs(n, u));
}
```

### Date → タイムスタンプ

```js
function dateToTs(date, unit) {
  if (!(date instanceof Date) || isNaN(date.getTime())) return null;
  const ms = date.getTime();
  return unit === 'ms' ? ms : Math.floor(ms / 1000);
}
```

`Math.floor` で切り捨て（ミリ秒を丸めない）。

### 相対時間

```js
function formatRelative(date) {
  const diff = (Date.now() - date.getTime()) / 1000;
  const abs  = Math.abs(diff);
  const fut  = diff < 0;
  if (abs < 5)        return 'just now';
  if (abs < 60)       { const s = Math.round(abs);       return fut ? `in ${s}s`     : `${s}s ago`; }
  if (abs < 3600)     { const m = Math.round(abs/60);    return fut ? `in ${m}m`     : `${m}m ago`; }
  if (abs < 86400)    { const h = Math.round(abs/3600);  return fut ? `in ${h}h`     : `${h}h ago`; }
  if (abs < 2592000)  { const d = Math.round(abs/86400); return fut ? `in ${d} days` : `${d} days ago`; }
  if (abs < 31536000) { const mo= Math.round(abs/2592000); return fut ? `in ${mo}mo` : `${mo}mo ago`; }
  const y = Math.round(abs/31536000);
  return fut ? `in ${y}yr` : `${y}yr ago`;
}
```

過去・未来どちらも対応。5秒以内は `just now`、以降は秒→分→時→日→月→年で段階的に表示します。

### うるう年・月別日数

```js
function isLeapYear(year) {
  return year % 4 === 0 && (year % 100 !== 0 || year % 400 === 0);
}

function daysInMonth(year, month) {
  return new Date(year, month, 0).getDate();
}
```

`new Date(year, month, 0)` は「その月の0日目」= 前月末日を使うブラウザネイティブな手法。

### 年通算日・年通算週

```js
function dayOfYear(date) {
  const start = new Date(Date.UTC(date.getUTCFullYear(), 0, 0));
  return Math.floor((date.getTime() - start.getTime()) / 86400000);
}

function weekOfYear(date) {
  const start = new Date(Date.UTC(date.getUTCFullYear(), 0, 1));
  const diff = date.getTime() - start.getTime();
  return Math.ceil((diff / 86400000 + start.getUTCDay() + 1) / 7);
}
```

## 機能

- **ライブクロック** — 現在のUnix秒をリアルタイム表示、ワンクリックで変換フォームに転送
- **タイムスタンプ → 日時** — UTC / ISO 8601 / ローカル / 相対時間 / 曜日 / 月名 / 年通算日 / 年通算週 / ms値を表示
- **日付パーツグリッド** — 年・月・日・時・分・秒・ミリ秒をカード表示
- **日時 → タイムスタンプ** — `datetime-local` ピッカー、秒・ミリ秒の両方を表示
- **単位タブ** — Auto-detect / Seconds / Milliseconds で切り替え
- **クリックでコピー** — 各結果行をクリックでクリップボードにコピー

## テスト

Node.js `assert` モジュールで **122テスト** を実装、全パス確認済み。

テストカテゴリ:
- `detectUnit` — 秒/ミリ秒判定境界値
- `toMs` — 単位変換
- `tsToDate` — 文字列・数値・auto検出・既知タイムスタンプ
- `dateToTs` — roundtrip、floor除算、無効値
- `nowTimestamp` — 桁数チェック
- `isLeapYear` — 4/100/400の法則
- `daysInMonth` — 全12月、うるう年
- `dateParts` — 全フィールド、曜日
- `pad` — ゼロパディング
- `dayOfYear` — うるう年、年末
- `formatRelative` — 全タイムレンジ、未来・過去
- Integration — 既知タイムスタンプ `1700000000` のroundtrip

## 使い方

https://unix-timestamp-converter.pages.dev/ にアクセスして、タイムスタンプまたは日付を入力してください。

---

DevNest.io では他にも開発者向けツールを公開しています。
