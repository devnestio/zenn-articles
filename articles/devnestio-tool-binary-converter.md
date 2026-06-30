---
title: "Binary/Hex/Octal Converterをゼロ依存で作った話（ビットビューア・IEEE 754・2の補数）"
emoji: "🔢"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "javascript"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に **Binary / Hex / Octal Converter** を追加しました。2進数・8進数・10進数・16進数をリアルタイムに相互変換できます。外部ライブラリゼロ、すべてブラウザ内で完結します。

👉 https://binary-converter-alg.pages.dev

## なぜ作ったか

組み込み開発やネットワーク設定でビット演算をするとき、電卓アプリを開いて基数変換するのが地味に面倒でした。特にIEEE 754のfloatがどのようにビット列として表現されるかを可視化できるツールが欲しかった。

## 主な機能

- **4基数リアルタイム変換** — Binary / Octal / Decimal / Hex を同時更新
- **インタラクティブビットビューア** — 8/16/32/64ビットモード切替、クリックでビットをトグル
- **IEEE 754 フロート解析** — 32bit float / 64bit double の符号・指数部・仮数部を色分けで可視化
- **2の補数計算機** — 8/16/32ビット符号付き整数の補数表現を表示
- **入力バリデーション** — 無効文字を即時エラー表示

## 技術的なポイント：BigIntでの64ビット処理

JavaScriptのNumber型は安全に扱えるのが53ビット整数まで（`Number.MAX_SAFE_INTEGER = 2^53 - 1`）。64ビットのビットビューアを実現するためにBigIntを使っています。

```javascript
// 64ビット処理はBigIntで
binStr = (BigInt(val || 0) & ((1n << BigInt(total)) - 1n))
  .toString(2).padStart(total, '0');
```

32ビット以下は`>>> 0`で符号なし32ビット整数に変換してからtoString(2)を使うことで、負数の2進数表現も正しく取得できます。

## IEEE 754の可視化

Float32を4バイトのArrayBufferに書き込み、Uint32Arrayとして読み返すことでビット列を取得しています。

```javascript
function float32ToBits(f) {
  const buf = new ArrayBuffer(4);
  new Float32Array(buf)[0] = f;
  return new Uint32Array(buf)[0];
}
```

符号ビット（1bit）・指数部（8bit）・仮数部（23bit）を赤・オレンジ・緑でそれぞれ色分けして表示します。

## テスト

Node.js標準の`assert`モジュールのみを使い80件以上のテストを書いています。

- 基数変換ロジック（bin→dec, dec→hex, oct→dec など全方向）
- ラウンドトリップテスト（0〜65535の範囲で相互変換が一致するか）
- 2の補数計算（-1の8ビット補数が0xFFになるか等）
- バリデーションパターン（`/^[01]+$/`など）

```
$ node tests/test.js
All tests passed! (80+ assertions)
```

## ハブへの追加

devnestioのハブ（https://devnestio.pages.dev）に「Math」カテゴリでカードを追加しました。

---

devnestioは開発者向け無料ツール群です。ログイン不要・トラッキングなし。
