---
title: "Docker Compose GeneratorをVanilla JSで作った話（マルチサービスYAMLビルダー・プリセット付き）"
emoji: "🐳"
type: "tech"
topics: ["claude", "ai", "devtools", "docker", "javascript"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に **Docker Compose Generator** を追加しました。UIでサービスを追加・設定するだけで`docker-compose.yml`を生成できます。

👉 https://docker-compose-gen-7l6.pages.dev

## なぜ作ったか

新しいプロジェクトを始めるたびに`docker-compose.yml`の書き方を調べていました。特にvolumesのnamed volumeとbind mountの書き分け、networksブロックのdriver設定、healthcheckの書式など、毎回確認が必要な定形コードをUIから生成できるツールが欲しかった。

## 主な機能

- **マルチサービスUI** — サービスを動的に追加/削除、各サービスは折りたたみ可能
- **各サービス設定** — image・build context・restart policy・command・healthcheck・depends_on・共有ネットワーク
- **ポート・ボリューム・環境変数** — 行単位で追加/削除
- **Named volumeの自動宣言** — `./`や`/`で始まらないボリュームを検出してトップレベルのvolumesブロックに追加
- **ネットワーク自動生成** — 1つ以上のサービスがネットワーク接続時にnetworksブロックを生成
- **クイックプリセット** — Node.js+Postgres / WordPress+MySQL / Redis / Nginx+App

## 技術的なポイント：Reactなしでの動的UI

ライブラリゼロで動的なUIを実現するため、サービス配列をステートとして持ち、変更のたびにUIを全再レンダリングする方式を採りました。

```javascript
let services = [];

function renderUI() {
  const list = document.getElementById('service-list');
  list.innerHTML = services.map(s => `...`).join('');
}

function addService(overrides = {}) {
  services.push(newService(overrides));
  renderUI();
  render(); // YAML再生成
}
```

フォームの値は`render()`呼び出し前に`readService(id)`でDOMから読み取り、ステートに反映させます。

## Named volumeの自動検出

`./`や`/`で始まるパスはbind mount（ホスト側パスを直接マウント）、それ以外はnamed volumeとして扱います。Named volumeはトップレベルのvolumesブロックへの宣言が必要です。

```javascript
const namedVols = services.flatMap(s =>
  s.volumes
    .filter(v => v && !v.startsWith('./') && !v.startsWith('/') && v.includes(':'))
    .map(v => v.split(':')[0])
);
const uniqueVols = [...new Set(namedVols)]; // 重複除去
```

## YAMLの生成

テンプレートエンジンを使わず、配列に行を積み上げて最後に`join('\n')`する方式です。インデントを文字列連結で管理するため、スペース数のバグが混入しやすく、テストでインデントを厳密にチェックしています。

## テスト

80件以上のアサーション。特に以下を重点的にテストしました：

- Named volumeの重複除去（2サービスが同じvolume名を使うとき1回だけ宣言されるか）
- Proxyモードでは`expires`（キャッシュ）が出力されないこと
- `depends_on`のカンマ区切りが複数エントリに正しく展開されるか
- YAMLインデント（ポートは6スペース、サービスは2スペースなど）

```
$ node tests/test.js
All tests passed! (80+ assertions)
```

## ハブへの追加

devnestioのハブに「DevOps」カテゴリで追加しました。

---

devnestioは開発者向け無料ツール群です。ログイン不要・トラッキングなし。
