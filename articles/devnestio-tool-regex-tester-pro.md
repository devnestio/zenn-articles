---
title: "Regex Tester Pro を作った — ライブハイライト、名前付きグループ、40+パターンライブラリ"
emoji: "🔍"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "javascript"]
published: false
---

## どんなツール？

正規表現のデバッグは思いのほか手間がかかる。マッチしているのかしていないのか、グループが正しく取れているのか、フラグの組み合わせが合っているのか…

**Regex Tester Pro** は、そういった確認をリアルタイムで行える高機能な正規表現テスターです。

👉 https://regex-tester-pro-aba.pages.dev

devnestioには既にライン GにシンプルなRegex Testerがありますが、こちらはより高機能なバージョンとして別途作成しました。

## 主な機能

### ライブマッチハイライト
入力するたびにリアルタイムでハイライト。マッチが複数ある場合は色を変えて区別（最大5色サイクル）。

### 全6フラグ対応
`g`（グローバル）、`i`（大文字小文字無視）、`m`（複数行）、`s`（dotAll）、`u`（Unicode）、`y`（スティッキー）をチェックボックスで切り替え。

### マッチ詳細テーブル
各マッチについて以下を表示：
- マッチ番号
- マッチした文字列
- 開始インデックス
- 終了インデックス
- キャプチャグループ（$1, $2...）
- 名前付きグループ（`(?<name>...)` の値）

### 置換モード
`$1`、`$2`、`$<name>` を使った置換結果をリアルタイム確認。

### 40+パターンライブラリ
クリックするだけで正規表現をセット：

| パターン名 | 用途 |
|-----------|------|
| Email | メールアドレス |
| UUID | RFC 4122 UUID |
| URL | HTTP/HTTPS URL |
| IPv4/IPv6 | IPアドレス |
| Hex Color | CSS カラーコード |
| JWT Token | JSON Web Token |
| Semantic Version | v1.2.3-beta |
| Git Commit Hash | 7〜40桁の16進数 |
| Duplicate Words | 重複単語 |
| SQL SELECT | SELECT...FROM句 |

### クイックリファレンス
`.` `\d` `\w` `\s` `^` `$` `*` `+` `?` `{n,m}` `(...)` `(?:...)` `(?<name>...)` など、正規表現の構文をすぐ確認できる表。

## 実装のポイント

```javascript
function getAllMatches(re, str, flags) {
  const results = [];
  if (flags.includes('g') || flags.includes('y')) {
    let m;
    const safeRe = new RegExp(re.source, re.flags);
    while ((m = safeRe.exec(str)) !== null) {
      results.push(matchObj(m));
      if (m.index === safeRe.lastIndex) safeRe.lastIndex++;  // 無限ループ防止
    }
  } else {
    const m = re.exec(str);
    if (m) results.push(matchObj(m));
  }
  return results;
}
```

空文字列にマッチする正規表現（例: `a*`）でループしないよう `lastIndex` を手動で進める処理が重要。

## テスト

69件のテスト全通過。

```bash
node tests/test.js
# Total: 69 | Passed: 69 | Failed: 0
```

## デプロイ

Cloudflare Pages で公開。Vanilla JS単一ファイル。

---

[devnestio](https://devnestio.pages.dev) - 無料の開発者ツール集
