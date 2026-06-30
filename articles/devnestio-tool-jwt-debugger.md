---
title: "JWTデバッガー：ブラウザでJWTトークンをデコード・検証"
emoji: "🔑"
type: "idea"
topics: ["jwt", "security", "javascript", "webdev", "tools"]
published: true
---

## JWT Debugger — 無料オンラインツール

[JWT Debugger](https://jwt-debugger-abe.pages.dev) は、JSONウェブトークン（JWT）をブラウザ内で完全にデコード・検証できる無料ツールです。データがサーバーに送信されることはありません。

## 主な機能

- **カラーコード表示** — ヘッダー（ピンク）、ペイロード（紫）、署名（ティール）を色分け表示
- **HMAC署名検証** — Web Crypto APIを使ってHS256/HS384/HS512を検証
- **クレームインスペクター** — iss, sub, aud, exp, nbf, iat, jtiを人間が読める形式で表示
- **有効期限ステータス** — 残り時間または期限切れバッジを表示
- **アルゴリズム自動検出** — ヘッダーからalgを自動判定
- **3つのビュー** — ビジュアル、Raw JSON、クレームグリッド
- **Base64エンコードシークレット対応**

## 使い方

1. 入力ボックスにJWTトークンを貼り付け
2. ヘッダーとペイロードが即座にデコード表示
3. HMACシークレットを入力して署名を検証
4. タブを切り替えて個別のクレームを確認

## なぜクライアントサイドなのか

JWTトークンにはユーザーID、ロール、権限などの機密情報が含まれることが多いです。クライアントサイドのツールを使うことで、トークンがサーバーに触れることがありません。

## リンク

- ツール: https://jwt-debugger-abe.pages.dev
- DevNest.io コレクション: https://devnestio.pages.dev
