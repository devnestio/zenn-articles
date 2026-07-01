---
title: "Set-Cookieヘッダーをブラウザで解析する — XSS/CSRFリスクも判定（84テスト）"
emoji: "🍪"
type: "tech"
topics: ["security", "http", "cookie", "javascript", "devtools"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に、**HTTP Cookie Inspector** を追加しました。

👉 https://devnestio.pages.dev/cookie-inspector/

Set-Cookieヘッダーを貼り付けると、全属性を構造化表示＋セキュリティスコアを出します。

---

## 主な機能

- **属性カード表示** — name, value, expires, max-age, domain, path, Secure, HttpOnly, SameSite
- **セキュリティスコア（0〜100点）** — Secure+25, HttpOnly+25, SameSite≠None+25, 期限設定+25
- **XSS/CSRFリスク警告** — HttpOnly未設定→XSSリスク, SameSite未設定→CSRFリスク
- **カラー構文ハイライト** — 生ヘッダーを属性種別で色分け
- **プリセット** — セッション, 永続, Secure+HttpOnly, SameSite=Strict, 最小構成

---

## Cookieセキュリティフラグ早見表

| フラグ | 未設定のリスク | 設定のメリット |
|--------|--------------|--------------|
| `Secure` | HTTPでも送信 | HTTPSのみ送信 |
| `HttpOnly` | JS経由で盗難（XSS） | `document.cookie`からアクセス不可 |
| `SameSite=Strict` | CSRF攻撃可能 | クロスサイトリクエストで送信しない |
| `SameSite=Lax` | 一部CSRF | トップレベルナビゲーションのみ |

---

## テスト

**84 / 84 PASS ✅**

全属性パース、スコア計算、XSS/CSRFロジック、プリセット、HTMLエスケープを検証。

👉 https://devnestio.pages.dev
