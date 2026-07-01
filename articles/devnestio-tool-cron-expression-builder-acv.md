---
title: "Cronの書き方を視覚化するツールを作った（次回実行時刻も表示）"
emoji: "⏰"
type: "tech"
topics: ["cron", "devtools", "javascript", "cloudflare", "devops"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に、**Cron Expression Builder** を追加しました。

👉 https://devnestio.pages.dev/cron-expression-builder/

Cron式を視覚的に組み立て、スケジュールを英語で説明し、次の10回の実行時刻を表示できます。ブラウザ完結・外部API不要です。

---

## 作った動機

「`0 9 * * 1-5`」のような式を書くとき、毎回「これ合ってるか？」と不安になることがあります。わざわざcrontabで試すほどでもないけど確認はしたい。そんな場面のために作りました。

---

## 機能

- **ビジュアルビルダー**: 分・時・日・月・曜日を個別に選択してCron式を組み立て
- **式の説明**: `Every weekday at 9:00 AM` のように平易な英語で解説
- **次回実行時刻プレビュー**: 現在時刻から次の10回分を表示
- **標準5フィールド＆拡張6フィールド対応**
- **よく使うプリセット**: 毎時・毎日・毎週・毎月・平日のみなど

---

## 技術

- 単一HTMLファイル（CDNなし、ゼロ依存）
- Cloudflare Pagesでホスティング
- Cron解析は自前のJSロジック

---

## 関連ツール

devnestioでは他にも多くの開発者ツールを公開しています。

👉 https://devnestio.pages.dev/
