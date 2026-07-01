---
title: "SVGパスコマンドをブラウザで可視化するツールを作った"
emoji: "✏️"
type: "tech"
topics: ["javascript","svg","webdev","browser","visualization"]
published: false
---

## はじめに

SVGの `d` 属性は強力ですが、コマンドの羅列は読みにくいです。各コマンドを解析して説明・プレビューするブラウザツールを実装しました。

**ツール:** https://devnestio.pages.dev/svg-path-editor/

## 対応コマンド

| コマンド | 名前 | パラメータ数 |
|---|---|---|
| M/m | MoveTo | 2 (x, y) |
| L/l | LineTo | 2 (x, y) |
| H/h | HorizontalLineTo | 1 (x) |
| V/v | VerticalLineTo | 1 (y) |
| C/c | CubicBezier | 6 (x1,y1,x2,y2,x,y) |
| S/s | SmoothCubic | 4 (x2,y2,x,y) |
| Q/q | QuadraticBezier | 4 (x1,y1,x,y) |
| T/t | SmoothQuadratic | 2 (x, y) |
| A/a | Arc | 7 (rx,ry,angle,large,sweep,x,y) |
| Z/z | ClosePath | 0 |

## パーサー実装

```js
function parsePath(d) {
  // コマンド文字と数値（科学記数法含む）をトークン化
  const tokens = d.trim().match(
    /[MmLlHhVvCcSsQqTtAaZz]|[+-]?(?:\d+\.?\d*|\.\d+)(?:[eE][+-]?\d+)?/g
  ) || [];

  const cmds = [];
  let i = 0;
  while (i < tokens.length) {
    const letter = tokens[i];
    if (!/[MmLlHhVvCcSsQqTtAaZz]/.test(letter)) { i++; continue; }
    const info = CMD_INFO[letter];
    i++;
    if (info.params.length === 0) {
      cmds.push({ cmd: letter, args: [] });
    } else {
      const n = info.params.length;
      while (i < tokens.length && !/[MmLlHhVvCcSsQqTtAaZz]/.test(tokens[i])) {
        const args = [];
        for (let j = 0; j < n; j++, i++) args.push(parseFloat(tokens[i]));
        if (args.length === n) cmds.push({ cmd: letter, args });
      }
    }
  }
  return cmds;
}
```

## バウンディングボックス計算

絶対座標・相対座標の両方を処理して近似バウンディングボックスを求めます。

```js
function getBounds(cmds) {
  let xs = [], ys = [], cx = 0, cy = 0;
  for (const { cmd, args } of cmds) {
    const abs = cmd === cmd.toUpperCase();
    switch (cmd.toUpperCase()) {
      case 'M': case 'L':
        for (let i = 0; i < args.length; i += 2) {
          const x = abs ? args[i] : cx + args[i];
          const y = abs ? args[i+1] : cy + args[i+1];
          xs.push(x); ys.push(y); cx = x; cy = y;
        }
        break;
      // H, V, C, Q, S, A も同様に処理
    }
  }
  return { minX: Math.min(...xs), maxX: Math.max(...xs),
           minY: Math.min(...ys), maxY: Math.max(...ys) };
}
```

## 機能まとめ

- コマンドをクリックするとプレビュー上で端点をハイライト
- 制御点（ベジェ曲線のハンドル）の表示/非表示
- フィル・ストロークの色をカスタマイズ
- サンプルパス: 矢印、星、ハート、ベジェ曲線、楕円弧
- 400msのデバウンスで入力に合わせて自動解析
