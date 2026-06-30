---
title: "ブラウザだけで動くコードスニペット管理ツールを作った（19言語・タグ・検索・ローカル保存）"
emoji: "📋"
type: "tech"
topics: ["javascript", "devtools", "cloudflare", "localstorage", "productivity"]
published: true
---

## 作ったもの

開発者向け無料ツール群「devnestio」に、**Code Snippet Manager** を追加しました。

👉 https://code-snippet-manager-bdd.pages.dev

コードスニペットをブラウザ内に保存・検索・タグ管理できるツールです。サーバーなし・アカウントなし・すべてlocalStorageに保存されます。

---

## 作った動機

スニペット管理ツールの要件として考えていたのはこうでした：

- ログインなしで即使える
- 自分のコードを外部に送りたくない
- オフラインで動作する
- 起動が速く、シンプル

GitHub Gistはログインが必要で、Notionは重く、VS Codeのスニペット機能はエディタ外から見られない。「ブラウザのタブを1枚開けばすぐ使える」というツールが欲しかったのが動機です。

---

## 主な機能

### スニペット保存

- **タイトル**（必須）
- **言語**（19種類から選択）
- **説明**（任意）
- **タグ**（複数設定可能、Enterかカンマで追加）
- **コード**（必須）

### 19言語のシンタックスハイライト

```
JavaScript / TypeScript / Python / Bash / SQL
HTML / CSS / JSON / YAML / Go / Rust / Java
C# / PHP / Ruby / Swift / Kotlin / Dockerfile / Other
```

外部ライブラリなし・手書きのハイライターで実装。キーワード・文字列・数値・コメント・関数名に色付けします。

### 検索とフィルタリング

- **全文検索**：タイトル・説明・コード内容・言語名・タグを横断検索
- **言語フィルタ**：保存済みスニペットの言語のみ動的に表示
- **タグフィルタ**：タグをクリックして絞り込み

### データのエクスポート・インポート

```json
[
  {
    "id": "abc123",
    "title": "Fetch with error handling",
    "lang": "javascript",
    "desc": "Async fetch wrapper",
    "tags": ["fetch", "async", "api"],
    "code": "async function fetchJSON(url) { ... }",
    "createdAt": 1700000000000,
    "updatedAt": 1700000000000
  }
]
```

JSONとして書き出し・読み込みができます。ブラウザ間の移行やバックアップに使えます。インポート時は重複IDをスキップします。

---

## 技術的なポイント

### localStorage の使い方

```javascript
const STORAGE_KEY = 'devnestio-snippets-v1';

function load() {
  try { snippets = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]'); }
  catch { snippets = []; }
}

function save() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(snippets));
}
```

localStorageはオリジンごとに約5〜10MBの容量があります。テキストベースのスニペットなら十分です。

### タグの追加

Enterキーまたはカンマキーで即座にタグを追加します。重複は自動で除外します。

```javascript
function handleTagKey(e) {
  if (e.key === 'Enter' || e.key === ',') {
    e.preventDefault();
    addTag();
  }
}
```

### シンタックスハイライト（例：JSON）

```javascript
function hlJson(code) {
  return code
    .replace(/("(?:[^"\\]|\\.)*")\s*:/g, `<span class="tok-attr">$1</span>:`)
    .replace(/:\s*("(?:[^"\\]|\\.)*")/g, `: <span class="tok-str">$1</span>`)
    .replace(/:\s*(\b\d+\.?\d*\b)/g, `: <span class="tok-num">$1</span>`)
    .replace(/:\s*(true|false|null)\b/g, `: <span class="tok-kw">$1</span>`);
}
```

`highlight-js` などを使えば圧倒的に楽ですが、devnestioは「外部依存ゼロ」ポリシーなので手書きです。

---

## テスト結果

Node.js `assert` モジュールで **82件のテスト**をすべてパス。

```
[LANG_COLORS]              10 tests
[makeSnippet]               6 tests
[addSnippet]                8 tests
[editSnippet]               4 tests
[deleteSnippet]             4 tests
[filterSnippets]           10 tests
[getStats]                  4 tests
[getAllTags / getAllLangs]   3 tests
[toggleTag / removeTagFromList] 7 tests
[exportData / importData]   7 tests
[lineCount]                 4 tests
[Integration]               5 tests
[Additional coverage]      10 tests
────────────────────────────────────────
Results: 82 passed, 0 failed
```

---

## 使ってみてください

👉 **[Code Snippet Manager — devnestio](https://code-snippet-manager-bdd.pages.dev)**

全ツール一覧: https://devnestio.pages.dev

「この言語が足りない」「この機能が欲しい」というフィードバックはコメントで大歓迎です。
