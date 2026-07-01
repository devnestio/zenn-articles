---
title: "テキスト行の並び替え・重複削除・フィルター ツールを作った (Line Sorter)"
emoji: "🔀"
type: "tech"
topics: ["javascript", "webdev", "tools", "text"]
published: false
---

ブラウザだけで動くテキスト行操作ツール「Line Sorter & Transformer」を作りました。

**ツール:** https://devnestio.pages.dev/line-sorter/

## 機能一覧

- **並び替え:** A→Z / Z→A / 数値昇順・降順 / 長さ順 / シャッフル / 逆順
- **重複削除:** Set を使って高速除去（大文字小文字無視オプションあり）
- **フィルター:** 正規表現フィルター（マッチ・非マッチ選択）
- **検索置換:** 文字列 or 正規表現で一括置換
- **プレフィックス/サフィックス追加:** 全行に一括付与
- **"入力として使用":** 出力を入力に戻してチェーン操作

## 実装のポイント

### Fisher-Yates シャッフル

```js
function sortLines(mode) {
  let lines = prep(getLines('input-area'));
  if (mode === 'shuffle') {
    for (let i = lines.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [lines[i], lines[j]] = [lines[j], lines[i]];
    }
  }
  setOutput(lines);
}
```

### Set による重複削除

```js
function dedup() {
  const seen = new Set();
  setOutput(lines.filter(l => {
    const key = ci(l); // 大文字小文字無視
    if (seen.has(key)) return false;
    seen.add(key); return true;
  }));
}
```

### ライン差分表示

入力と出力の行数差を +/- でリアルタイム表示。削除なら緑、追加なら赤。

```js
const diff = lines.length - inLines;
el.textContent = (diff > 0 ? '+' : '') + diff + ' lines';
el.style.color = diff < 0 ? '#4ade80' : diff > 0 ? '#f87171' : '#64748b';
```

## まとめ

82 テストすべてパス。データはすべてブラウザ内で処理されます。

[DevNestio](https://devnestio.pages.dev) — ブラウザだけで動く開発者ツール集
