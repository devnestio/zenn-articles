---
title: "robots.txtをビジュアルで生成する — AIクローラーブロックも簡単（84テスト）"
emoji: "🤖"
type: "tech"
topics: ["seo", "webdev", "javascript", "devtools"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に、**Robots.txt Generator** を追加しました。

👉 https://devnestio.pages.dev/robots-txt-gen/

ボットを選んでパスを追加するだけでrobots.txtが生成できます。

---

## 主な機能

- **ボットチップ** — Googlebot, Bingbot, GPTBot, CCBot, anthropic-ai, Applebotをトグル
- **カスタムUser-Agent** — 任意のクローラーを追加可能
- **パスルール** — Allow/Disallowをワンクリック追加（/admin/, /api/, /wp-admin/等）
- **テンプレート** — 全許可, 全拒否, AIクローラーブロック, SEOフレンドリー, WordPress標準
- **Crawl-delay & Sitemap** — オプション設定
- **ダウンロード** — robots.txtとして直接保存
- **構文ハイライト** — コメント・キー・値・エージェント名を色分け

---

## AIクローラーのUser-Agent一覧（2024〜2025年）

| ボット | 企業 | 用途 |
|--------|------|------|
| `GPTBot` | OpenAI | GPT学習データ収集 |
| `CCBot` | Common Crawl | オープンデータセット |
| `anthropic-ai` | Anthropic | Claude学習データ |
| `Applebot` | Apple | Siri/ML用データ |

---

## テスト

**84 / 84 PASS ✅**

全5テンプレート、ボットトグル、ルール追加、ダウンロード、HTMLエスケープをテスト。

👉 https://devnestio.pages.dev
