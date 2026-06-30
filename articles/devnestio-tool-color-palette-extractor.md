---
title: "Color Palette Extractor — 画像から主要カラーパレットを抽出するブラウザツール"
emoji: "🎨"
type: "idea"
topics: ["color", "design", "tools", "javascript", "canvas"]
published: false
---

## Color Palette Extractorとは

[Color Palette Extractor](https://color-palette-extractor-8es.pages.dev) は、画像をアップロードするだけで主要なカラーパレットを抽出できる無料ツールです。k-meansクラスタリングをブラウザ内で実行するため、画像データがサーバーに送信されることはありません。

## 主な機能

- **k-meansによるカラー抽出**: 2〜16色のパレットを設定可能
- **3フォーマット出力**: HEX・RGB・HSL（個別または全表示）
- **使用率表示**: 各色の画像内での占有率（%）をプログレスバーで可視化
- **パレットダウンロード**: HEX/RGB/HSL値入りのPNGとして保存
- **ドラッグ&ドロップ対応**: PNG・JPG・GIF・WebP・BMP・SVGをサポート
- **プライバシー保護**: 処理はすべてブラウザ内で完結

## 使い方

1. 画像をドロップまたはクリックしてアップロード
2. カラー数スライダー（2〜16）で抽出色数を調整
3. 各色カードをクリックしてHEX/RGB/HSLをコピー
4. 「Download PNG」でパレット画像を保存

## devnestioについて

[devnestio](https://devnestio.pages.dev) は、ログイン不要・無料で使えるデベロッパーツール集です。
