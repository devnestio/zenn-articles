---
title: "SSL証明書チェッカー：有効期限・SANs・チェーン情報をブラウザで確認"
emoji: "🔒"
type: "idea"
topics: ["ssl", "security", "javascript", "webdev", "tools"]
published: false
---

## SSL Certificate Checker — 無料オンラインツール

[SSL Certificate Checker](https://ssl-checker-abh.pages.dev) は、ドメイン検索またはPEM貼り付けでSSL証明書を検査できる無料ツールです。

## 主な機能

- **ドメイン検索** — crt.sh証明書透明性ログ経由で証明書情報を取得
- **PEM貼り付け** — 組み込みASN.1 DERデコーダーで任意の証明書をパース
- **有効期限ステータス** — 残り日数をカラーコードのプログレスバーで表示
- **自己署名検出** — CNと発行者CNが一致する場合にフラグ表示
- **SAN一覧** — ワイルドカードをハイライト表示した全サブジェクト代替名
- **証明書チェーン** — エンドエンティティと発行者/CAを表示
- **詳細情報** — シリアル番号、署名アルゴリズム、鍵アルゴリズム、鍵サイズ

## 確認できる内容

| フィールド | 情報 |
|-----------|------|
| 有効期限 | 残り日数、期限切れ/警告/有効ステータス |
| コモンネーム | サブジェクトCN |
| 発行者 | 認証局 |
| SAN | 対象ドメイン全件（ワイルドカードフラグ付き） |
| シリアル | 16進数シリアル番号 |
| アルゴリズム | RSA/ECDSAとハッシュ |

## リンク

- ツール: https://ssl-checker-abh.pages.dev
- DevNest.io コレクション: https://devnestio.pages.dev
