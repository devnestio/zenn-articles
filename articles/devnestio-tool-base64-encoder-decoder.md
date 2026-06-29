---
title: "Base64エンコーダー/デコーダーを作った — ブラウザ完結で安全に変換する"
emoji: "🔐"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "cloudflare"]
published: true
---

## どんなツール？

文字列やファイルをBase64に変換したり、Base64から元に戻したりできるシンプルなWebツールです。すべてブラウザ内で完結し、データは一切サーバーに送信されません。

👉 https://base64-encoder-decoder-3yw.pages.dev

devnestio（[devnestio.pages.dev](https://devnestio.pages.dev)）で公開している開発者向け無料ツール群の一つとして作りました。

## なぜ作ったか

Base64変換は地味だけど頻度が高い作業です。データURIの埋め込み、JWTのペイロード確認、APIのBasic認証ヘッダー生成、メール添付のデコードなど、場面は意外と多い。

ところが「base64 encode online」で検索して出てくるサイトの多くは、入力をサーバーに送信して処理します。社内の認証トークンや個人情報を含む文字列を、素性のわからないサイトに貼り付けるのは正直こわい。だったら**完全にクライアントサイドで動くもの**を自分で用意しよう、というのが動機でした。

## 主な機能

- テキスト ⇔ Base64 の双方向変換
- ファイルをドラッグ&ドロップしてBase64化（画像のデータURI生成に便利）
- URLセーフBase64（`+/` を `-_` に置換）のオン/オフ
- UTF-8マルチバイト文字に正しく対応
- ワンクリックコピー

## 技術的なポイント

実装はVanilla JS。ビルドツールもフレームワークも使っていません。

地味にハマるのがマルチバイト対応です。素朴に `btoa()` を使うと日本語や絵文字で例外が飛びます。`btoa` はLatin-1しか受け付けないからです。そこで `TextEncoder` でUTF-8のバイト列に変換してから処理する形にしました。

```js
function encodeBase64(str) {
  const bytes = new TextEncoder().encode(str);
  let bin = "";
  bytes.forEach((b) => (bin += String.fromCharCode(b)));
  return btoa(bin);
}

function decodeBase64(b64) {
  const bin = atob(b64);
  const bytes = Uint8Array.from(bin, (c) => c.charCodeAt(0));
  return new TextDecoder().decode(bytes);
}
```

ファイル変換は `FileReader.readAsDataURL()` の結果からヘッダー部分を取り除く形で実装。URLセーフ変換は出力後に文字を置換するだけのシンプルな処理です。

ホスティングはCloudflare Pages。静的ファイルだけなので無料枠で十分、表示も速い。

## AIエージェントの活用

devnestioのツールは、Claude Codeのセッションを使って building-in-public 的に量産しています。「こういうツールを作って」と投げると、HTML/CSS/JSの雛形からエッジケースの洗い出しまで一気に進む。今回もマルチバイトの落とし穴はAIが先回りして指摘してくれたので、`btoa` の罠を踏まずに済みました。

人間がやるのは仕様の判断とテスト、そして「ここは安全側に倒したい」といった方針決めです。

## テスト結果

- ASCII文字列の往復変換 ✅
- 日本語・絵文字を含む文字列 ✅（`🔐 test` も正しく往復）
- 空文字列・改行を含む入力 ✅
- 不正なBase64入力時のエラーハンドリング ✅
- 数百KBのPNG画像のデータURI生成 ✅

## 今後の展望

要望が多ければ、Base64URLのパディング有無の切り替えや、Hex/Binaryとの相互変換も足したいと考えています。まずはシンプルさを保ったまま、確実に動くことを優先します。

## おわりに

「サーバーに送らない」ことを軸に、小さくて信頼できるツールを増やしていきます。他のツールもまとめてここに置いてあります：

👉 https://devnestio.pages.dev

フィードバックやリクエストがあれば気軽にどうぞ。
