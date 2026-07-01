---
title: "px/rem/em/vw など10種類のCSS単位を相互変換するツール"
emoji: "📐"
type: "tech"
topics: ["css","javascript","webdev","browser","units"]
published: false
---

## はじめに

CSSの単位変換は毎日行う作業です。px、rem、em、vw、vh、pt、pc、cm、mm、in の10種類を相互変換するブラウザツールを作りました。

**ツール:** https://devnestio.pages.dev/px-rem-converter/

## ユニット定義

```js
const UNITS = [
  { id: 'px',  label: 'px',  desc: 'Pixels (absolute)' },
  { id: 'rem', label: 'rem', desc: 'Root em (relative to base)' },
  { id: 'em',  label: 'em',  desc: 'Em (relative to parent)' },
  { id: 'vw',  label: 'vw',  desc: 'Viewport width %' },
  { id: 'vh',  label: 'vh',  desc: 'Viewport height %' },
  { id: 'pt',  label: 'pt',  desc: 'Points (1pt = 1.333px)' },
  { id: 'pc',  label: 'pc',  desc: 'Picas (1pc = 16px)' },
  { id: 'cm',  label: 'cm',  desc: 'Centimeters' },
  { id: 'mm',  label: 'mm',  desc: 'Millimeters' },
  { id: 'in',  label: 'in',  desc: 'Inches' },
];
```

## 変換の仕組み

すべてのユニットをいったんpxに変換し、そこから目的のユニットへ変換します。

```js
function toPx(value, unit) {
  const base = getBase(), parent = getParent(), vpW = getVpW(), vpH = getVpH();
  switch(unit) {
    case 'px':  return value;
    case 'rem': return value * base;
    case 'em':  return value * parent;
    case 'vw':  return value * vpW / 100;
    case 'vh':  return value * vpH / 100;
    case 'pt':  return value / 0.75;   // 1pt = 1.333px
    case 'pc':  return value * 16;     // 1pc = 16px
    case 'cm':  return value / 0.026458;
    case 'mm':  return value / 0.26458;
    case 'in':  return value / 0.010417;
  }
}
```

## 変換フォーミュラ一覧

| ユニット | フォーミュラ |
|---|---|
| rem | px ÷ base |
| em | px ÷ parent |
| vw | px ÷ viewport-width × 100 |
| vh | px ÷ viewport-height × 100 |
| pt | px × 0.75 |
| pc | px ÷ 16 |
| cm | px × 0.026458 |
| mm | px × 0.26458 |
| in | px × 0.010417 |

## 機能

- ベースフォントサイズ、親フォントサイズ、ビューポートサイズを設定可能
- どのユニットにフォーカスしてもそこから全ユニットへリアルタイム変換
- よく使うpx値の参照テーブル付き（1〜96px）
