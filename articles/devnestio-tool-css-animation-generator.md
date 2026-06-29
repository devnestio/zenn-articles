---
title: "CSSアニメーションジェネレーターをVanilla JSで作った話（ビジュアルキーフレームビルダー・110テスト）"
emoji: "✨"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "javascript"]
published: true
---

## 作ったもの

開発者向け無料ツール群「devnestio」に **CSS Animation Generator** を追加しました。@keyframesアニメーションをビジュアルで組み立てて、すぐ使えるCSSコードをコピーできます。外部ライブラリゼロ、すべてブラウザ内で完結します。

👉 https://css-animation-generator.pages.dev

## なぜ作ったか

CSSアニメーションは`animation-timing-function`やキーフレームのパーセンテージを手書きするのが面倒です。ちょっとしたfade-inやslide-inを書きたいときに、毎回MDNを開いてコピペするより、ビジュアルで調整できるツールがほしかった。

## 主な機能

- **イージング選択** — ease / ease-in / ease-out / ease-in-out / linear / cubic-bezier
- **duration・delay・iteration設定** — スライダーで直感的に調整
- **opacity / translateX / translateY のキーフレーム設定**
- **リアルタイムプレビュー** — 設定変更がアニメーションに即反映
- **@keyframesコードのコピー** — そのままCSSに貼り付け可能
- **プリセット12種** — fade-in、slide-up、bounce、shake、pulsなど

## 技術的なポイント：CSSアニメーションをJSで動的生成

プレビューにはStyleSheetのinsertRule/deleteRuleを使ってJSから動的に`@keyframes`を注入しています。

```js
function applyAnimation(config) {
  // 既存のkeyframesを削除
  for (let i = sheet.cssRules.length - 1; i >= 0; i--) {
    if (sheet.cssRules[i].name === 'preview-anim') {
      sheet.deleteRule(i);
    }
  }
  // 新しいkeyframesを生成・挿入
  const kf = buildKeyframes(config);
  sheet.insertRule(kf, sheet.cssRules.length);
  
  // アニメーションをリセットして再適用
  el.style.animation = 'none';
  el.offsetHeight; // reflow
  el.style.animation = `preview-anim ${config.duration}s ${config.easing} ${config.delay}s ${config.iteration}`;
}
```

`el.offsetHeight`でリフローを強制することで、同じアニメーション名でも再スタートが確実に動きます。

## テスト結果

**110/110テスト PASS**。各プリセットのCSSコード生成、イージング関数の文字列変換、cubic-bezierのバリデーション、キーフレームのパーセンテージ計算などを確認しています。

## 実装はAIエージェントに任せた

Claude Codeに「外部CSSライブラリなしでリアルタイムプレビューを実装すること」を指示しました。StyleSheetのinsertRuleを使う方針はエージェントが提案したもので、フレームワーク不要の軽量実装になっています。

## 使ってみてください

👉 **[devnestio - Developer Tools Hub](https://devnestio.pages.dev)**

ランディングページのfade-inやボタンのhoverアニメーションなど、すぐ使えるCSSコードを生成できます。
