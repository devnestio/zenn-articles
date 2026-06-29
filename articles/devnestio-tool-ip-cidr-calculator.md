---
title: "IPアドレス & CIDRサブネット計算ツールをVanilla JSで作った"
emoji: "🌐"
type: tech
topics: ["javascript", "webdev", "network", "tools"]
published: false
---

## これは何？

[DevNest.io](https://devnestio.pages.dev/) のツールとして作ったIPアドレス & CIDRサブネット計算ツールです。

**URL**: https://ip-cidr-calculator.pages.dev/

`192.168.1.100/24` のようなCIDR表記を入力すると、ネットワークアドレス・ブロードキャストアドレス・ホスト範囲・サブネットマスクなどを即座に計算します。

## 実装

### IPv4 を32ビット整数として扱う

IPv4の演算はすべて32ビット整数のビット演算で行います。

```js
function ipToInt(octets) {
  return ((octets[0] << 24) | (octets[1] << 16) | (octets[2] << 8) | octets[3]) >>> 0;
}

function intToIp(n) {
  return [
    (n >>> 24) & 0xff,
    (n >>> 16) & 0xff,
    (n >>> 8) & 0xff,
    n & 0xff,
  ].join('.');
}
```

`>>> 0` は符号なし右シフト — JavaScriptのビット演算が符号付き32ビット整数で行われるため、`255.x.x.x` などで負の値になるのを防ぎます。

### サブネットマスク生成

```js
function cidrToMask(prefix) {
  if (prefix < 0 || prefix > 32) return null;
  return prefix === 0 ? 0 : (0xffffffff << (32 - prefix)) >>> 0;
}
```

`/24` → `0xffffff00` = `255.255.255.0`。`prefix === 0` を特別扱いするのは、`0xffffffff << 32` が JS では `0` ではなく `0xffffffff` になるためです（シフト量は mod 32）。

### サブネット計算

```js
function calcSubnet(octets, prefix) {
  const ipInt     = ipToInt(octets);
  const mask      = cidrToMask(prefix);
  const network   = (ipInt & mask) >>> 0;
  const broadcast = (network | (~mask >>> 0)) >>> 0;
  const firstHost = prefix < 31 ? network + 1 : network;
  const lastHost  = prefix < 31 ? broadcast - 1 : broadcast;
  const totalHosts = prefix >= 31 ? (prefix === 32 ? 1 : 2) : Math.pow(2, 32 - prefix) - 2;
  // ...
}
```

- **ネットワークアドレス**: `IP & mask`
- **ブロードキャスト**: `network | ~mask`
- **/31・/32** は特別扱い（ホスト数が 2・1 のみ）

### IPアドレス種別分類

```js
function ipType(octets) {
  const [a, b] = octets;
  if (a === 10) return 'Private (Class A)';
  if (a === 172 && b >= 16 && b <= 31) return 'Private (Class B)';
  if (a === 192 && b === 168) return 'Private (Class C)';
  if (a === 127) return 'Loopback';
  if (a === 169 && b === 254) return 'Link-local (APIPA)';
  if (a >= 224 && a <= 239) return 'Multicast';
  // ...
  return 'Public';
}
```

RFC 1918 のプライベートアドレス、RFC 5735 のループバック・リンクローカル、RFC 5771 のマルチキャストを識別します。

## 機能

- **CIDR 入力** — `192.168.1.0/24` 形式 またはプレフィックスなし（`/32` として扱う）
- **9つのサブネット情報** — IP・ネットワーク・ブロードキャスト・マスク・プレフィックス・最初のホスト・最後のホスト・使用可能ホスト数・IPタイプ
- **バイナリ表示** — IP・マスク・ネットワーク・ブロードキャストのオクテット単位の2進数表示
- **ワイルドカードマスク** — `~subnetMask`（ACL設定などで使用）
- **クリックでコピー** — 各情報カードをクリックでクリップボードにコピー
- **クイック例** 8パターン

## テスト

Node.js `assert` モジュールで **122テスト** を実装、全パス確認済み。

テストカテゴリ:
- `parseIPv4` — 有効/無効、トリム、境界値
- `ipToInt` / `intToIp` — ラウンドトリップ、符号なし整数
- `cidrToMask` — 全プレフィックス（0〜32）、エラー
- `maskToPrefix` — `cidrToMask` との往復
- `parseCIDR` — CIDR文字列、エラーケース
- `calcSubnet` — ネットワーク・ブロードキャスト・ホスト数、/0〜/32
- `ipType` — 全アドレス種別

## 使い方

https://ip-cidr-calculator.pages.dev/ にアクセスして、IPアドレスまたはCIDR表記を入力してCalculateをクリックしてください。

---

DevNest.io では他にも開発者向けツールを公開しています。
