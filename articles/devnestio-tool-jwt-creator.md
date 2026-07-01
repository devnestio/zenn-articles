---
title: "JWTをブラウザで作って署名する — Web Crypto APIでHS256/384/512（77テスト）"
emoji: "🔐"
type: "tech"
topics: ["jwt", "security", "javascript", "webdev", "devtools"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に、**JWT Creator & Signer** を追加しました。

👉 https://devnestio.pages.dev/jwt-creator/

JWTの作成・署名・検証がすべてブラウザ内で完結します。外部サービスにシークレットを送信しません。

---

## 作った動機

JWT認証のデバッグをするとき、複数のタブとツールを行き来するのが面倒でした。jwt.ioは便利ですが、署名シークレットをサードパーティサーバーに送ることに抵抗がありました。

---

## 主な機能

- **HS256/HS384/HS512** の HMAC 署名（Web Crypto API使用）
- **ヘッダー・ペイロード編集** — JSON直接編集、クレームワンクリック追加（sub, name, exp, iss）
- **ランダムシークレット生成** — `crypto.getRandomValues()` で256ビット
- **JWT検証** — 貼り付けて署名＋有効期限チェック
- **色分け出力** — ヘッダー赤・ペイロード緑・署名青
- **完全ブラウザ完結** — シークレットがサーバーに出ない

---

## Web Crypto API での署名

```js
const key = await crypto.subtle.importKey(
  "raw",
  new TextEncoder().encode(secret),
  { name: "HMAC", hash: "SHA-256" },
  false, ["sign"]
);
const sig = await crypto.subtle.sign("HMAC", key,
  new TextEncoder().encode(header + "." + payload)
);
```

署名はbase64url形式（`+`→`-`、`/`→`_`、`=`除去）でエンコードします。

---

## テスト

Node.jsで77件のテストを実装：

- base64urlエンコードの正確性
- JWT 3パート構造の検証
- HMAC アルゴリズムマッピング（HS256→SHA-256など）
- 有効期限チェック（期限切れ・有効）
- エラーケース：無効JSON・不正なJWT形式

**77 / 77 PASS ✅**

---

## devnestioについて

無料の開発者ツールを集めたサイトです。すべてブラウザ完結、外部依存なし。

👉 https://devnestio.pages.dev
