---
title: ".htpasswdジェネレーター：bcrypt・MD5・SHA1をブラウザで生成"
emoji: "🔐"
type: "idea"
topics: ["apache", "security", "javascript", "webdev", "tools"]
published: false
---

## .htpasswd Generator — 無料オンラインツール

[.htpasswd Generator](https://htpasswd-generator-abf.pages.dev) は、Apacheの認証ファイルに使うパスワードハッシュをブラウザ内で生成できる無料ツールです。

## 主な機能

- **bcrypt** — 最もセキュア、本番環境推奨（コストファクター8〜12）
- **MD5-APR1** (`$apr1$`) — Apacheが使うMD5バリアント、ランダムソルト付き
- **SHA1** (`{SHA}`) — レガシー互換フォーマット
- **一括生成** — `username:password` 形式で複数エントリを一度に生成
- **ランダムパスワード生成** — 暗号論的乱数、強度インジケーター付き
- **ダウンロード** — `.htpasswd` ファイルとして直接保存
- **追記モード** — エントリを1件ずつ追加してファイルを構築

## 使い方

```
# 単一エントリ
admin:$2y$10$...

# 一括（username:password 形式で貼り付け）
admin:secret123
user1:mypassword
guest:guestpass
```

## リンク

- ツール: https://htpasswd-generator-abf.pages.dev
- DevNest.io コレクション: https://devnestio.pages.dev
