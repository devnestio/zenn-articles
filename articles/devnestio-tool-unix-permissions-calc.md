---
title: "Unixパーミッション計算ツールをブラウザで作った（chmod 755 ↔ rwxr-xr-x）"
emoji: "🔐"
type: "idea"
topics: ["linux", "tools", "devops", "javascript"]
published: false
---

# Unix Permissions Calculator

チェックボックスでUNIXファイルパーミッションを視覚的に設定し、8進数と記号表記をリアルタイム変換するブラウザツールを作りました。

https://devnestio.pages.dev/unix-permissions-calc/

## 機能

- **視覚的チェックボックス** — Owner / Group / Other × Read / Write / Execute
- **リアルタイム変換** — 8進数 (`755`) ↔ 記号表記 (`rwxr-xr-x`)
- **直接入力** — 8進数や記号を入力するとチェックボックスが自動更新
- **12のプリセット** — 644、755、700、600、777、sticky dir、setUIDなど
- **コピーボタン** — 8進数・記号表記をワンクリックでコピー

## パーミッションビットの対応表

| 8進数 | 2進数 | rwx | 意味 |
|------|------|-----|------|
| 7 | 111 | rwx | 読み取り・書き込み・実行 |
| 6 | 110 | rw- | 読み取り・書き込み |
| 5 | 101 | r-x | 読み取り・実行 |
| 4 | 100 | r-- | 読み取りのみ |
| 0 | 000 | --- | 権限なし |

## よく使うプリセット

```
644  rw-r--r--   Webファイル（所有者rw、グループ・その他r）
600  rw-------   秘密ファイル（所有者rwのみ）
755  rwxr-xr-x   実行ファイル・公開ディレクトリ
700  rwx------   プライベートディレクトリ
777  rwxrwxrwx   全員フルアクセス（要注意）
```

## 実装

各8進数の桁は3ビット（r=4, w=2, x=1）に対応：

```js
function bitsToOctal(b) {
  const owner = (b[0]<<2)|(b[1]<<1)|b[2];
  const group = (b[3]<<2)|(b[4]<<1)|b[5];
  const other = (b[6]<<2)|(b[7]<<1)|b[8];
  return '' + owner + group + other;
}
```

---

[DevNestio](https://devnestio.pages.dev/) — ブラウザで使える開発者向けツール集
