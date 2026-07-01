---
title: "カラーコントラストチェッカーをVanilla JSで作った — WCAG 2.1 AA/AAA対応"
emoji: "🎨"
type: tech
topics: ["javascript", "webdev", "accessibility", "wcag"]
published: false
---

## これは何？

[DevNest.io](https://devnestio.pages.dev/) のツールとして作ったカラーコントラストチェッカーです。

**URL**: https://devnestio.pages.dev/color-contrast-checker/

2色を選ぶとコントラスト比を計算し、WCAG 2.1 のAA・AAA基準を満たしているかを即座に判定します。アクセシビリティ対応のWebデザインに必要な情報を、外部依存なしのブラウザのみで提供します。

## WCAG コントラスト基準

| レベル | テキスト種別 | 最小比 |
|--------|-------------|--------|
| AA     | 通常テキスト（18pt未満 / 14pt bold未満）| 4.5:1 |
| AA     | 大きいテキスト（18pt以上 / 14pt bold以上）| 3.0:1 |
| AA     | UIコンポーネント・グラフィック | 3.0:1 |
| AAA    | 通常テキスト | 7.0:1 |
| AAA    | 大きいテキスト | 4.5:1 |

コントラスト比は 1:1（同色）から 21:1（白黒）の範囲を取ります。

## 実装

### 相対輝度の計算（WCAG 定義）

```js
function relativeLuminance(rgb) {
  const [r, g, b] = rgb.map(c => {
    const s = c / 255;
    return s <= 0.04045 ? s / 12.92 : Math.pow((s + 0.055) / 1.055, 2.4);
  });
  return 0.2126 * r + 0.7152 * g + 0.0722 * b;
}
```

各チャンネルをリニア光に変換（sRGBガンマ除去）してから、ITU-R BT.709の係数で加重平均します。係数の合計 `0.2126 + 0.7152 + 0.0722 = 1.0` で、白が輝度1.0になります。

### コントラスト比

```js
function contrastRatio(hex1, hex2) {
  const rgb1 = hexToRgb(hex1), rgb2 = hexToRgb(hex2);
  if (!rgb1 || !rgb2) return null;
  const l1 = relativeLuminance(rgb1), l2 = relativeLuminance(rgb2);
  const lighter = Math.max(l1, l2), darker = Math.min(l1, l2);
  return (lighter + 0.05) / (darker + 0.05);
}
```

WCAG定義の式: `(L1 + 0.05) / (L2 + 0.05)` ただし L1 ≥ L2。`+ 0.05` は絶対黒（輝度0）での除算ゼロを防ぎます。

### HEX変換

```js
function hexToRgb(hex) {
  const clean = hex.replace(/^#/, '');
  if (clean.length === 3) {
    return [
      parseInt(clean[0]+clean[0], 16),
      parseInt(clean[1]+clean[1], 16),
      parseInt(clean[2]+clean[2], 16),
    ];
  }
  if (clean.length === 6) {
    return [
      parseInt(clean.slice(0,2), 16),
      parseInt(clean.slice(2,4), 16),
      parseInt(clean.slice(4,6), 16),
    ];
  }
  return null;
}
```

3文字形式（`#abc`）は各文字を2回繰り返して展開します（`#aabbcc`と同等）。

### バリデーション

```js
function isValidHex(hex) {
  return /^#([0-9a-fA-F]{3}|[0-9a-fA-F]{6})$/.test(hex);
}
```

## 機能

- **カラーピッカー** + **テキスト入力**（HEX値）の両方に対応
- **スワップボタン** で前景・背景を入れ替え
- **プレビュー** — 通常テキスト、大テキスト、UIコンポーネントの3種類を実際の色で表示
- **WCAG 5段階判定** — AA通常/AA大/AA UI/AAA通常/AAA大それぞれのPASS・FAIL
- **よくある組み合わせ** 8パターンのクイック選択

## テスト

Node.js `assert` モジュールで **121テスト** を実装、全パス確認済み。

主なテストカテゴリ:
- `hexToRgb` — 6文字・3文字・大文字・無効入力
- `isValidHex` — 有効・無効パターン
- `relativeLuminance` — 黒(0)・白(1)・純色R/G/B、単調増加
- `contrastRatio` — 黒白の21:1、同色の1:1、対称性、無効入力
- `rgbToHex` — ラウンドトリップ、クランプ、四捨五入
- WCAG基準値との照合

## 使い方

https://devnestio.pages.dev/color-contrast-checker/ にアクセスして、前景色と背景色を選ぶだけです。HEX値を直接入力することもできます。

---

DevNest.io では他にも開発者向けツールを公開しています。
