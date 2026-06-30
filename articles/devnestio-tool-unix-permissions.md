---
title: "Unix パーミッション計算ツール — chmod 数値↔シンボルを即変換"
emoji: "🔐"
type: "idea"
topics: ["linux", "javascript", "webdev", "tools"]
published: false
---

## Unix Permissions Calculator

chmod の数値表記とシンボル表記を即座に相互変換できる無料ツールをリリースしました。

🔗 **[unix-perm-calc.pages.dev](https://unix-perm-calc.pages.dev)**

## 主な機能

- **数値 → シンボル**: `755` と入力 → `rwxr-xr-x`
- **シンボル → 数値**: `rw-r--r--` と入力 → `644`
- **ビットトグル**: owner/group/other の r/w/x ボタンをクリックして設定
- chmod コマンド・`ls -l`形式・2進数表記を表示
- よく使うプリセット（755, 644, 600 など）ワンクリック

## パーミッションビットの仕組み

```
r (read)    = 4 = 100
w (write)   = 2 = 010
x (execute) = 1 = 001
```

`755` = `111 101 101` → `rwxr-xr-x`

## よく使うパーミッション

| 数値 | シンボル      | 用途 |
|------|-------------|------|
| 755  | rwxr-xr-x   | 実行ファイル・ディレクトリ |
| 644  | rw-r--r--   | 通常ファイル |
| 600  | rw-------   | SSH鍵・秘密ファイル |
| 700  | rwx------   | プライベートディレクトリ |

## devnestio について

🔗 [devnestio.pages.dev](https://devnestio.pages.dev)
