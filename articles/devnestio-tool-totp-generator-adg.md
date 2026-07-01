---
title: "ブラウザだけで動くTOTPジェネレーター（RFC 6238準拠）"
emoji: "🔐"
type: "tech"
topics: ["javascript","security","webapi","2fa","crypto"]
published: false
---

## はじめに

ブラウザのみで動作するTOTP（Time-based One-Time Password）ジェネレーターを作りました。サーバー不要、依存ライブラリなしで、Web Crypto APIだけを使っています。

**ツール:** https://devnestio.pages.dev/totp-generator/

## TOTPとは

RFC 6238で定義されたTOTP（Time-based OTP）は、現在時刻をカウンターとして使うHOTPです。

```
TOTP = HOTP(secret, T)
T = floor(epoch / period)
```

## 実装

### Base32デコード

TOTP秘密鍵はBase32でエンコードされています。

```js
const BASE32_CHARS = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ234567';

function base32Decode(s) {
  s = s.toUpperCase().replace(/=+$/, '').replace(/[^A-Z2-7]/g, '');
  let bits = 0, value = 0;
  const output = [];
  for (const c of s) {
    value = (value << 5) | BASE32_CHARS.indexOf(c);
    bits += 5;
    if (bits >= 8) {
      output.push((value >>> (bits - 8)) & 0xff);
      bits -= 8;
    }
  }
  return new Uint8Array(output);
}
```

### TOTP生成（Web Crypto API）

```js
async function generateTOTP(secret, algorithm, digits, period) {
  const keyBytes = base32Decode(secret);
  const counter = Math.floor(Date.now() / 1000 / period);

  // 8バイトのビッグエンディアンカウンター
  const counterBuf = new Uint8Array(8);
  let tmp = counter;
  for (let i = 7; i >= 0; i--) {
    counterBuf[i] = tmp & 0xff;
    tmp = Math.floor(tmp / 256);
  }

  const key = await crypto.subtle.importKey(
    'raw', keyBytes,
    { name: 'HMAC', hash: algorithm },
    false, ['sign']
  );
  const sig = await crypto.subtle.sign('HMAC', key, counterBuf);
  const hash = new Uint8Array(sig);

  // Dynamic truncation
  const offset = hash[hash.length - 1] & 0x0f;
  const code = (
    ((hash[offset] & 0x7f) << 24) |
    (hash[offset + 1] << 16) |
    (hash[offset + 2] << 8) |
    hash[offset + 3]
  ) % Math.pow(10, digits);

  return String(code).padStart(digits, '0');
}
```

## 機能

- **アルゴリズム:** SHA-1（最も一般的）、SHA-256、SHA-512
- **桁数:** 6桁 / 8桁
- **更新周期:** 30秒 / 60秒
- **残り時間:** プログレスバー + 5秒以下で赤くパルス
- **秘密鍵生成:** `crypto.getRandomValues` でランダム生成

すべての処理はブラウザ内で完結し、秘密鍵はサーバーに送信されません。
