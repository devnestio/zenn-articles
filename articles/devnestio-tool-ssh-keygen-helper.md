---
title: "SSH Key Generator HelperをVanilla JSで作った（コマンドビルダー・鍵種解説付き）"
emoji: "🔑"
type: "tech"
topics: ["claude", "ai", "devtools", "ssh", "javascript"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に **SSH Key Generator Helper** を追加しました。鍵の種類・ビット数・ファイルパス・パスフレーズ設定を選択すると、正確な`ssh-keygen`コマンドをその場で生成します。

👉 https://ssh-keygen-helper.pages.dev

## なぜ作ったか

SSHキーを生成するとき、`-t`オプションの意味や`-b`で指定すべきビット数を毎回調べていました。Ed25519とRSAのどちらを使うべきか、KDFラウンド（`-a`オプション）とは何かなど、説明を読みながらコマンドを組み立てるツールが欲しかった。

## 主な機能

- **鍵種選択UIカード** — Ed25519（推奨）・RSA・ECDSA・DSAをカード形式でセキュリティレーティング付きで比較
- **コマンドビルダー** — 選択内容から`ssh-keygen`コマンドをリアルタイム生成
- **パスフレーズ・KDFラウンド設定** — `-N ""`オプションや`-a`オプションも対応
- **公開鍵コピー&デプロイコマンド** — `cat ~/.ssh/id_ed25519.pub`や`ssh-copy-id`コマンドも自動生成
- **SSHチップスタブ** — `known_hosts` / `authorized_keys` / `~/.ssh/config` / `ssh-agent`の4タブ

## 技術的なポイント：鍵種ごとの設定差分

鍵種によってビット数の選択肢が異なります。Ed25519はアルゴリズム上256ビット固定で`-b`フラグが無意味、RSAは2048/3072/4096/8192から選択、ECDSAは256/384/521の3曲線に対応します。

```javascript
const keyConfigs = {
  ed25519: { bits: [{ label: 'Default (256-bit)', val: '' }] },
  rsa:     { bits: [{ val: '2048' }, { val: '3072' }, { val: '4096' }, { val: '8192' }] },
  ecdsa:   { bits: [{ val: '256' }, { val: '384' }, { val: '521' }] },
  dsa:     { bits: [{ val: '1024' }] }, // 非推奨
};
```

DSAカードには「OpenSSH 7.0以降で無効化」の警告を赤字で表示し、誤った選択を防ぎます。

## コマンド生成ロジック

オプションが空の場合はフラグを出力しない設計にしています。

```javascript
function buildCommand({ type, bits, file, comment, noPass, rounds }) {
  let parts = ['ssh-keygen', `-t ${type}`];
  if (bits)    parts.push(`-b ${bits}`);
  if (file)    parts.push(`-f ${file}`);
  if (comment) parts.push(`-C "${comment}"`);
  if (noPass)  parts.push('-N ""');
  if (rounds)  parts.push(`-a ${rounds}`);
  return parts.join(' \\\n  ');
}
```

## テスト

80件以上のアサーションで鍵種・ビット・ファイルパス・コメント・パスフレーズ・KDFラウンドの全組み合わせを検証しています。パスインジェクション（`;`, `|`, バッククォートなど）の検出テストも含めました。

```
$ node tests/test.js
All tests passed! (80+ assertions)
```

## ハブへの追加

devnestioのハブに「DevOps」カテゴリで追加しました。

---

devnestioは開発者向け無料ツール群です。ログイン不要・トラッキングなし。
