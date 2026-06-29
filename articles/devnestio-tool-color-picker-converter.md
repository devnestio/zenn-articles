---
title: "Color Picker & ConverterをVanilla JSで作った — HEX/RGB/HSL/HSV相互変換とカラーハーモニー"
emoji: "🎨"
type: tech
topics: ["javascript", "color", "webdev", "tools"]
published: false
---

## これは何？

[DevNest.io](https://devnestio.pages.dev/) のツールとして作ったColor Picker & Converterです。

**URL**: https://color-picker-converter.pages.dev/

HEX・RGB・HSL・HSVを相互変換し、補色・三色配色・類似色・スプリット配色のカラーハーモニーをリアルタイムで表示します。外部ライブラリなし、ブラウザ完結。

## 実装

### HEX ↔ RGB

```js
function hexToRgb(hex) {
  const h = hex.replace('#', '');
  if (h.length !== 6) return null;
  const r = parseInt(h.slice(0, 2), 16);
  const g = parseInt(h.slice(2, 4), 16);
  const b = parseInt(h.slice(4, 6), 16);
  if (isNaN(r) || isNaN(g) || isNaN(b)) return null;
  return { r, g, b };
}

function rgbToHex(r, g, b) {
  return '#' + [r, g, b].map(v => Math.round(v).toString(16).padStart(2, '0')).join('');
}
```

`padStart(2,'0')` で1桁の場合のゼロ埋めを保証。`Math.round` で浮動小数点を整数に丸めてから変換します。

### RGB → HSL

```js
function rgbToHsl(r, g, b) {
  r /= 255; g /= 255; b /= 255;
  const max = Math.max(r, g, b), min = Math.min(r, g, b);
  let h = 0, s = 0;
  const l = (max + min) / 2;
  if (max !== min) {
    const d = max - min;
    s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
    switch (max) {
      case r: h = (g - b) / d + (g < b ? 6 : 0); break;
      case g: h = (b - r) / d + 2; break;
      case b: h = (r - g) / d + 4; break;
    }
    h /= 6;
  }
  return { h: Math.round(h * 360), s: Math.round(s * 100), l: Math.round(l * 100) };
}
```

明度 `l = (max + min) / 2`、彩度は明度が50%を超えるか否かで分岐します。色相は最大成分チャンネルで場合分け。

### HSL → RGB（CSS仕様準拠）

```js
function hslToRgb(h, s, l) {
  s /= 100; l /= 100;
  const k = n => (n + h / 30) % 12;
  const a = s * Math.min(l, 1 - l);
  const f = n => l - a * Math.max(-1, Math.min(k(n) - 3, Math.min(9 - k(n), 1)));
  return { r: Math.round(f(0) * 255), g: Math.round(f(8) * 255), b: Math.round(f(4) * 255) };
}
```

CSS Color Level 4 仕様の参照実装です。三角関数なしで正確にHSL→RGBを計算できます。

### RGB → HSV / HSV → RGB

```js
function rgbToHsv(r, g, b) {
  r /= 255; g /= 255; b /= 255;
  const max = Math.max(r, g, b), min = Math.min(r, g, b);
  const v = max;
  const s = max === 0 ? 0 : (max - min) / max;
  // ...色相計算はrgbToHslと同様
  return { h, s: Math.round(s * 100), v: Math.round(v * 100) };
}

function hsvToRgb(h, s, v) {
  s /= 100; v /= 100;
  const i = Math.floor(h / 60) % 6;
  const f = h / 60 - Math.floor(h / 60);
  const p = v * (1 - s), q = v * (1 - f * s), t = v * (1 - (1 - f) * s);
  const map = [[v,t,p],[q,v,p],[p,v,t],[p,q,v],[t,p,v],[v,p,q]][i];
  return { r: Math.round(map[0]*255), g: Math.round(map[1]*255), b: Math.round(map[2]*255) };
}
```

HSV は画像編集ソフトで一般的な色空間。Value（明度）が0なら完全な黒、Saturation（彩度）が0なら無彩色になります。

### カラーハーモニー

```js
function shiftHue(hex, delta) {
  const rgb = hexToRgb(hex);
  const { h, s, l } = rgbToHsl(rgb.r, rgb.g, rgb.b);
  const newH = ((h + delta) % 360 + 360) % 360;
  const nr = hslToRgb(newH, s, l);
  return rgbToHex(nr.r, nr.g, nr.b);
}

function harmonies(hex) {
  return {
    complementary: [hex, shiftHue(hex, 180)],
    triadic:       [hex, shiftHue(hex, 120), shiftHue(hex, 240)],
    analogous:     [shiftHue(hex, -30), hex, shiftHue(hex, 30)],
    split:         [hex, shiftHue(hex, 150), shiftHue(hex, 210)],
  };
}
```

HSLの色相環で角度をシフトするだけでカラーハーモニーが実現できます。`((h + delta) % 360 + 360) % 360` は負のデルタで負値になるのを防ぐイディオムです。

## 機能

- **HEX入力** — `#rrggbb` 形式で直接編集可能
- **ネイティブカラーピッカー** — `<input type="color">` で視覚的に選択
- **HSLスライダー** — 色相・彩度・明度を個別に調整
- **4形式表示** — HEX / RGB / HSL / HSV をコピーボタン付きで表示
- **カラーハーモニー** — 補色・三色・類似色・スプリット配色をスウォッチで表示
- **ハーモニーをクリックで選択** — スウォッチをクリックするとその色に切り替え

## テスト

Node.js `assert` モジュールで **110テスト** を実装、全パス確認済み。

テストカテゴリ:
- `hexToRgb` — 黒/白/RGB単色、大文字、ハッシュなし、不正入力(null返却)
- `rgbToHex` — 黒/白/RGB単色、浮動小数点丸め、7文字確認、round-trip
- `rgbToHsl` — 黒/白/RGB/グレー、各成分範囲確認
- `hslToRgb` — 黒/白/RGB、round-trip精度（±2以内）
- `rgbToHsv` — 黒/白/RGB、各成分範囲確認
- `hsvToRgb` — 黒/白/RGB、round-trip精度、各成分範囲
- `clamp` — 範囲内/範囲外/境界値/浮動小数点
- `isValidHex` — 有効パターン、無効パターン各種
- `shiftHue` — 0シフト、180/360シフト、二重シフト、負のデルタ
- `harmonies` — 4タイプ確認、各色数、先頭色一致、全色validHex
- `formatRgb/Hsl/Hsv` — 書式・% 有無確認
- Integration — hex→hsl→hex round-trip、hex→hsv→hex round-trip、補色の補色≈原色

## 使い方

https://color-picker-converter.pages.dev/ にアクセスして色を選んでください。

---

DevNest.io では他にも開発者向けツールを公開しています。
