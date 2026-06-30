---
title: "Content Security Policyヘッダーをビジュアルで生成するツールを作った"
emoji: "🛡️"
type: "tech"
topics: ["csp", "security", "devtools", "cloudflare", "javascript"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に、**CSP Generator** を追加しました。

👉 https://csp-generator.pages.dev

Content Security PolicyヘッダーをGUI操作で組み立て、完成したヘッダー文字列をコピーできます。外部送信なし・単一HTMLファイルです。

---

## CSPを手書きするのが辛い理由

CSP文字列はこんな見た目です：

```
Content-Security-Policy: default-src 'none'; script-src 'self' https://cdn.jsdelivr.net; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; img-src 'self' data: https:; font-src 'self' https://fonts.gstatic.com; connect-src 'self' https:; object-src 'none'; frame-ancestors 'none'; upgrade-insecure-requests
```

問題点：
- `'self'` の引用符を一つ外すだけでディレクティブが無効になる
- ディレクティブ名を1文字でも間違えるとサイレントに失敗する
- 各ディレクティブが何をブロックするか覚えていないと設定できない
- `script-src 'unsafe-inline'` を入れると実質CSPが無効化されることに気づきにくい

---

## 主な機能

### 4つのクイックプリセット

| プリセット | 用途 |
|---|---|
| 🔒 Strict | `default-src 'none'` から始める最厳格設定。新規プロジェクト向け |
| 🛡️ Moderate | `default-src 'self'` + unsafe-inline許可。既存コードがある場合 |
| ⚡ SPA / CDN | JSDelivr・unpkg・Google Fonts許可。CDN利用SPAの出発点 |
| 🔌 API Only | HTML未配信のAPIサーバー向け。script/style全ブロック |

### 16のディレクティブをトグルで制御

- `default-src` / `script-src` / `style-src` / `img-src` / `font-src`
- `connect-src` / `frame-src` / `media-src` / `object-src`
- `frame-ancestors` / `form-action` / `base-uri`
- `upgrade-insecure-requests` / `block-all-mixed-content`
- `report-uri` / `report-to`

### ソース値をクリックで追加

各ディレクティブに対して、よく使う値がタグ形式で表示されます：

**キーワード値（青）**: `'none'` `'self'` `'unsafe-inline'` `'strict-dynamic'` など  
**ホスト名（緑）**: `https://cdn.jsdelivr.net` `https://fonts.gstatic.com` など  
**スキーム（紫）**: `data:` `blob:` など

クリックで選択状態をトグル。カスタム入力フィールドで任意のURLも追加可能。

### unsafe-inline 警告

`'unsafe-inline'` や `'unsafe-eval'` が含まれる場合、警告バナーが表示されます。

```
⚠️ unsafe-inline or unsafe-eval significantly weakens CSP protection.
   Consider using nonces or hashes instead.
```

### 出力形式

生成されたCSPは2つの形式でコピーできます：

```http
# HTTP ヘッダー形式
Content-Security-Policy: default-src 'none'; script-src 'self'; ...
```

```html
<!-- HTML meta タグ形式 -->
<meta http-equiv="Content-Security-Policy" content="default-src 'none'; ...">
```

---

## Strictプリセットで生成されるCSP

```
default-src 'none'; script-src 'self'; style-src 'self'; img-src 'self' data:; font-src 'self'; connect-src 'self'; object-src 'none'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'; upgrade-insecure-requests
```

このCSPが意味すること：
- `default-src 'none'` — 明示的に許可していないリソースはすべてブロック
- `object-src 'none'` — Flash等のプラグイン完全無効化
- `frame-ancestors 'none'` — clickjacking防止（X-Frame-Options: DENYの代替）
- `base-uri 'self'` — `<base>`タグ経由のDOM injection防止
- `form-action 'self'` — フォームの外部送信防止
- `upgrade-insecure-requests` — HTTP→HTTPSの自動アップグレード

---

## テスト結果

Node.js `assert` モジュールで **81件のテスト**をすべてパス。

```
[DIRECTIVES data]     14 tests
[initState]            4 tests
[buildCSPValue]        9 tests
[Presets]             13 tests
[hasUnsafe]            5 tests
[countDirectives]      5 tests
[Integration]          4 tests
[Additional coverage] 27 tests
────────────────────────────────────────
Results: 81 passed, 0 failed
```

---

## 使ってみてください

👉 **[CSP Generator — devnestio](https://csp-generator.pages.dev)**

全ツール一覧: https://devnestio.pages.dev

「このディレクティブが足りない」「このプリセットの設定がおかしい」というフィードバックはコメントで大歓迎です。
