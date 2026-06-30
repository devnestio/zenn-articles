---
title: "ブラウザで動くテキスト差分ツール — Text Diff Tool をリリースしました"
emoji: "🔍"
type: "idea"
topics: ["javascript", "webdev", "tools", "diff"]
published: false
---

## Text Diff Tool

2つのテキストをブラウザ上で比較できる無料ツールをリリースしました。

🔗 **[text-diff-tool-1ia.pages.dev](https://text-diff-tool-1ia.pages.dev)**

## 主な機能

- **3つの差分モード**: 行単位 / 単語単位 / 文字単位
- **2つの表示モード**: サイドバイサイド（分割表示）/ ユニファイド表示
- 変更箇所のインラインハイライト
- 行番号表示と追加/削除カウント
- 入力しながらリアルタイムに差分更新
- テキストの左右入れ替え・クリア機能

## 技術

LCS（最長共通部分列）アルゴリズムを純粋なVanilla JSで実装しています。

```js
function lcs(a, b) {
  const dp = Array.from({length: a.length+1}, () => new Array(b.length+1).fill(0));
  for (let i = 1; i <= a.length; i++)
    for (let j = 1; j <= b.length; j++)
      dp[i][j] = a[i-1] === b[j-1] ? dp[i-1][j-1]+1 : Math.max(dp[i-1][j], dp[i][j-1]);
  // バックトラックで操作列を構築...
}
```

トークン化の方法を変えるだけで、行・単語・文字の3モードすべてに対応。

## devnestio について

開発者向け無料マイクロツール集です。

🔗 [devnestio.pages.dev](https://devnestio.pages.dev)
