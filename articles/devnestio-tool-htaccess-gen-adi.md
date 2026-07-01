---
title: "Apacheの.htaccessをブラウザで生成するツールを作った"
emoji: "⚙️"
type: "tech"
topics: ["javascript","apache","htaccess","webdev","browser"]
published: false
---

## はじめに

Apache `.htaccess` ファイルは強力ですが、書き方を忘れがちです。セクションごとにトグルできるビジュアルジェネレーターをブラウザのみで実装しました。

**ツール:** https://devnestio.pages.dev/htaccess-gen/

## 対応セクション

| セクション | 内容 |
|---|---|
| リダイレクト | 301/302 Redirect、from/to パス |
| HTTPS強制 | RewriteEngine + HTTPS条件 |
| Basic認証 | AuthType、AuthName、AuthUserFile |
| CORSヘッダー | Access-Control-Allow-* |
| エラーページ | ErrorDocument 404/403/500 |
| ディレクトリ | Options Indexes、FollowSymLinks |

## セクションのトグル

```js
const sections = {
  redirects: true, https: true, auth: false,
  cors: false, errors: true, dirlisting: false
};

function toggleSection(name) {
  sections[name] = !sections[name];
  const btn = document.getElementById('toggle-' + name);
  btn.classList.toggle('on', sections[name]);
  generate();
}
```

## シンタックスハイライト

```js
function generate() {
  let html = '', raw = '';
  function line(htmlPart, rawPart) { html += htmlPart + '\n'; raw += rawPart + '\n'; }

  if (sections.https) {
    line(span('c-comment', '# Force HTTPS'), '# Force HTTPS');
    line(span('c-directive', 'RewriteEngine') + ' ' + span('c-on', 'On'), 'RewriteEngine On');
    line(span('c-directive', 'RewriteCond') + ' %{HTTPS} off', 'RewriteCond %{HTTPS} off');
    line(span('c-directive', 'RewriteRule') + ' ^ https://%{HTTP_HOST}%{REQUEST_URI} [' + span('c-value', 'R=301,L') + ']',
         'RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]');
  }
  // ...
  document.getElementById('htaccess-output').innerHTML = html;
  document.getElementById('htaccess-output').dataset.raw = raw;
}
```

## ダウンロード

```js
function downloadOutput() {
  const raw = document.getElementById('htaccess-output').dataset.raw;
  const blob = new Blob([raw], { type: 'text/plain' });
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = '.htaccess';
  a.click();
}
```

`dataset.raw` にプレーンテキストを保存してシンタックスハイライトHTMLと共存させるのがポイントです。
