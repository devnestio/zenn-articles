---
title: "テキストDiffビューアをVanilla JSで作った — LCSアルゴリズムをゼロから実装"
emoji: "🔀"
type: tech
topics: ["javascript", "webdev", "algorithm", "tools"]
published: false
---

## これは何？

[DevNest.io](https://devnestio.pages.dev/) のツールとして作ったテキストDiffビューアです。

**URL**: https://text-diff-viewer.pages.dev/

2つのテキストを貼り付けると、追加行・削除行・変更なし行を行単位でハイライト表示します。外部ライブラリなし、LCSアルゴリズムをVanilla JSで実装しています。

## LCS（最長共通部分列）アルゴリズム

Diffの核心は **LCS (Longest Common Subsequence)** — 2つの配列に共通して現れる最長の部分列を求めることです。

```js
function lcs(a, b) {
  const m = a.length, n = b.length;
  // DPテーブル
  const dp = Array.from({length: m+1}, () => new Array(n+1).fill(0));
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      dp[i][j] = a[i-1] === b[j-1]
        ? dp[i-1][j-1] + 1
        : Math.max(dp[i-1][j], dp[i][j-1]);
    }
  }
  // バックトラック
  const seq = [];
  let i = m, j = n;
  while (i > 0 && j > 0) {
    if (a[i-1] === b[j-1]) { seq.unshift([i-1, j-1]); i--; j--; }
    else if (dp[i-1][j] >= dp[i][j-1]) i--;
    else j--;
  }
  return seq;
}
```

DPテーブルを O(m×n) で構築し、バックトラックで共通要素のインデックスペアを取り出します。戻り値は `[indexInA, indexInB]` の配列。

## Diff変換

LCSの共通行を基準に、A・B それぞれから `del`・`add`・`eq` ハンクを生成します。

```js
function diff(linesA, linesB) {
  const common = lcs(linesA, linesB);
  const result = [];
  let ai = 0, bi = 0, ci = 0;
  while (ci < common.length) {
    const [ca, cb] = common[ci];
    // 共通行の手前: del と add を出力
    while (ai < ca) { result.push({ type: 'del', text: linesA[ai++] }); }
    while (bi < cb) { result.push({ type: 'add', text: linesB[bi++] }); }
    // 共通行: eq
    result.push({ type: 'eq', text: linesA[ai++] }); bi++;
    ci++;
  }
  // 残り
  while (ai < linesA.length) { result.push({ type: 'del', text: linesA[ai++] }); }
  while (bi < linesB.length) { result.push({ type: 'add', text: linesB[bi++] }); }
  return result;
}
```

重要な不変条件:
- `eq_count + del_count === linesA.length`
- `eq_count + add_count === linesB.length`
- `del` と `eq` のハンクからAを再構築できる
- `add` と `eq` のハンクからBを再構築できる

## XSS対策

```js
function escHtml(s) {
  return String(s)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;');
}
```

テキスト内容をHTMLとしてレンダリングする前に必ずエスケープします。`&` を先に変換しないと、後続の変換で `&amp;` が `&amp;amp;` になる点に注意。

## 機能

- **Unified view** — 追加(+)/削除(−)/変更なし の1カラム表示
- **Split view** — 左(A)/右(B)の2カラム並列表示
- **統計情報** — +N added / −N removed / N unchanged をリアルタイム表示
- **サンプルテキスト** 初期ロード時に JavaScript 関数の差分例を表示

## テスト

Node.js `assert` モジュールで **120テスト** を実装、全パス確認済み。

主なテストカテゴリ:
- `escHtml` — 特殊文字エスケープ、型強制、XSSペイロード
- `lcs` — 空配列、一致なし、全一致、重複要素、インデックスの昇順確認
- `diff` — 追加/削除/変更なし、再構築検証、Unicode、空行、大文字小文字区別
- 統計不変条件 (`add+eq = B.length`, `del+eq = A.length`)
- 100行の同一テキストでのパフォーマンス確認

## 使い方

https://text-diff-viewer.pages.dev/ にアクセスして、Original(A)とModified(B)にテキストを貼り付け、Compareをクリックしてください。

---

DevNest.io では他にも開発者向けツールを公開しています。
