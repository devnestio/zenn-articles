---
title: "JWTデコーダーを作った — 秘密鍵不要でヘッダー・ペイロード・クレームを即確認"
emoji: "🔓"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "cloudflare"]
published: false
---

## どんなツール？

JWTトークンをデコードしてヘッダー・ペイロードをシンタックスハイライト表示し、標準クレームの意味・有効期限・署名アルゴリズムを一覧で確認できるWebツールです。

👉 https://jwt-decoder-38t.pages.dev

devnestio（[devnestio.pages.dev](https://devnestio.pages.dev)）で公開している開発者向け無料ツール群の一つです。

## なぜ作ったか

APIデバッグ時にJWTの中身を確認したい場面は多い。jwt.io は有名だが、秘密情報をサードパーティサイトに貼るのに抵抗がある場面もある。このツールはすべてブラウザ内で完結するため、トークンが外部に送信されません。

## 主な機能

- ヘッダー・ペイロードをJSONシンタックスハイライト表示
- 標準クレーム（iss/sub/aud/exp/nbf/iat/jti）の解説と値表示
- `exp`・`nbf` の有効期限チェック（期限切れは赤、有効は緑で表示）
- `exp` まで/からの相対時間表示（"in 2h 30m" など）
- アルゴリズム（HS256/RS256/ES256など）バッジ表示
- トークンの3パートを色分けして表示（ヘッダー/ペイロード/署名）
- ペースト時に自動デコード
- サンプルトークン4種（Basic/Expired/RS256/nbf付き）

## 技術

Base64URLデコードのみでJWT構造を解析。署名検証はブラウザ完結の性質上省略（構造確認・デバッグ用途に特化）。Cloudflare Pages でホスト、外部ライブラリ不使用。
