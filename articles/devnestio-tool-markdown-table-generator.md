---
title: "MarkdownテーブルジェネレーターをスプレッドシートUIで作った話（CSV取込・HTML出力付き）"
emoji: "📊"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "markdown"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に **Markdown Table Generator** を追加しました。スプレッドシートのようなグリッドUIでデータを入力するか、CSVを貼り付けると、GitHub/Zenn/Qiita対応のMarkdownテーブルを即座に生成します。

👉 https://markdown-table-generator-50m.pages.dev

## なぜ作ったか

Markdownでテーブルを手書きするのは辛い。列を追加するたびに全行のパイプ文字を揃え直す必要があり、列幅を揃えるのも手間です。「Excelで整えたデータをMarkdownにしたい」という需要も多い。

## 主な機能

- **グリッドUI** — セルをクリックして直接編集、Tab/Enterで移動
- **行・列の追加削除** — ボタン一発
- **CSVからのインポート** — コピペで一括取込
- **アライメント設定** — 左寄せ・中央・右寄せを列ごとに設定
- **Markdownコードのコピー** — `:---:`, `---`, `---:` の標準形式
- **HTMLテーブルとしての出力** — `<table>`タグ形式でも書き出し可能
- **列幅の自動調整** — セルの最大文字数に合わせてパイプを揃える

## 技術的なポイント：パイプ揃えのアルゴリズム

GitHubのMarkdownは実はパイプが揃っていなくても表示されますが、ソースを読む人への配慮として列幅を揃えるオプションを付けています。

```js
function buildAlignedTable(rows, alignments) {
  // 各列の最大幅を計算
  const colWidths = rows[0].map((_, ci) =>
    Math.max(...rows.map(r => (r[ci] || '').length), 3)
  );
  
  return rows.map((row, ri) => {
    const cells = row.map((cell, ci) => {
      const w = colWidths[ci];
      const a = alignments[ci];
      if (a === 'center') return ` ${cell.padStart(Math.floor((w + cell.length) / 2)).padEnd(w)} `;
      if (a === 'right')  return ` ${cell.padStart(w)} `;
      return ` ${cell.padEnd(w)} `;
    });
    return `|${cells.join('|')}|`;
  }).join('\n');
}
```

ヘッダー行の次に区切り行（`---`）を挿入するとき、アライメントに応じて`---`, `:---:`, `---:`の形式を選択します。

## テスト結果

120件以上のテストスイートで全PASS。CSV解析（引用符内のカンマ・改行対応）、列幅自動計算、アライメント文字列生成、特殊文字エスケープを網羅しています。

## 実装はAIエージェントに任せた

Claude Codeに「スプレッドシート的なUX」「CSVインポート」「HTML出力」を要件として渡しました。Tab/Enterによるセル移動など、細かいUXの指定を発注書に含めたことで使いやすいものが一発で仕上がりました。

## 使ってみてください

👉 **[devnestio - Developer Tools Hub](https://devnestio.pages.dev)**

READMEやZennの記事でテーブルを書くとき、ぜひ。
