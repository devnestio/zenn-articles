---
title: "ASCIIアートジェネレーターをVanilla JSで作った話（11フォント・ゼロ依存、144テスト）"
emoji: "🎨"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "javascript"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に **ASCII Art Generator** を追加しました。テキストを入力するとBanner・Block・Doom・Shadow・Double・Thin・Mini・Stars・Pipes・Dots・Boxの11種フォントでASCIIアートに変換できます。外部ライブラリゼロ、すべてブラウザ内で完結します。

👉 https://ascii-art-gen.pages.dev

## なぜ作ったか

CLIツールのヘルプ出力やREADMEのバナー、Slackのお知らせなどでASCIIアートを使いたい場面があります。`figlet`コマンドをインストールするまでもない小さな用途に、ブラウザで動く軽量ツールがほしかった。

## 主な機能

- **11種のフォント** — Banner（#文字）、Block（█文字）、Doom、Shadow、Double（罫線文字）、Thin、Mini（3行コンパクト）、Stars（*）、Pipes（|）、Dots（·）、Box（█）
- **カラー5色** — Green/Blue/Purple/Orange/White
- **全フォントプレビューグリッド** — 入力テキストを全フォントで一覧表示
- **ワンクリックコピー**
- **.txtダウンロード**

## 技術的なポイント：フォントを外部依存なしで実装

figletなどのライブラリは膨大なフォントファイルを持ちますが、今回は11フォントをすべてJavaScriptオブジェクトとして埋め込みました。各文字は行の配列として定義されています。

```js
// Banner font — A文字の定義
BANNER_CHARS['A'] = [
  ' #  # ',
  '#    #',
  '#    #',
  '######',
  '#    #',
  '#    #',
  '#    #',
  '      ',
];

function renderBanner(text) {
  const rows = 8;
  const lines = Array(rows).fill('');
  for (const ch of text.toUpperCase()) {
    const glyph = BANNER_CHARS[ch] || BANNER_CHARS[' '];
    const w = Math.max(...glyph.map(r => r.length));
    for (let r = 0; r < rows; r++) {
      lines[r] += (glyph[r] || '').padEnd(w, ' ') + ' ';
    }
  }
  return lines.join('\n');
}
```

Stars・Pipes・Dots・Boxフォントは、Bannerの出力の`#`を別の文字に置換するだけで実現しています。これにより4フォントを追加コストなしで作れました。

```js
const renderStars = (text) => renderBanner(text).replace(/#/g, '*');
const renderPipes = (text) => renderBanner(text).replace(/#/g, '|');
```

## テスト結果

**144/144テスト PASS**。各フォントの行数・文字出力・大文字変換・未知文字フォールバック・複数フォント間の一貫性などを確認しています。

## 実装はAIエージェントに任せた

Claude Codeに「ゼロ依存でBanner・Block・Doom・Shadowフォントを実装すること」を発注書に含めました。フォント定義をJavaScriptオブジェクトとして埋め込む発想はエージェントが提案したもので、ビルドステップなしで配布できる構成になっています。

## 使ってみてください

👉 **[devnestio - Developer Tools Hub](https://devnestio.pages.dev)**

READMEのバナーやCLIツールのヘルプ出力にどうぞ。
