---
title: "URLパーサー＆ビルダーをVanilla JSで作った — ブラウザのURL APIを活かしたツール"
emoji: "🔗"
type: tech
topics: ["javascript", "webdev", "tools", "url"]
published: false
---

## これは何？

[DevNest.io](https://devnestio.pages.dev/) のツールとして作ったURLパーサー＆ビルダーです。

**URL**: https://url-parser-builder.pages.dev/

URLを分解して各コンポーネント（スキーム、ホスト、パス、クエリパラメータなど）を表示する「Parser」タブと、各コンポーネントを入力してURLを組み立てる「Builder」タブを備えたツールです。

## 実装した機能

### Parser タブ
- スキーム、ユーザー情報、ホスト、ポート、パス、クエリ、フラグメント を個別表示
- クエリパラメータをテーブルで表示（キー / 値 / デコード値）
- Origin と完全URL のコピーボタン

### Builder タブ
- スキーム、認証情報、ホスト、ポート、パス、フラグメント の入力フィールド
- クエリパラメータの動的追加・削除
- リアルタイムプレビューでURL生成

## 技術的なポイント

### ブラウザのURL APIを使う

```js
function parseURL(raw) {
  const trimmed = raw.trim();
  if (!trimmed) return { error: 'Please enter a URL.' };
  try {
    const u = new URL(trimmed);
    const params = [];
    u.searchParams.forEach((v, k) => params.push({
      key: k,
      value: v,
      decoded: decodeURIComponent(v)
    }));
    return {
      scheme: u.protocol.replace(/:$/, ''),
      username: u.username || '',
      password: u.password || '',
      host: u.hostname,
      port: u.port || '',
      path: u.pathname,
      query: u.search.replace(/^\?/, ''),
      fragment: u.hash.replace(/^#/, ''),
      origin: u.origin,
      href: u.href,
      params,
    };
  } catch(e) {
    return { error: `Invalid URL: ${e.message}` };
  }
}
```

正規表現でURLを自前解析するのではなく、ブラウザ標準の `URL` コンストラクタに委ねます。RFC 3986 準拠のパースが無料で手に入り、エッジケースも自動処理されます。

### URLビルダー

```js
function buildURL({ scheme, username, password, host, port, path, fragment, params }) {
  if (!scheme || !host) return null;
  let url = `${scheme}://`;
  if (username) {
    url += encodeURIComponent(username);
    if (password) url += ':' + encodeURIComponent(password);
    url += '@';
  }
  url += host;
  if (port) url += ':' + port;
  url += path || '/';
  const qs = params
    .filter(p => p.key.trim())
    .map(p => `${encodeURIComponent(p.key)}=${encodeURIComponent(p.value)}`)
    .join('&');
  if (qs) url += '?' + qs;
  if (fragment) url += '#' + fragment;
  return url;
}
```

ユーザー名・パスワード・パラメータはすべて `encodeURIComponent` でエンコードします。ホスト名はエンコードしない（`example.com` はそのまま、IPv6は `[...]` 形式で保持）。

### XSS対策 — escHtml

```js
function escHtml(s) {
  return String(s)
    .replace(/&/g,'&amp;')
    .replace(/</g,'&lt;')
    .replace(/>/g,'&gt;')
    .replace(/"/g,'&quot;');
}
```

パースした値をHTMLとして出力する前に必ずエスケープします。URLにはスクリプト文字列が含まれる可能性があるため重要です。

## テスト

Node.js `assert` モジュールで **120テスト** を実装、全パス確認済み。

主なテストカテゴリ:
- `parseURL` — 正常系（フル URL、最小 URL、認証情報付き）
- `parseURL` — エラー系（空文字、スキームなし）
- `parseURL` — エッジケース（国際化ドメイン、IPv6、データURL）
- `buildURL` — 正常系（パラメータあり・なし、認証情報あり）
- `buildURL` — エッジケース（空パラメータのスキップ、フラグメント）
- `escHtml` — XSSインジェクション文字のエスケープ

## 使い方

https://url-parser-builder.pages.dev/ にアクセスして、ParserタブにURLを貼り付けるか、Builderタブで各フィールドを入力してください。

---

DevNest.io では他にも開発者向けツールを公開しています。
