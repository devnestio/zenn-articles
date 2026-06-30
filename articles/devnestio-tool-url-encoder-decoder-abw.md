---
title: "URLエンコード/デコードをより詳しく — devnestioのURL Encoder/Decoder（ABW版）を作った話"
emoji: "🔗"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "cloudflare"]
published: false
---

## 一言で言うと

URLの特殊文字をパーセントエンコーディングで変換・逆変換するツールです。文字ごとの変換内訳、URLパーツ分解（protocol/host/path/query/hash）、バッチモードにも対応した高機能版です。

👉 https://url-encoder-decoder-abw.pages.dev

devnestio（ https://devnestio.pages.dev ）は、英語圏の開発者向けに無料ツールを少しずつ公開しているサイトです。URL Encoder/Decoderの改良版について書いてみます。

## なぜ作ったのか

以前のURL Encoder/Decoderに加えて、「どの文字がどう変換されたか」の詳細や、URLを構成要素ごとに分解して確認したいという需要に応えるため、機能を大幅に拡張しました。

## 主な機能

- **2つのエンコードモード**: `encodeURIComponent`（パーツ用）と `encodeURI`（URL全体用）
- **文字ごとの変換内訳**: どの文字がどのエンコード結果になったか一覧表示
- **URLパーツ分解**: protocol / host / path / query / hash に自動分解
- **バッチモード**: 複数のURLを一括でエンコード/デコード
- **即時コピー**: 結果をワンクリックでクリップボードへ

## 技術的なポイント

`URL` APIを活用してURLのパース・分解を実装しています。`encodeURI` と `encodeURIComponent` の違いを視覚的に理解できるよう、変換前後の文字を並べて表示しています。

## まとめ

APIの開発やWebスクレイピング、SEO対策など、URLを扱う場面で役立つツールです。より詳細な情報が必要なときにご活用ください。
