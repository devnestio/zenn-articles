---
title: "Nginx Config GeneratorをVanilla JSで作った話（Static/Proxy/PHP-FPM・SSL・レートリミット）"
emoji: "⚙️"
type: "tech"
topics: ["claude", "ai", "devtools", "nginx", "javascript"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に **Nginx Config Generator** を追加しました。UIで設定を選ぶだけで本番用のnginx.confサーバーブロックを生成できます。

👉 https://nginx-config-gen.pages.dev

## なぜ作ったか

新しいサーバーやサービスをセットアップするたびにnginxの設定をドキュメントから引っ張り出していました。`proxy_set_header`の一覧、SSL設定の書き方、gzip対象MIMEタイプなど、毎回コピペする定形コードをUIから生成できるツールが欲しかった。

## 主な機能

- **3つのサーバーモード** — Static Site / Reverse Proxy / PHP-FPM
- **SSL/TLS** — Let's Encrypt証明書パス・TLSv1.2/1.3・ssl_session_cache・HSTSヘッダー・HTTP→HTTPSリダイレクト
- **Gzip圧縮** — gzip_types・gzip_comp_level 6・gzip_vary
- **ブラウザキャッシュ** — 静的ファイルのexpiresヘッダー（1日/7日/30日/1年）
- **セキュリティヘッダー** — X-Frame-Options・X-Content-Type-Options・X-XSS-Protection・Referrer-Policy・Permissions-Policy
- **レートリミット** — `limit_req_zone`ゾーン定義・burst設定
- **隠しファイル拒否** — `location ~ /\\.`でdotfileをdeny all
- **ロギング** — access_log・error_log・ログレベル設定

## 技術的なポイント：条件分岐でのYAML的構造

各設定は独立したフラグとして持ち、buildConfig()内で条件分岐してlinesに追加していく構造です。

```javascript
if (sslEn && sslRedirect) {
  // HTTP→HTTPSリダイレクト用の別serverブロックを先に出力
  lines.push('server {');
  lines.push('    listen 80;');
  lines.push('    return 301 https://$host$request_uri;');
  lines.push('}');
}
```

PHPモードでは`try_files $uri $uri/ /index.php?$query_string`でフロントコントローラーパターンを生成し、`.php`ロケーションでfastcgi_passを設定します。

## プロキシモードのヘッダー設定

WebSocket対応のUpgrade/Connectionヘッダーも含め、実運用で必要な`proxy_set_header`を網羅しています。

```nginx
proxy_set_header   Upgrade $http_upgrade;
proxy_set_header   Connection "upgrade";
proxy_set_header   Host $host;
proxy_set_header   X-Real-IP $remote_addr;
proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header   X-Forwarded-Proto $scheme;
proxy_cache_bypass $http_upgrade;
proxy_read_timeout 90s;
```

## テスト

buildConfig()関数をNode.jsで直接テストする形で80件以上のアサーションを書いています。すべての組み合わせで「あるべき文字列が含まれるか」「ないべき文字列が含まれないか」を確認しています。

```
$ node tests/test.js
All tests passed! (80+ assertions)
```

## ハブへの追加

devnestioのハブに「DevOps」カテゴリで追加しました。

---

devnestioは開発者向け無料ツール群です。ログイン不要・トラッキングなし。
