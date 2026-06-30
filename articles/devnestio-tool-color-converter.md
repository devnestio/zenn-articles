---
title: "カラーコンバーターを作った — HEX/RGB/HSL/HSV/CMYKをブラウザで即変換"
emoji: "🎨"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "cloudflare"]
published: false
---

## どんなツール？

カラーコードをHEX、RGB、HSL、HSV、CMYK、CIE Lab間で相互変換できるWebツールです。カラーピッカー、パレット履歴、カラーハーモニー表示も搭載。

👉 https://color-converter-51e.pages.dev

devnestio（[devnestio.pages.dev](https://devnestio.pages.dev)）で公開している開発者向け無料ツール群の一つです。

## なぜ作ったか

デザインとコードの間で色変換が必要な場面は多い。Figmaは HSL、CSS は HEX、印刷は CMYK、照明系は HSV と、文脈ごとに使う形式が違う。毎回検索するのが面倒なので、全部一画面で完結するツールを作りました。

## 主な機能

- HEX ↔ RGB ↔ HSL ↔ HSV ↔ CMYK ↔ CIE Lab の相互変換
- ブラウザ組み込みのカラーピッカーで色を選択
- パレット履歴（最大20色）をlocalStorageに保存
- 補色・三色・分割補色・類似・四角形などのカラーハーモニー表示
- ワンクリックでコピー

## 技術

Vanilla JS単一ファイル、Cloudflare Pages でホスト。外部ライブラリ不使用。
