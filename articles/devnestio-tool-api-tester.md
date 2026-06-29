---
title: "APIリクエストテスターをリリース — ブラウザ上でGET/POST/PUT/DELETEを送信"
emoji: "🚀"
type: "idea"
topics: ["api", "javascript", "webdev", "tools"]
published: true
---

## API Request Tester

ブラウザ上でHTTPリクエストを送信できる無料ツールをリリースしました。Postmanのようなシンプルなインターフェースです。

🔗 **[api-tester-dev.pages.dev](https://api-tester-dev.pages.dev)**

## 主な機能

- **7つのHTTPメソッド**: GET / POST / PUT / PATCH / DELETE / HEAD / OPTIONS
- **カスタムヘッダー**: キーと値を追加・削除
- **リクエストボディエディタ**: JSONフォーマット・サンプル挿入ボタン
- **クエリパラメータエディタ**: URLを自動構築
- **レスポンス表示**:
  - ステータスバッジ（2xx緑・3xx黄・4xx/5xx赤）
  - レスポンスタイム（ms）・サイズ（B/KB）
  - JSONシンタックスハイライト
  - レスポンスヘッダーテーブル
- **クイックサンプル**: JSONPlaceholder・ipify・httpbin など5種類

## 使い方

1. メソッドを選択
2. URLを入力
3. ヘッダー・ボディ・パラメータを設定
4. 「Send」をクリック

⚠ ブラウザから直接リクエストを送るためCORS制限が適用されます。パブリックAPIで動作します。

## devnestio について

🔗 [devnestio.pages.dev](https://devnestio.pages.dev)
