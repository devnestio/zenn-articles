---
title: "ブラウザだけで YAML ↔ JSON 変換ツールを作った"
emoji: "🔄"
type: "tech"
topics: ["javascript", "yaml", "json", "webdev"]
published: false
---

YAML と JSON を相互変換するブラウザ専用ツールを作りました。依存ライブラリなし、サーバー通信なし。

**ツール:** https://devnestio.pages.dev/yaml-json-converter/

## 機能

- YAML → JSON / JSON → YAML 変換
- 入力形式の自動検出
- インデント: 2スペース / 4スペース / タブ
- キーのアルファベット順ソート
- パネルスワップでチェーン変換
- `.yaml` / `.json` ダウンロード

## 実装のポイント

### YAML パーサー（依存なし）

インデントスタックで再帰的にネストを処理します。

```js
function parseYamlLines(lines, startIdx, parentIndent) {
  const result = {};
  const arr = [];
  let isArr = null;
  let i = startIdx;

  while (i < lines.length) {
    const stripped = lines[i].replace(/#.*$/, '').trimEnd();
    if (stripped.trim() === '') { i++; continue; }
    const indent = stripped.length - stripped.trimStart().length;
    if (indent <= parentIndent) break;
    const line = stripped.trimStart();

    if (line.startsWith('- ')) {
      isArr = true;
      arr.push(parseScalar(line.slice(2).trim()));
      i++;
    } else if (line.includes(': ') || line.endsWith(':')) {
      isArr = false;
      // キーと値を分離
      const colonIdx = line.indexOf(': ');
      const key = colonIdx === -1 ? line.slice(0, -1) : line.slice(0, colonIdx);
      const val = colonIdx === -1 ? '' : line.slice(colonIdx + 2).trim();
      if (!val) {
        i++;
        const sub = parseYamlLines(lines, i, indent);
        result[key] = sub.value; i = sub.nextIdx;
      } else { result[key] = parseScalar(val); i++; }
    } else i++;
  }
  return { value: isArr ? arr : result, nextIdx: i };
}
```

### スカラー型の自動判定

```js
function parseScalar(v) {
  if (v === 'true' || v === 'yes') return true;
  if (v === 'false' || v === 'no') return false;
  if (v === 'null' || v === '~' || v === '') return null;
  if (/^-?\d+$/.test(v)) return parseInt(v, 10);
  if (/^-?\d*\.\d+$/.test(v)) return parseFloat(v);
  if (v.startsWith('"') && v.endsWith('"')) return v.slice(1, -1);
  return v;
}
```

### YAML シリアライザー

数字・キーワードに見える文字列はクォートして曖昧さを排除します。

```js
function yamlStr(s) {
  if (s === '') return '""';
  if (['true','false','null','yes','no','~'].includes(s)) return `"${s}"`;
  if (/^\d+$/.test(s) || /^\d*\.\d+$/.test(s)) return `"${s}"`;
  return s;
}
```

### 入力形式の自動検出

```js
function detectFormat(text) {
  const t = text.trim();
  if (t.startsWith('{') || t.startsWith('[')) return 'json';
  try { JSON.parse(t); return 'json'; } catch(_) {}
  return 'yaml';
}
```

## テスト

97 テストすべてパス。YAML パース・シリアライズ・ラウンドトリップを網羅しています。

[DevNestio](https://devnestio.pages.dev) — ブラウザだけで動く開発者ツール集
