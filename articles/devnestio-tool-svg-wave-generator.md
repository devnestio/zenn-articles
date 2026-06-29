---
title: "SVGウェーブジェネレーターをVanilla JSで作った話（数式からSVGパスを生成する）"
emoji: "🌊"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "svg"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に **SVG Wave Generator** を追加しました。サイン波・ランダム波・ジグザグなど複数のウェーブタイプを、スライダーひとつで調整してSVGコードとして書き出せるツールです。

👉 https://svg-wave-generator.pages.dev

外部ライブラリゼロ・単一HTMLファイル・サーバー送信なしで完結しています。

## なぜ作ったか

ランディングページのセクション区切りに「波形のSVG」を使いたい場面がよくあります。CSSだけで実現する方法もありますが、複雑な波形はどうしてもSVGの`<path>`を手書きする羽目になります。計算が面倒で、試行錯誤のたびにブラウザで確認が必要でした。

**リアルタイムプレビューしながらパラメータを調整できる**ツールがあれば、そのコードをコピーして貼るだけで済みます。

## 主な機能

- **4種のウェーブタイプ** — サイン波・ランダム（バンプ）・ジグザグ・フラット
- **振幅・周波数・位相のリアルタイム調整** — スライダーで即座に反映
- **前景色・背景色の設定**
- **反転オプション**（波の上下を入れ替え）
- **SVGコードのワンクリックコピー**

## 技術的なポイント：SVGパスの数値計算

サイン波のパスを生成する核心部分は以下のような計算です。

```js
function buildSinePath(width, height, amplitude, frequency, phase) {
  const points = [];
  for (let x = 0; x <= width; x += 2) {
    const y = height / 2 + amplitude * Math.sin((x / width) * Math.PI * 2 * frequency + phase);
    points.push(`${x},${y.toFixed(2)}`);
  }
  return `M 0,${height} L ${points.join(' L ')} L ${width},${height} Z`;
}
```

`Math.sin()` で横方向の進みを角度に変換し、振幅を掛けて縦方向のオフセットを求めます。パスは左下 → 波形上端 → 右下 → 左下と閉じることで、塗りつぶし可能な`<path>`になります。

ランダム波は `Math.sin()` を複数重ね合わせる代わりに、乱数シードで制御点を生成してスプライン補間に近い効果を出しています。

## 実装はAIエージェントに任せた

devnestioのツールはClaude Codeエージェントに発注書を渡して実装してもらっています。今回は「パラメータ変更のたびにSVGをリアルタイム再描画すること」「コピーしたコードがそのままHTMLに貼れること」を要件に含めました。

## テスト結果

Node.js実行型テストスイートで **120件以上 全PASS**。パス文字列のフォーマット検証、各ウェーブタイプの端点座標確認、パラメータの境界値テストを網羅しています。

## 使ってみてください

👉 **[devnestio - Developer Tools Hub](https://devnestio.pages.dev)**

セクション区切りのSVG波形を素早く生成したい方に。コピーして`<svg>`タグをHTMLに貼るだけです。
