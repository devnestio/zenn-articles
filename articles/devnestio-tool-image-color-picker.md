---
title: "画像からピクセルの色を取得するブラウザツールを作った（HEX/RGB/HSL出力）"
emoji: "🎨"
type: "tech"
topics: ["javascript", "canvas", "devtools", "design", "cloudflare"]
published: true
---

## 作ったもの

開発者向け無料ツール群「devnestio」に、**Image Color Picker** を追加しました。

👉 https://image-color-picker-ebm.pages.dev

画像をアップロードしてピクセルをクリックすると、そのピクセルの HEX・RGB・HSL 値を即座に取得できます。

---

## 主な機能

- **ドラッグ&ドロップ**でPNG・JPG・GIF・WebP・SVGに対応
- **ルーペ（8倍ズーム）** — カーソルに追従して精密な位置確認が可能
- **クリック** → HEX / RGB / HSL 値を即表示
- 個別の **R・G・B チャンネル値**も表示
- 各値に**ワンクリックコピー**ボタン
- **パレット履歴** — ピックした色をスウォッチとして保存、クリックで再選択
- 画像はすべてブラウザ内で処理（サーバー送信なし）

---

## 色変換の実装

```js
// RGB → HEX
function rgbToHex(r, g, b) {
  return '#' + [r, g, b].map(v => v.toString(16).padStart(2, '0')).join('');
}

// RGB → HSL
function rgbToHsl(r, g, b) {
  r /= 255; g /= 255; b /= 255;
  const max = Math.max(r, g, b), min = Math.min(r, g, b);
  let h, s, l = (max + min) / 2;
  if (max === min) { h = s = 0; }
  else {
    const d = max - min;
    s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
    switch(max) {
      case r: h = ((g - b) / d + (g < b ? 6 : 0)) / 6; break;
      case g: h = ((b - r) / d + 2) / 6; break;
      case b: h = ((r - g) / d + 4) / 6; break;
    }
  }
  return { h: Math.round(h * 360), s: Math.round(s * 100), l: Math.round(l * 100) };
}
```

Canvas API の `getImageData` で1ピクセルのRGBA値を取得し、上記の変換関数で HEX / HSL に変換しています。

---

## 使いどころ

- ロゴや画像からブランドカラーを抽出したい
- デザインカンプのスクリーンショットから正確な色コードを取得したい
- UIスクリーンショットの特定ピクセルが何色か確認したい

---

## テスト

Node.js の `assert` で **86件** のテストを実装し全通過確認済みです。色変換ロジック（rgbToHex / hexToRgb / rgbToHsl）をカバーしています。

---

## 使ってみてください

👉 **[Image Color Picker — devnestio](https://image-color-picker-ebm.pages.dev)**

全ツール一覧: https://devnestio.pages.dev
