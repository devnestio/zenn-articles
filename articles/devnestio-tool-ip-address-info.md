---
title: "IPアドレスの位置情報・ISP・ASNをブラウザで調べるツールを作った"
emoji: "🌐"
type: "tech"
topics: ["network", "devtools", "javascript", "cloudflare", "ip"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に、**IP Address Info** を追加しました。

👉 https://devnestio.pages.dev/ip-address-info/

任意のIPv4/IPv6アドレスのジオロケーション（国・都市・地域）、ISP、ASN、タイムゾーンなどをブラウザ上で取得できます。APIキー不要。

---

## 作った動機

VPNの出口確認、サービスからのリクエスト元確認、ネットワークデバッグなど、「このIPは何者？」を素早く確認したい場面は多い。curl + ip-api.comで済むことが多いですが、UIがあると楽なので作りました。

---

## 機能

- **IPルックアップ**: 国・地域・都市・郵便番号・タイムゾーン・ISP・ASN・ホスト名
- **自分のIPを自動表示**: ページロード時に自分のIPを検出
- **プライベートIP判定**: RFC1918やループバックは「Private」バッジ表示
- **地図リンク**: 緯度経度からOpenStreetMapへリンク
- **JSON出力コピー**: 取得データをそのままJSONでクリップボードへ
- **検索履歴**: 最近10件をlocalStorageで保持

---

## 技術

- 使用API: [ip-api.com](https://ip-api.com)（無料・APIキー不要）
- 単一HTMLファイル・ゼロ依存
- Cloudflare Pagesでホスティング

---

## 関連ツール

👉 https://devnestio.pages.dev/
