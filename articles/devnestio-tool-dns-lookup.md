---
title: "DNS Lookupツール：A/MX/TXT/CNAMEレコードをブラウザで検索"
emoji: "🌐"
type: "idea"
topics: ["dns", "networking", "javascript", "webdev", "tools"]
published: false
---

## DNS Lookup Tool — 無料オンラインツール

[DNS Lookup Tool](https://dns-lookup-abg.pages.dev) は、DNS-over-HTTPS（DoH）を使ってブラウザから直接DNSレコードを検索できる無料ツールです。

## 主な機能

- **8種類のレコードタイプ** — A, AAAA, MX, TXT, CNAME, NS, SOA, PTR
- **3つのリゾルバー** — Cloudflare (1.1.1.1)、Google (8.8.8.8)、Quad9 (9.9.9.9)
- **TTL表示** — 人間が読みやすい形式（30s、5m、1h、7dなど）
- **コピー機能** — 個別レコードまたは全結果を一括コピー
- **検索履歴** — 最近検索したドメインを素早く再クエリ
- **URL自動正規化** — `https://` やパスを貼り付けても自動除去
- **サーバー不要** — DNS-over-HTTPS（DoH）を直接使用

## 対応レコードタイプ

| タイプ | 用途 |
|--------|------|
| A | IPv4アドレス |
| AAAA | IPv6アドレス |
| MX | メールサーバー（優先度付き） |
| TXT | SPF、DKIM、確認トークン |
| CNAME | 別ドメインへのエイリアス |
| NS | 権威ネームサーバー |
| SOA | ゾーン権威情報 |
| PTR | 逆引きDNS |

## リンク

- ツール: https://dns-lookup-abg.pages.dev
- DevNest.io コレクション: https://devnestio.pages.dev
