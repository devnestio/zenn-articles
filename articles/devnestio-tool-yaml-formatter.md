---
title: "YAML Formatterをブラウザで完結させた話（インフラ・k8s/Dockerエンジニア向け）"
emoji: "📋"
type: "tech"
topics: ["yaml", "devtools", "cloudflare", "javascript", "kubernetes"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に、**YAML Formatter & Validator** を追加しました。

👉 https://devnestio.pages.dev/yaml-formatter/

YAMLの書き方ミスをブラウザ上で即チェックでき、JSON↔YAML の双方向変換にも対応しています。外部ライブラリゼロ・単一HTMLファイル・サーバー送信なし（すべてブラウザ内で完結）という、devnestioの共通方針に沿って実装しています。

---

## 作った動機

インフラ・k8s・Docker 周りの作業では、YAML を書く場面が日常的に発生します。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    spec:
      containers:
        - name: my-app
          image: my-app:latest
```

こういった Kubernetes Manifest や Docker Compose、GitHub Actions の YAML を書いていると、インデントミス・コロン漏れ・タブ混入などのエラーが頻発します。`kubectl apply` や `docker compose up` を実行して初めてエラーに気づく、というのはあるあるです。

既存のオンラインYAMLバリデーターは、コンフィグファイルをサーバーに送信するものが多く、**クラウドインフラの設定ファイルをサードパーティに送りたくない**という場面ではためらいます。完全クライアントサイドで動くものを自分で持っておきたい、というのが動機です。

---

## 主な機能

### 1. フォーマット・インデント正規化

```yaml
# 入力（インデントがバラバラ）
name: John
address:
 street: "123 Main St"
  city: Springfield

# 出力（2スペースで正規化）
name: John
address:
  street: 123 Main St
  city: Springfield
```

ペーストした瞬間に2スペースインデントで整形されます。

### 2. バリデーション＋行番号エラー表示

YAMLの構文エラーがある場合、エラーメッセージに**行番号**が表示されます。

```
Error: bad indentation of a mapping entry at line 4, column 3
```

`kubectl apply` の「Invalid YAML」エラーより遥かに親切です。コードエディタを開かなくてもブラウザのタブ1枚で原因を特定できます。

### 3. YAML → JSON 変換

Kubernetes の Manifest を JSON で書くケースや、API レスポンスの検証時に YAML→JSON したい場面で使えます。

```yaml
# YAML
server:
  host: localhost
  port: 8080
  debug: true
```

```json
{
  "server": {
    "host": "localhost",
    "port": 8080,
    "debug": true
  }
}
```

### 4. JSON → YAML 変換

`terraform output -json` や AWS CLI のレスポンスを YAML Manifest に変換するときに便利です。逆方向（JSON→YAML）も一発変換できます。

### 5. ミニファイ

コメントを除去してコンパクトな JSON 形式で出力します。CI/CDパイプラインに埋め込む値の圧縮などに使えます。

---

## 対応するYAML構造

手書きパーサーを実装しているため、対応範囲を明示しておきます。

- ✅ ネストしたマッピング（Kubernetes spec の深いネスト）
- ✅ シーケンス（`- item` 記法）
- ✅ ブロックスカラー（`|` リテラル、`>` フォールデッド）
- ✅ クォート文字列（`'single'` / `"double"`）
- ✅ インラインコメント（`# comment`）
- ✅ 数値・真偽値・null の型判定
- ⚠️ アンカー（`&anchor`）・エイリアス（`*alias`）は部分対応

---

## 技術的なポイント：なぜ外部ライブラリを使わないか

`js-yaml` などの npm パッケージを使えば実装は圧倒的に楽です。しかし devnestio では「単一 HTML ファイル・外部依存ゼロ」をポリシーとしています。

理由は2つ：

**1. ユーザーの信頼性**  
CDN から読み込む外部スクリプトは、利用者の設定ファイルをどこかに送る可能性があります。「ライブラリが何かしている」という疑念をゼロにするため、自前実装にしています。

**2. オフライン動作**  
初回アクセス以降はブラウザキャッシュで完全オフライン動作します。VPN接続中・機内モード・ステージング環境など、インターネットが制限された場面でも使えます。

---

## 実装はAIエージェントに任せた

devnestioのツールは、Claude Code sessions に発注書（PRD）を渡して自律的に実装してもらう形で作っています。

今回のポイントは、YAMLパーサーの実装方針を発注書に詳細に書いたことです。「Kubernetes Manifest / Docker Compose / GitHub Actions YAML を対象とする、ネストしたマッピングとシーケンスは必須、ブロックスカラーも対応、js-yaml は使用禁止」という具体的な指示を与えることで、エッジケースの考慮漏れが大幅に減りました。

**テスト結果:** ブラウザ実行型テスト（test.html）で **114/114 PASS**

---

## こんな場面で使ってください

- Kubernetes Manifest を書いて `kubectl apply` する前の事前チェック
- Docker Compose ファイルのインデントを整えたい
- GitHub Actions の workflow YAML でエラーになった行を特定したい
- Terraform の HCL を JSON/YAML で書き換えたい
- REST API から返ってきた JSON を YAML に変換して可読性を上げたい

---

## 使ってみてください

👉 **[YAML Formatter & Validator — devnestio](https://devnestio.pages.dev/yaml-formatter/)**

全ツール一覧: https://devnestio.pages.dev

フィードバック（対応できていないYAML構造・エラーメッセージの改善点など）はコメントで大歓迎です。
