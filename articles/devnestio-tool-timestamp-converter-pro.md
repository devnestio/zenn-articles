---
title: "Unix Timestamp Converter Pro を作った — エポック秒↔人間が読める日時、40+タイムゾーン対応"
emoji: "⏱️"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "cloudflare"]
published: true
---

## どんなツール？

ログファイルを見ていると `1735689600` という数字が出てきて「これ何日だっけ？」となることが多い。また逆に「2025年1月1日のエポック秒は？」を調べたいこともある。

**Unix Timestamp Converter Pro** は、そういった変換をブラウザだけで即座にできるツール。

👉 https://timestamp-converter-aaz.pages.dev

## 主な機能

### エポック→日時変換
- **秒・ミリ秒を自動判定**（境界値: 10,000,000,000）
- **40以上のタイムゾーン**に対応
- 出力形式: ISO 8601、RFC 2822、UTC文字列、日付のみ、時刻のみ、相対時間（"3 days ago"）

### ライブクロック
現在のUNIXタイムスタンプをリアルタイム表示（1秒更新）。コピーボタンつき。

### バッチ変換
複数のタイムスタンプを一括変換。秒/ミリ秒混在でも自動判定してくれる。

### 期間計算
2つのタイムスタンプの差を、ミリ秒・秒・分・時・日・週単位で表示。

### 日時→エポック変換
日時ピッカーで選んだ日時をエポック秒に変換。タイムゾーンも指定できる。

### 参照タイムスタンプ
クリックで即変換できる有名な日時：
- Unix Epoch (0)
- Y2K (946684800)
- Y2K38問題 (2147483647)
- 2025年1月1日 など

## 実装のポイント

```javascript
function detectUnit(val) {
  const n = Number(val);
  if (isNaN(n)) return null;
  if (n > 9999999999) return 'ms';  // 10桁超はミリ秒
  return 's';
}
```

秒とミリ秒の境界値は `9,999,999,999`（2286年頃）。現実的な範囲では安全に判定できる。

## テスト

Node.js の `assert` モジュールで90件のテストを実装。

```bash
node tests/test.js
# Total: 90 | Passed: 90 | Failed: 0
```

---

[devnestio](https://devnestio.pages.dev) - 無料の開発者ツール集
