---
title: "VS Code・Chrome・Macのキーボードショートカット一覧ツールを作った"
emoji: "⌨"
type: "idea"
topics: ["webdev", "tools", "javascript", "vscode"]
published: false
---

# Keyboard Shortcut Reference

VS Code・Chrome・Macのショートカットを150件以上まとめた検索・フィルタリング対応のブラウザツールを作りました。

https://devnestio.pages.dev/keyboard-shortcut-ref/

## 機能

- **即時検索** — アクション名またはキー名で絞り込み
- **アプリフィルター** — VS Code・Chrome・Mac を切り替え
- **カテゴリ別表示** — Editing・Navigation・DevTools・System など
- **ワンクリックコピー** — キーコンボをクリップボードへ

## 収録ショートカット

### VS Code
Editing（マルチカーソル、フォーマット、コメント）、Navigation（定義ジャンプ、参照）、Search & Replace、View、Refactor、Debug

### Chrome
Tabs（開く・閉じる・復元）、Navigation（戻る・リロード）、DevTools、Page操作

### Mac
General（コピー・ペースト等）、Finder、スクリーンショット、システム（Spotlight・強制終了）、テキスト操作

## 技術

Vanilla JS・CSS変数によるダークテーマ・`kbd`要素でキーを視覚的に表現。フレームワーク不使用で即座に読み込まれます。

---

[DevNestio](https://devnestio.pages.dev/) — ブラウザで使える開発者向けツール集
