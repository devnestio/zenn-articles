---
title: "ブラウザだけで動くDNSルックアップツールを作った話（DNS-over-HTTPS、149テスト）"
emoji: "🔎"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "dns"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に **DNS Lookup Tool** を追加しました。A・AAAA・MX・TXT・NS・CNAME・SOA・CAA・SRVレコードをブラウザだけでクエリできるツールです。サーバーサイドの処理は一切なく、DNS-over-HTTPS（DoH）を使ってブラウザから直接DNSを叩きます。

👉 https://dns-lookup-tool-bd4.pages.dev

## なぜ作ったか

`dig`や`nslookup`コマンドが手元にない環境（Windowsのコマンドプロンプト、他人のPCなど）でDNSを確認したい場面があります。既存のWebサービスはサーバーが代わりにDNSを叩いてくれる仕組みが多く、プライバシーが気になります。

DNS-over-HTTPSなら**ブラウザからGoogle/CloudflareのDoHエンドポイントへ直接クエリできる**ため、サーバーを通さずに済みます。

## 主な機能

- **9種のレコードタイプ対応** — A, AAAA, MX, TXT, NS, CNAME, SOA, CAA, SRV
- **DoHプロバイダーの選択** — Google（`dns.google`）/ Cloudflare（`1.1.1.1`）
- **MXレコードの優先度順ソート**
- **クエリ履歴** — 直近の検索を記憶
- **結果のコピー** — 生テキストでクリップボードに

## 技術的なポイント：DNS-over-HTTPS

DoHはHTTPSのGETリクエストでDNSクエリを送る仕組みです。JSONレスポンス形式（RFC 8427）を使うと、JavaScriptで扱いやすい構造が返ってきます。

```js
async function queryDoH(domain, type, provider = 'google') {
  const base = provider === 'google'
    ? 'https://dns.google/resolve'
    : 'https://cloudflare-dns.com/dns-query';
  const url = `${base}?name=${encodeURIComponent(domain)}&type=${encodeURIComponent(type)}`;
  
  const res = await fetch(url, {
    headers: { 'Accept': 'application/dns-json' }
  });
  if (!res.ok) throw new Error(`DoH error: ${res.status}`);
  return res.json();
}
```

レスポンスの`Answer`配列にレコードが入っています。MXレコードは`data`フィールドが`"10 mail.example.com."`の形式なので、優先度と交換先ホストを分割してソートします。

```js
function parseMX(data) {
  const [prio, host] = data.split(' ');
  return { priority: parseInt(prio), host: host.replace(/\.$/, '') };
}
```

## テスト結果

**149/149テスト PASS**。URLのビルドロジック、各レコードタイプのパース、MXの優先度ソート、TXTの引用符処理、CAA・SRVのフィールド抽出などを確認しています。ブラウザ外（Node.js）で実際のDoHクエリは発火しませんが、レスポンス処理ロジックは全て単体テストでカバーしています。

## 実装はAIエージェントに任せた

Claude Codeに発注書を渡して実装してもらいました。「実際のネットワークに依存しない単体テストを150件書くこと」という要件を入れたことで、モックレスポンスでパース処理を網羅したテストスイートが得られました。

## 使ってみてください

👉 **[devnestio - Developer Tools Hub](https://devnestio.pages.dev)**

ドメインのMXレコードを素早く確認したいときや、DNSの伝播確認にどうぞ。
