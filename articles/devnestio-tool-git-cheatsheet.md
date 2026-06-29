---
title: "Gitコマンドチートシート＆コマンドビルダーを作った（60コマンド以上・検索対応）"
emoji: "🔀"
type: "tech"
topics: ["git", "javascript", "devtools", "cloudflare", "webdev"]
published: true
---

## 作ったもの

開発者向け無料ツール群「devnestio」に、**Git Cheatsheet & Command Builder** を追加しました。

👉 https://git-cheatsheet-dev.pages.dev

60以上のGitコマンドをカテゴリ別・検索でフィルタリングできるリファレンスです。パラメータが必要なコマンドは「Build」ボタンで値を入力し、そのままコピーできます。

---

## カテゴリ一覧

| カテゴリ | 代表コマンド |
|---------|------------|
| **Basics** | init, clone, status, add, commit, log, diff |
| **Staging** | add -p, reset HEAD, restore, clean |
| **Branch** | branch, checkout, switch, merge, rebase |
| **Remote** | fetch, pull, push, force-with-lease |
| **History** | log --author, log -S (pickaxe), blame, bisect |
| **Undo** | revert, reset --soft/mixed/hard, amend, cherry-pick |
| **Stash** | stash push -m, list, pop, apply, drop |
| **Tags** | tag -a, push --tags |
| **Advanced** | worktree, reflog, submodule, grep |

---

## コマンドビルダー

ブランチ名・コミットハッシュ・URL・メッセージなどのパラメータを持つコマンドには「Build」ボタンがあります。クリックするとパネルが開き、値を入力するとコマンドがリアルタイムで完成します。

例：`git push {remote} {branch}` の場合、remote と branch のフィールドが表示され、入力値（または未入力時のプレースホルダー）で置換されたコマンドがすぐにコピーできます。

---

## 主な機能

- コマンド名・構文・説明文をまたいだ**横断検索**
- カテゴリタブで絞り込み
- ヒット数リアルタイム表示
- 検索キーワードのハイライト
- コマンドの直接コピー or ビルダーでパラメータ設定してコピー
- 初回ロード後はオフラインで動作

---

## テスト

Node.js の `assert` で **80件** のテストを実装し全通過確認済みです。

- COMMANDSデータの整合性（name/cmd/desc/cat/paramsの存在確認）
- buildCommandのパラメータ置換ロジック
- escapeHTML / highlightの動作
- filterCommandsの検索・カテゴリフィルター

---

## 使ってみてください

👉 **[Git Cheatsheet & Command Builder — devnestio](https://git-cheatsheet-dev.pages.dev)**

全ツール一覧: https://devnestio.pages.dev
