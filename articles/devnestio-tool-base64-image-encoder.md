---
title: "Base64画像エンコーダーをVanilla JSで作った話（PNG/JPG/SVGをData URLに変換、ゼロ依存）"
emoji: "🖼️"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "javascript"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に **Base64 Image Encoder** を追加しました。PNG・JPG・GIF・WebP・SVG画像をBase64 data URLに変換できます。外部ライブラリゼロ、すべてブラウザ内で完結し、画像データはサーバーに送信されません。

👉 https://base64-image-encoder.pages.dev

## なぜ作ったか

CSSの`background-image`やHTMLの`<img src="">`にインライン画像を埋め込みたいとき、小さなアイコン画像をHTTP requestなしで使いたいとき、Base64エンコードが必要になります。コマンドラインの`base64`コマンドでもできますが、ブラウザでドラッグ&ドロップして即コピーできる方が早い。

## 主な機能

- **ドラッグ&ドロップ対応** — ファイルを画面にドロップするだけ
- **対応フォーマット** — PNG / JPG / GIF / WebP / SVG
- **出力形式3種** — data URL全体 / Base64部分のみ / CSSコード
- **画像プレビュー** — エンコード前後のプレビュー表示
- **ファイルサイズ表示** — 元サイズとBase64後のサイズを比較
- **ワンクリックコピー**

## 技術的なポイント：FileReader APIでBase64変換

ブラウザのFileReader APIを使ってBase64エンコードを実装しています。

```js
function encodeImageToBase64(file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = (e) => resolve(e.target.result); // data:image/png;base64,AAA...
    reader.onerror = reject;
    reader.readAsDataURL(file);
  });
}

async function handleFile(file) {
  if (!file.type.startsWith('image/')) {
    throw new Error('Image files only');
  }
  const dataUrl = await encodeImageToBase64(file);
  const base64Only = dataUrl.split(',')[1];
  const cssCode = `background-image: url("${dataUrl}");`;
  
  displayResults({ dataUrl, base64Only, cssCode, file });
}
```

`FileReader.readAsDataURL()`はブラウザネイティブのAPIなので、外部ライブラリなしでBase64変換が完結します。SVGの場合は`image/svg+xml`のMIMEタイプが正しく設定されます。

## Base64はファイルサイズが増える

Base64エンコードすると元ファイルより約33%サイズが増えます。小さなアイコン（数KB）ならインライン埋め込みがHTTPリクエスト削減になりますが、大きな画像はファイル参照の方が効率的です。ツールではエンコード後のサイズを表示してこの判断をサポートしています。

## 実装はAIエージェントに任せた

Claude Codeに「FileReader APIとドラッグ&ドロップをシンプルに実装すること」を指示しました。コードはHTML単一ファイルに収まっており、ビルドステップなしでCloudflare Pagesにそのままデプロイできます。

## 使ってみてください

👉 **[devnestio - Developer Tools Hub](https://devnestio.pages.dev)**

CSSにアイコンをインライン埋め込みしたいときや、メールHTMLに画像を埋め込みたいときにどうぞ。
