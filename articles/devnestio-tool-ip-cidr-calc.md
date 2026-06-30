---
title: "IP/CIDRサブネット計算ツールをリリースしました — ip-cidr-calc"
emoji: "🌐"
type: "idea"
topics: ["networking", "javascript", "webdev", "tools"]
published: false
---

## IP/CIDR Calculator

IPv4アドレスとCIDR表記からサブネット情報を即座に計算する無料ツールをリリースしました。

🔗 **[ip-cidr-calc.pages.dev](https://ip-cidr-calc.pages.dev)**

## 計算できる内容

`192.168.1.0/24` のように入力するだけで以下をすべて表示：

- ネットワークアドレス
- ブロードキャストアドレス
- サブネットマスク / ワイルドカードマスク
- 最初/最後の使用可能ホスト
- 総IP数 / 使用可能ホスト数
- IPクラス（A/B/C/D/E）
- 2進数表記（ドット区切り）

## 対応プレフィックス

- `/0`〜`/30`: 標準サブネット
- `/31`: RFC 3021 ポイントツーポイント
- `/32`: 単一ホスト

## 実装

JavaScriptのビット演算でサブネット計算を実装：

```js
const mask = (0xffffffff << (32 - prefix)) >>> 0;
const network = (ip & mask) >>> 0;
const broadcast = (network | (~mask >>> 0)) >>> 0;
```

## devnestio について

🔗 [devnestio.pages.dev](https://devnestio.pages.dev)
