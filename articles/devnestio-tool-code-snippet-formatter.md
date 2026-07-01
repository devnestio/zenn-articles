---
title: "コードスニペットをシンタックスハイライトしてPNGで保存するツールを作った"
emoji: "🎨"
type: "tech"
topics: ["javascript", "devtools", "cloudflare", "frontend", "design"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に、**Code Snippet Formatter** を追加しました。

👉 https://devnestio.pages.dev/code-snippet-formatter/

コードを貼り付けてテーマ・言語・フォントサイズを選ぶと、シンタックスハイライトされたプレビューが即座に表示されます。PNG書き出し機能付き。

---

## 作った動機

ブログやSNSにコードを貼るとき、見栄えのよい画像が欲しいことがあります。でも専用アプリを使うのは面倒。ブラウザだけで完結するツールが欲しくて作りました。

---

## 機能

- **19言語対応**: JavaScript, TypeScript, Python, HTML, CSS, JSON, Bash, SQL, Rust, Go, Java, C++, PHP, Ruby, Swift, Kotlin, YAML, Markdown, PlainText
- **6テーマ**: One Dark / GitHub Dark / Dracula / Solarized / Nord / Light
- **カスタマイズ**:
  - フォントサイズ（11〜22px）
  - パディング（8〜60px）
  - ウィンドウフレーム（macOS風のトラフィックライトドット）のON/OFF
  - 行番号表示のON/OFF
  - 背景色（ダーク/ホワイト/グラデーション3種）
- **PNG書き出し**: SVGの `foreignObject` を使ってブラウザ完結で高解像度PNG出力
- **ハイライトHTMLコピー**: `<pre><code>` 付きハイライト済みHTMLをコピー

---

## 技術

- 単一HTMLファイル・ゼロ依存（シンタックスハイライトも自前の正規表現ベース実装）
- PNG書き出しはCanvas + SVG foreignObject
- Cloudflare Pagesでホスティング

---

## 関連ツール

👉 https://devnestio.pages.dev/
