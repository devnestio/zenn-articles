---
title: ".envファイルをブラウザで管理・変換するツールを作った（JSON/shell export対応）"
emoji: "🔐"
type: "tech"
topics: ["dotenv", "devtools", "cloudflare", "javascript", "nodejs"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に、**Env Var Manager** を追加しました。

👉 https://env-var-manager.pages.dev

`.env` ファイルをブラウザに貼り付けると、テーブル形式で編集・検索でき、`.env` / `JSON` / `shell export` の3形式でエクスポートできます。外部送信なし・単一HTMLファイル・依存ゼロです。

---

## 作った動機

プロジェクトごとに `.env`、`docker-compose.yml`、`GitHub Actions secrets`、デプロイスクリプトの間で環境変数を使い回す場面が多くあります。

```bash
# .envから読んで
source .env

# docker-composeのenv_fileに渡して
env_file:
  - .env

# CI/CDのsecretに設定して
echo "SECRET=$SECRET" >> $GITHUB_OUTPUT
```

この変換・コピー作業を手作業でやると、タイポ・重複キー・空のまま放置した値を見落としがちです。特に `.env` → JSON（API呼び出し用）→ shell export（スクリプト用）という流れは頻繁に発生するのに、ツールがない状態でした。

---

## 主な機能

### 1. .env パース

```
# Application config
APP_NAME=MyWebApp
DATABASE_URL=postgres://admin:secret@localhost:5432/mydb
PORT=3000          # default port
SECRET_KEY="has spaces and #hash"
EMPTY_VALUE=
```

上記を貼り付けてParseボタンを押すと：
- コメント行（`# ...`）はコメントとして認識
- インラインコメント（`value # comment`）は値から除去
- クォートされた値（`"..."` / `'...'`）は自動アンクォート
- `KEY=A=B=C` のようなイコール含む値も正しく処理

### 2. テーブルエディタ

| KEY | VALUE | |
|-----|-------|---|
| APP_NAME | MyWebApp | × |
| DATABASE_URL | postgres://... | × |

- キーと値を直接編集
- 検索バーでフィルタリング
- 行削除ボタン
- 新規変数の追加フォーム

### 3. 統計ダッシュボード

変数数・コメント数・空の値の数・**重複キーの数**をリアルタイム表示。デプロイ前に重複や空の秘密鍵を検出できます。

### 4. エクスポート形式

**.env形式**（デフォルト）:
```
# Application config
APP_NAME=MyWebApp
DATABASE_URL=postgres://admin:secret@localhost:5432/mydb
SECRET_KEY="has spaces and #hash"
EMPTY_VALUE=""
```
スペースや`#`を含む値は自動クォート。

**JSON形式**:
```json
{
  "APP_NAME": "MyWebApp",
  "DATABASE_URL": "postgres://admin:secret@localhost:5432/mydb",
  "EMPTY_VALUE": ""
}
```

**Shell export形式**:
```bash
export APP_NAME=MyWebApp
export DATABASE_URL=postgres://admin:secret@localhost:5432/mydb
export SECRET_KEY="has spaces and #hash"
```
bashスクリプトや `.bashrc` に直接貼れる形式。

---

## 技術的なポイント

### パーサーの実装

外部ライブラリ（`dotenv` など）を使わず、手書きパーサーで実装しています。

```javascript
function parseEnvText(text) {
  const lines = text.split('\n');
  const result = [];
  for (const line of lines) {
    const trimmed = line.trim();
    if (trimmed === '') { result.push({ empty: true }); continue; }
    if (trimmed.startsWith('#')) { result.push({ comment: true, text: trimmed }); continue; }
    const eqIdx = trimmed.indexOf('=');
    if (eqIdx === -1) { /* invalid line → treat as comment */ continue; }
    const key = trimmed.slice(0, eqIdx).trim();
    let value = trimmed.slice(eqIdx + 1); // 最初の=以降がすべて値
    value = stripInlineComment(value);
    value = unquote(value);
    result.push({ key, value });
  }
  return result;
}
```

`VALUE=a=b=c` のようなケースは `indexOf('=')` の結果（最初の`=`位置）をキーの終わりとし、それ以降をすべて値として扱います。

### クォートが必要な値の自動判定

```javascript
function quoteIfNeeded(val) {
  if (val.includes(' ') || val.includes('#') || val === '') return `"${val}"`;
  return val;
}
```

スペース・`#`・空文字列の場合のみダブルクォートで囲みます。URLやパスワードは通常そのまま出力されます。

---

## テスト結果

Node.js の `assert` モジュールで **81件のテスト**をすべてパス。

```
[parseEnvText]    20 tests
[unquote]          8 tests
[quoteIfNeeded]    6 tests
[renderEnv]        6 tests
[renderJson]       7 tests
[renderShell]      5 tests
[getStats]         7 tests
[integration]     14 tests + 8 extras
────────────────────────────────────────
Results: 81 passed, 0 failed
```

---

## 使ってみてください

👉 **[Env Var Manager — devnestio](https://env-var-manager.pages.dev)**

全ツール一覧: https://devnestio.pages.dev

フィードバック（対応できていない `.env` の書き方・エクスポート形式の要望など）はコメントで大歓迎です。
