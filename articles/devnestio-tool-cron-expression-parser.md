---
title: "Cron式パーサーをVanilla JSで作った — 次回実行時間の計算まで"
emoji: "⏰"
type: tech
topics: ["javascript", "webdev", "tools", "cron"]
published: false
---

## これは何？

[DevNest.io](https://devnestio.pages.dev/) のツールとして作ったCron式パーサーです。

**URL**: https://cron-expression-parser.pages.dev/

Cron式を入力すると、各フィールドの意味を解説し、次回10件の実行時間をプレビューします。5フィールド（標準形式）と6フィールド（秒フィールド付き）の両方に対応しています。

## Cron式の基本

```
┌───────── 分 (0-59)
│ ┌─────── 時 (0-23)
│ │ ┌───── 日 (1-31)
│ │ │ ┌─── 月 (1-12 または jan-dec)
│ │ │ │ ┌─ 曜日 (0-7、0と7が日曜)
│ │ │ │ │
* * * * *
```

## 実装

### フィールド解析

```js
function parseField(token, def) {
  const t = applyAlias(token.trim(), def.aliases);
  const { min, max } = def;
  const values = new Set();
  const parts = t.split(',');

  for (const part of parts) {
    if (part === '*') {
      for (let i = min; i <= max; i++) values.add(i);
      continue;
    }
    // step: */5 or 0-30/5
    const stepMatch = part.match(/^(.+)\/(\d+)$/);
    if (stepMatch) { /* ... */ }
    // range: 1-5
    const dashMatch = part.match(/^(\d+)-(\d+)$/);
    if (dashMatch) { /* ... */ }
    // single value
    if (/^\d+$/.test(part)) { /* ... */ }
    return null; // invalid
  }
  return [...values].sort((a,b) => a-b);
}
```

各フィールドは `Set<number>` として「このフィールドに一致する値の集合」を返します。`*` はすべての値、`1-5` は範囲、`*/15` はステップ、カンマ区切りはリストとして処理します。

### エイリアス解決

```js
const monthAliases = { jan:1, feb:2, mar:3, /* ... */ dec:12 };
const dowAliases   = { sun:0, mon:1, tue:2, /* ... */ sat:6 };

function applyAlias(token, aliases) {
  return token.replace(/[a-zA-Z]+/g, m => {
    const v = aliases[m.toLowerCase()];
    return v !== undefined ? String(v) : m;
  });
}
```

`jan-mar` → `1-3`、`mon-fri` → `1-5` のように、数値変換してから範囲・ステップ解析に回します。

### 次回実行時間の計算

```js
function nextRuns(parsed, count = 10, from = new Date()) {
  // 1秒進めた cursor から始めてループ
  const cursor = new Date(from.getTime() + 1000);
  cursor.setMilliseconds(0);

  while (runs.length < count && iterations < 200000) {
    // 月が一致しない → 翌月1日0:00へジャンプ
    // 日/曜日が一致しない → 翌日0:00へジャンプ
    // 時が一致しない → 次の一致する時間へジャンプ
    // 分が一致しない → 次の一致する分へジャンプ
    // すべて一致 → runs に追加
  }
}
```

ブルートフォースではなく、一致しないフィールドが見つかった時点で大きくジャンプします。最大イテレーション数を 200,000 に制限して無限ループを防いでいます。

### Cron の曜日と日付の AND/OR ルール

Cron には「DOM（日）と DOW（曜日）の両方が `*` でない場合は OR 条件」という標準外ルールがあります。

```js
const domWild = domVals.includes(1) && domVals.includes(31) && domVals.length === 31;
const dowWild = fields[idx+4].raw === '*';
const dayMatch = dowWild ? domMatch : (domWild ? dowMatch : (domMatch || dowMatch));
```

- DOW が `*` → DOM だけで判定
- DOM が `*` → DOW だけで判定
- 両方指定 → DOM **または** DOW のどちらかが一致すれば実行（Vixie cron の動作）

## テスト

Node.js `assert` モジュールで **121テスト** を実装、全パス確認済み。

テストカテゴリ:
- `applyAlias` — エイリアス変換（月名・曜日名）
- `parseField` — `*`, 数値, 範囲, ステップ, カンマリスト, エイリアス, エラーケース
- `parseCron` — 5/6フィールド、エラー系、フィールド値の正確性
- `ordSuffix` — 序数サフィックス（1st, 2nd, 11th, 21st...）
- `relativeTime` — 秒/分/時/日/週単位の相対時間フォーマット
- `nextRuns` — 実行時間計算、昇順確認、インスタンス型確認

## 使い方

https://cron-expression-parser.pages.dev/ にアクセスして、Cron式を入力するか、下部のCommon Examplesをクリックして試してください。

---

DevNest.io では他にも開発者向けツールを公開しています。
