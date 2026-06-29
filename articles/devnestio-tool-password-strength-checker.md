---
title: "パスワード強度チェッカーをVanilla JSで作った — エントロピー計算とクラック時間推定まで"
emoji: "🔐"
type: tech
topics: ["javascript", "webdev", "security", "tools"]
published: false
---

## これは何？

[DevNest.io](https://devnestio.pages.dev/) のツールとして作ったパスワード強度チェッカーです。

**URL**: https://password-strength-checker.pages.dev/

入力したパスワードのエントロピー（ビット数）を計算し、オンライン/オフライン攻撃それぞれのクラック時間を推定します。外部ライブラリなし、ブラウザのみで動作します。

## 実装した機能

- **エントロピー計算** — 使用文字種（小文字・大文字・数字・記号・非ASCII）から文字集合サイズを推定し `length × log2(charset)` で計算
- **Leet変換検出** — `0→o`, `1→i`, `3→e`, `4→a`, `5→s` などの置換をした上で辞書単語とマッチ
- **パターン検出** — キーボード配列（`qwerty`, `asdf`, `1234`）、日付形式、繰り返し文字
- **5種のクラック時間推定**:
  1. オンライン（スロットリングあり）: 100回/秒
  2. オンライン（スロットリングなし）: 10,000回/秒
  3. オフライン bcrypt: 10,000ハッシュ/秒
  4. オフライン MD5: 100億ハッシュ/秒
  5. GPUクラスタ SHA-256: 10兆ハッシュ/秒
- **強度スコア** — Very Weak / Weak / Fair / Good / Strong / Very Strong の6段階
- **改善提案** — 文字種追加、長さ追加、パターン回避の具体的アドバイス

## 技術的なポイント

### エントロピー計算

```js
function charsetSize(s) {
  let size = 0;
  if (/[a-z]/.test(s)) size += 26;
  if (/[A-Z]/.test(s)) size += 26;
  if (/[0-9]/.test(s)) size += 10;
  if (/[ !-\/:-@\[-`{-~]/.test(s)) size += 32;
  if (/[^\x00-\x7F]/.test(s)) size += 64;
  return size || 1;
}

function entropy(s) {
  return s.length * Math.log2(charsetSize(s));
}
```

実際の文字集合サイズ（使われた文字の種類数）ではなく、「使用された文字種のカテゴリが含む最大文字数」を使います。これはクラッカーが最悪ケースで試す必要がある組み合わせ数に対応しています。

### Leet変換

```js
function deleet(s) {
  const LEET = {'0':'o','1':'i','3':'e','4':'a','5':'s','@':'a','$':'s','!':'i'};
  return s.toLowerCase().replace(/[0134@$!]/g, c => LEET[c] || c);
}
```

`p4ssw0rd` → `password` に変換してから辞書チェックします。

### 日付パターン検出

```js
function hasDatePattern(s) {
  return /(19|20)\d{2}|\b\d{1,2}[\/\-\.]\d{1,2}([\/\-\.]\d{2,4})?\b/.test(s);
}
```

最初のパターンから `\b` を外すことで `pass2000` のような埋め込み年号も検出します。

### クラック時間フォーマット

```js
function formatTime(seconds) {
  if (seconds < 60) return `${Math.round(seconds)} seconds`;
  if (seconds < 3600) return `${Math.round(seconds/60)} minutes`;
  if (seconds < 86400) return `${Math.round(seconds/3600)} hours`;
  if (seconds < 31536000) return `${Math.round(seconds/86400)} days`;
  if (seconds < 3.154e10) return `${Math.round(seconds/31536000)} years`;
  return `${(seconds/3.154e10).toExponential(1)} centuries`;
}
```

## テスト

Node.js の `assert` モジュールで **122テスト** を実装、全パス確認済み。

主なテストケース:
- `entropy('a')` → 約4.7ビット（26文字集合）
- `entropy('aA1!')` → 約25.8ビット（94文字集合）
- `deleet('p4ssw0rd')` → `'password'`
- `hasDatePattern('pass2000')` → `true`
- `isKeyboardPattern('qwer')` → `true`
- 長いパスワードほどクラック時間が指数的に長くなる

## 使い方

https://password-strength-checker.pages.dev/ にアクセスして、パスワードを入力するだけです。入力はすべてブラウザ内で処理され、サーバーには送信されません。

---

DevNest.io では他にも開発者向けツールを公開しています。興味があればトップページからどうぞ。
