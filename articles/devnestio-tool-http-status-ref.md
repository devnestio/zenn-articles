---
title: "HTTPステータスコード早見表をブラウザで作った（検索・ユースケース・RFC付き）"
emoji: "🌐"
type: "tech"
topics: ["http", "devtools", "cloudflare", "javascript", "webapi"]
published: true
---

## 作ったもの

開発者向け無料ツール群「devnestio」に、**HTTP Status Code Reference** を追加しました。

👉 https://http-status-ref.pages.dev

1xx〜5xxまで60件以上のHTTPステータスコードを検索・フィルタリングできます。コード番号・名前・キーワードで検索でき、各コードの説明・ユースケース・RFC参照が確認できます。

---

## 作った動機

APIを設計・実装していると、こういう場面が頻繁に発生します。

- 401と403の使い分け（「認証されていない」vs「認証済みだが権限なし」）
- 302と307の違い（POSTリクエストがリダイレクト後にGETになるかどうか）
- 202を返すべきか204を返すべきか
- 429を返すときにRetry-Afterヘッダーが必要か

MDNは充実していますが、コードごとにページを開く必要があります。チームメンバーへの共有も「このリンクで確認して」という一手間があります。

「rate limit」で検索すれば429が出て、「websocket」で検索すれば101が出る——そういう横断的な検索ができる手元ツールが欲しかったのが動機です。

---

## 主な機能

### 検索

```
キーワード例:
- "404"         → 404 Not Found
- "redirect"    → 301, 302, 303, 307, 308
- "timeout"     → 408, 504
- "websocket"   → 101
- "rate limit"  → 429
- "legal"       → 451
- "cache"       → 304
```

コード番号・名前・説明・ユースケースのいずれかにマッチします。

### カテゴリフィルタ

1xx / 2xx / 3xx / 4xx / 5xx のタブで絞り込み。各タブに現在の件数バッジが表示されます。

### 詳細パネル

カードをクリックするとパネルが開き、以下が表示されます：
- **Description**: そのコードが意味することの技術的説明
- **Common Use Cases**: 実際のAPIやWebアプリでの使いどころ
- **RFC / Spec**: 出典となるRFC番号

### コピーボタン

- `404` だけコピー（レスポンスコードとして使う）
- `"404 Not Found"` の形式でコピー（ログやドキュメント用）

---

## 覚えておくと便利な区別

### 401 vs 403

| | 401 Unauthorized | 403 Forbidden |
|--|--|--|
| 認証状態 | 未認証（ログインが必要） | 認証済み（だが権限なし） |
| 名前のミスリード | 「Unauthorized」という名前だが実態は「Unauthenticated」 | こちらが本来の「Unauthorized」 |
| クライアントの対応 | ログイン画面にリダイレクト | エラー表示（ログインしても意味がない） |

### 302 vs 307

| | 302 Found | 307 Temporary Redirect |
|--|--|--|
| HTTPメソッド | リダイレクト後にGETに変わることがある | 必ず同じメソッドを維持 |
| POST後のリダイレクト | POSTがGETになる（仕様上は非推奨だが実装依存） | POSTのままリダイレクト先に送られる |
| 用途 | 旧来の一時リダイレクト（互換性のため） | メソッドを保持したい一時リダイレクト |

### 202 vs 204

| | 202 Accepted | 204 No Content |
|--|--|--|
| 処理タイミング | 非同期（まだ処理中） | 同期（処理完了） |
| レスポンスボディ | ジョブID・ステータスURLなど | なし |
| 用途 | キューへの登録、バッチ処理 | DELETE成功、PUTで本体不要 |

---

## 技術メモ

### データ構造

外部APIに依存せず、ステータスコードデータをすべてJSオブジェクトとして埋め込んでいます。

```javascript
const STATUSES = [
  {
    code: 429,
    name: "Too Many Requests",
    cat: "4xx",
    desc: "The user has sent too many requests in a given amount of time (rate limiting).",
    use: "API rate limiting. Include Retry-After header with seconds to wait or a timestamp.",
    rfc: "RFC 6585"
  },
  // ...
];
```

### 検索ロジック

```javascript
function filterStatuses(statuses, query, category) {
  const q = query.toLowerCase().trim();
  return statuses.filter(s => {
    if (category !== 'all' && s.cat !== category) return false;
    if (!q) return true;
    return String(s.code).includes(q) ||
      s.name.toLowerCase().includes(q) ||
      s.desc.toLowerCase().includes(q) ||
      s.use.toLowerCase().includes(q);
  });
}
```

コード・名前・説明・ユースケース4フィールドを横断検索します。

### テスト結果

Node.js `assert` で **81件のテスト**すべてパス。

---

## 使ってみてください

👉 **[HTTP Status Code Reference — devnestio](https://http-status-ref.pages.dev)**

全ツール一覧: https://devnestio.pages.dev

「このコードの説明が正確でない」「このユースケースが足りない」といったフィードバックはコメントで大歓迎です。
