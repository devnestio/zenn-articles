---
title: "API Response FormatterをVanilla JSで作った話（JSON/XML/YAMLの整形・検証・Diff）"
emoji: "📋"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "javascript"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に **API Response Formatter** を追加しました。JSON・XML・YAMLをブラウザ内で整形・ミニファイ・バリデーション・Diffできます。

👉 https://api-response-formatter.pages.dev

## なぜ作ったか

APIのレスポンスをcurlで取得して確認するとき、ワンライナーのJSONが読みにくくて毎回`| python3 -m json.tool`でパイプしていました。また2つのレスポンスを比較するとき、手動でdiffするのが面倒でした。ブラウザで開いてペーストするだけで整形・比較できるツールが欲しかった。

## 主な機能

- **3フォーマット対応** — JSON / XML / YAML
- **整形（Pretty-print）** — インデント2スペースで読みやすく整形
- **ミニファイ** — 不要なホワイトスペースを除去してコンパクトに
- **バリデーション** — 入力中にリアルタイムでエラー検出
- **シンタックスハイライト** — キー・文字列・数値・真偽値・nullを色分け
- **統計情報** — 行数・文字数・バイト数を表示（ユニコード対応）
- **Diffモード** — 2つのレスポンスを並べて行単位で差分表示（追加は緑・削除は赤）
- **サンプルデータ** — ワンクリックでサンプルJSONを読み込み

## 技術的なポイント：JSONのシンタックスハイライト

DOMParserやexternal libraryを使わず、整形済み文字列に対して正規表現でHTMLタグを注入する方式です。

```javascript
function highlightJSON(str) {
  return str
    .replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;')
    .replace(/("(?:[^"\\]|\\.)*")\s*:/g, '<span class="j-key">$1</span>:')
    .replace(/:\s*("(?:[^"\\]|\\.)*")/g, ': <span class="j-str">$1</span>')
    .replace(/:\s*(-?\d+(?:\.\d+)?(?:[eE][+-]?\d+)?)/g, ': <span class="j-num">$1</span>')
    .replace(/:\s*(true|false)/g, ': <span class="j-bool">$1</span>')
    .replace(/:\s*(null)/g, ': <span class="j-null">$1</span>');
}
```

## XMLのPretty-print

DOMParserは使わず、`><`で分割→トークン列として処理→インデントを手動管理するアプローチです。

```javascript
function prettyXML(str) {
  let indent = 0;
  const tokens = str.replace(/>\s*</g, '><').split(/(?<=>)(?=<)|(?<=[^>])(?=<)/);
  tokens.forEach(tok => {
    if (tok.startsWith('</')) { indent--; }
    result.push('  '.repeat(Math.max(0, indent)) + tok);
    if (tok.startsWith('<') && !tok.startsWith('</') && !tok.endsWith('/>')) indent++;
  });
}
```

## シンプルなLCSベースDiff

Myers差分アルゴリズムの完全実装ではなく、先読みウィンドウ5行のグリーディーなアプローチです。APIレスポンスのdiffとしては十分な精度で、実装をシンプルに保てます。

```javascript
function diffLines(aLines, bLines) {
  // 一致しないとき5行先読みして最も近い一致を探す
  for (let k = 1; k <= 5; k++) {
    if (aLines[i + k] === bLines[j]) { foundA = k; break; }
    if (aLines[i] === bLines[j + k]) { foundB = k; break; }
  }
}
```

## テスト

80件以上のアサーション。JSON/XML/YAMLの整形・ミニファイ・バリデーション・ラウンドトリップ、diff（追加・削除・変更・空入力）、サイズフォーマット（B/KB）などを網羅しています。

```
$ node tests/test.js
All tests passed! (80+ assertions)
```

## ハブへの追加

devnestioのハブに「Data」カテゴリで追加しました。

---

devnestioは開発者向け無料ツール群です。ログイン不要・トラッキングなし。
