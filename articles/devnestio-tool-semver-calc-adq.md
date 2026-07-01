---
title: "Vanilla JSでsemverのバンプ・比較・レンジチェックを実装"
emoji: "📦"
type: "tech"
topics: ["javascript","semver","webdev","browser","devops"]
published: false
---

## はじめに

セマンティックバージョニング（semver）のバンプ、比較、レンジチェックをブラウザのみで実装しました。

**ツール:** https://devnestio.pages.dev/semver-calc/

## semverのパース

公式の正規表現でパースします。

```js
const SEMVER_RE = /^(0|[1-9]\d*)\.(0|[1-9]\d*)\.(0|[1-9]\d*)(?:-((?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*)(?:\.(?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*))*))?(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?$/;

function parseSemver(v) {
  v = v.trim().replace(/^v/, '');  // 先頭の 'v' を除去
  const m = SEMVER_RE.exec(v);
  if (!m) return null;
  return { major: +m[1], minor: +m[2], patch: +m[3], prerelease: m[4]||null, build: m[5]||null };
}
```

## プレリリースの比較

semver仕様に従い、プレリリースは数値識別子と文字列識別子で異なる比較を行います。

```js
function comparePre(a, b) {
  if (!a && !b) return 0;
  if (!a) return 1;   // プレリリースなし > プレリリースあり
  if (!b) return -1;
  const aParts = a.split('.'), bParts = b.split('.');
  for (let i = 0; i < Math.max(aParts.length, bParts.length); i++) {
    const ai = aParts[i], bi = bParts[i];
    if (ai === undefined) return -1;
    if (bi === undefined) return 1;
    const aNum = /^\d+$/.test(ai), bNum = /^\d+$/.test(bi);
    if (aNum && bNum) {
      const diff = parseInt(ai) - parseInt(bi);
      if (diff) return diff;
    } else if (aNum) return -1;
    else if (bNum) return 1;
    else { if (ai < bi) return -1; if (ai > bi) return 1; }
  }
  return 0;
}
```

## レンジの ^ と ~ 展開

```js
if (tok.startsWith('^')) {
  const p = parseSemver(tok.slice(1));
  if (p.major !== 0) return [`>=${fmtVersion(p)}`, `<${p.major+1}.0.0`];
  if (p.minor !== 0) return [`>=${fmtVersion(p)}`, `<0.${p.minor+1}.0`];
  return [`>=${fmtVersion(p)}`, `<0.0.${p.patch+1}`];
}
if (tok.startsWith('~')) {
  const p = parseSemver(tok.slice(1));
  return [`>=${fmtVersion(p)}`, `<${p.major}.${p.minor+1}.0`];
}
```

## バンプの種類

| タイプ | 例 (1.2.3) | 説明 |
|---|---|---|
| major | 2.0.0 | 破壊的変更 |
| minor | 1.3.0 | 後方互換な新機能 |
| patch | 1.2.4 | バグ修正 |
| premajor | 2.0.0-alpha.0 | 次期メジャーのプレリリース |
| preminor | 1.3.0-alpha.0 | 次期マイナーのプレリリース |
| prepatch | 1.2.4-alpha.0 | 次期パッチのプレリリース |
| prerelease | 1.2.3-0 (→0が+1) | プレリリース番号のインクリメント |

バンプカードをクリックするとそのバージョンがメインフィールドに反映され、直前10件の履歴に保存されます。
