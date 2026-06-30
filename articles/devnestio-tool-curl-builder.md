---
title: "cURL Command Builder を作った — フォームで入力してcurlコマンド生成、全認証方式対応"
emoji: "🌐"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "curl"]
published: false
---

## どんなツール？

curlコマンドは強力だが、複雑なリクエスト（認証あり、ヘッダーあり、ファイルアップロードあり）を書くとき、正確なフラグを思い出すのが面倒。

**cURL Command Builder** は、ビジュアルフォームに入力するだけで正確なcurlコマンドを生成するツール。

👉 https://curl-builder-abc.pages.dev

## 主な機能

### HTTPメソッド
GET、POST、PUT、PATCH、DELETE、HEAD、OPTIONS、カスタムメソッド（PROPFINDなど）

### ボディタイプ
| タイプ | 生成されるフラグ |
|--------|-----------------|
| JSON | `-H 'Content-Type: application/json' -d '...'` |
| Form (URL-encoded) | `--data-urlencode key=value` |
| Multipart | `-F key=value` or `-F file=@/path/to/file` |
| Raw | `-H 'Content-Type: text/xml' -d '...'` |
| Binary | `--data-binary @/path/to/file` |

### 認証方式
- **Basic Auth** — `-u user:pass`
- **Bearer Token** — `-H 'Authorization: Bearer ...'`
- **API Key** — ヘッダーまたはクエリパラメータに追加
- **Digest Auth** — `--digest -u user:pass`
- **NTLM** — `--ntlm -u user:pass`
- **AWS SigV4** — `--aws-sigv4 aws:amz:region:service` (curl 7.75+)

### クエリパラメータ
自動でURLエンコードしてURLに付与。APIキーをクエリパラメータとして追加する場合も対応。

### curlオプション
`-v`、`-s`、`-L`、`-k`、`-i`、`-I`、`--compressed`、`--max-time`、`--retry`、`--limit-rate`、`-o`（出力ファイル）、`-x`（プロキシ）、`--cert`、`--cacert`、`-b`（Cookie）、`-A`（User-Agent）、`--resolve`など15種以上。

### 出力形式
- **Single line**: `curl -X POST -H '...' -d '...' https://example.com`
- **Multi-line (bash)**: `\` で改行
- **Windows (PowerShell)**: `^` で改行

## 生成例

POST + Bearer認証 + JSONボディ：

```bash
curl -X POST \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -L \
  -d '{"name":"Alice","email":"alice@example.com"}' \
  https://api.example.com/v1/users
```

## 実装のポイント

シェルクォートは意外と難しい。スペース、`$`、バックティック、セミコロンなどを含む文字列はシングルクォートで囲む必要があり、シングルクォート自体は `'\''` でエスケープ：

```javascript
function shellQuote(str) {
  if (!str) return "''";
  if (!/[\s'"$`\\!#&;|<>(){}[\]*?~]/.test(str)) return str;
  return "'" + str.replace(/'/g, "'\\''") + "'";
}
```

## テスト

78件のテスト全通過。

```bash
node tests/test.js
# Total: 78 | Passed: 78 | Failed: 0
```

---

[devnestio](https://devnestio.pages.dev) - 無料の開発者ツール集
