---
title: "Dockerコマンド早見表＋コマンドビルダーをブラウザで作った（40コマンド・Composeスニペット付き）"
emoji: "🐳"
type: "tech"
topics: ["docker", "devops", "devtools", "cloudflare", "javascript"]
published: true
---

## 作ったもの

開発者向け無料ツール群「devnestio」に、**Docker Cheatsheet** を追加しました。

👉 https://docker-cheatsheet.pages.dev

40以上のDockerコマンドを検索・フィルタリングでき、フラグをクリックするだけでコマンドを組み立てられるインタラクティブなビルダーが付いています。Docker Composeのスニペットも4種類収録。

---

## 作った動機

Dockerを日常的に使っていても、たまにしか叩かないコマンドのフラグを毎回検索することがあります。

```bash
# これは覚えている
docker run -d -p 8080:80 nginx

# でもこれは毎回調べる
docker stats --format '{{.Name}}: {{.CPUPerc}}'
docker system prune -af --volumes
docker build --target production --platform linux/amd64 .
docker logs --since 1h --timestamps --tail 50 myapp
```

man pageやDocker公式ドキュメントは充実していますが、「あのフラグ何だっけ」という場面でサクッと確認できるツールが手元にあると便利です。

---

## 7カテゴリ・40以上のコマンド

| カテゴリ | 主なコマンド |
|---|---|
| containers | run, start, stop, restart, rm, ps, logs, inspect, stats, cp, pause, top, update |
| images | build, pull, push, images, rmi, tag, history, save, load |
| network | network create/ls/rm/inspect/connect/disconnect |
| volumes | volume create/ls/rm/inspect/prune |
| exec | exec, attach |
| system | system prune/df/info/events, image prune, container prune |
| registry | login, logout, search |

---

## コマンドビルダー

コマンドカードをクリックすると展開され、フラグがタグ形式で表示されます。クリックするたびにコマンドに追加・削除されます。

```
[docker run] を展開すると:

[-d (detach)]  [-p 8080:80 (port map)]  [-v /host:/container (volume)]
[-e KEY=val (env var)]  [--name myapp (name)]  [--rm (auto remove)]
[--network mynet]  [--restart always]  [--memory 512m]  [--cpus 1.5]

クリックして選択:
→ docker run -d -p 8080:80 --name myapp
```

コピーボタンで1クリックでクリップボードへ。

---

## Docker Composeスニペット

よく使うCompose構成を4種類収録：

### 1. Basic Web + DB
```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    depends_on:
      - db
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: mydb
```

### 2. Node.js + Redis
```yaml
services:
  app:
    build: .
    environment:
      - REDIS_URL=redis://redis:6379
    depends_on:
      - redis
  redis:
    image: redis:7-alpine
```

### 3. Nginx Reverse Proxy
### 4. Multi-stage + Health Check（CPUとメモリ制限付き）

各スニペットにCopyボタンが付いています。

---

## 検索の使い方

```
"volume"     → volume関連コマンドをすべて表示
"prune"      → system prune, image prune, container prune, volume prune
"interactive"→ docker exec（-it フラグの説明にマッチ）
"follow"     → docker logs（-f フラグの説明にマッチ）
"dangling"   → docker images, docker volume ls（フラグにマッチ）
```

コマンド名・説明・フラグ・exampleの4フィールドを横断検索します。

---

## テスト結果

Node.js `assert` モジュールで **82件のテスト**をすべてパス。

```
[Data integrity]      20 tests
[filterCommands]      20 tests
[buildCommand]         6 tests
[toggleFlag]           6 tests
[getCategories]        4 tests
[COMPOSE_SNIPPETS]     8 tests
[Integration]         10 tests + 8 extras
────────────────────────────────────────
Results: 82 passed, 0 failed
```

---

## 使ってみてください

👉 **[Docker Cheatsheet — devnestio](https://docker-cheatsheet.pages.dev)**

全ツール一覧: https://devnestio.pages.dev

「このコマンドが足りない」「このフラグが抜けている」というフィードバックはコメントで大歓迎です。
