---
title: "モックAPIをブラウザで定義してjson-server/MSW/Expressに出力する"
emoji: "🛠️"
type: "tech"
topics: ["claude", "ai", "devtools", "javascript", "cloudflare"]
published: false
---

## 一言で言うと

モックHTTPエンドポイントをGUIで定義し、json-server用のdb.json・MSW（Mock Service Worker）のhandlers.js・Express.jsのmock-server.jsとしてエクスポートできるブラウザツールです。

👉 https://http-mock-server.pages.dev

devnestio（ https://devnestio.pages.dev ）は、英語圏の開発者向けに無料ツールを少しずつ公開しているサイトです。今回はHTTP Mock Server Configuratorについてbuilding-in-publicで書いてみます。

## なぜ作ったのか

フロントエンド開発中に「バックエンドAPIがまだない」という状況はよくあります。json-serverやMSWをセットアップするとき、設定ファイルを手書きするのが意外と手間です。GUIで定義してコードを生成できれば、セットアップ時間を大幅に短縮できると思って作りました。

## 主な機能

- **エンドポイント定義**: メソッド（GET/POST/PUT/PATCH/DELETE）・パス・ステータスコード・レスポンスヘッダー・JSONボディを入力
- **3つのエクスポート形式**:
  - **json-server**: `db.json` 形式
  - **MSW**: `export const handlers = [...]` 形式（MSW v2対応）
  - **Express.js**: そのまま実行できる `mock-server.js`
- **クイックスタート表示**: 選択したツールに応じたセットアップコマンドを表示
- **コピー & ダウンロード**: 生成コードをクリップボードコピーまたはファイルダウンロード
- **サンプルデータ**: 起動時にユーザーAPIのサンプルエンドポイントを自動ロード

## 技術的なポイント

MSW v2では `rest` ではなく `http` と `HttpResponse` を使うAPIに変わっています。このツールは最新のMSW v2形式に対応した `http.get(...)` / `HttpResponse.json(...)` 形式でコードを生成します。

## まとめ

「APIのモックを手書きするのが面倒」という開発者の時間を節約するツールです。定義してコピーして貼るだけで、フロントエンド開発をすぐに始められます。
