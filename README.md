# enulabs-local-ansible

Raspberry Pi クラスタ向け Ansible 構成例

## 概要

このリポジトリは、Raspberry Pi（Debian系OS）を対象とした初期セットアップ用のAnsibleプレイブックです。
ホスト名・DNS・SSH公開鍵・sudo設定など、クラスタ運用に必要な基本設定を自動化します。

## ディレクトリ構成

```
├── inventory                # インベントリファイル
├── playbook.yml             # メインプレイブック
├── group_vars/              # グループ共通変数
│   └── all.yml
├── host_vars/               # ホスト個別変数
│   ├── pi4-master01.yml
│   ├── pi4-master02.yml
│   ├── pi4-master03.yml
│   └── pi4-worker01.yml
├── files/                   # 配布用ファイル（公開鍵など）
│   └── id_rsa.pub
├── roles/
│   ├── network/             # ネットワーク・ホスト名・DNS・sudo設定
│   │   └── tasks/main.yml
│   └── ssh/                 # SSH公開鍵・認証方式
│       ├── tasks/main.yml
│       └── handlers/main.yml
└── README.md
```

## 使い方

### 1. 依存パッケージのインストール
```bash
sudo apt update
sudo apt install -y ansible
```

### 2. インベントリ・変数・公開鍵の準備
- `inventory` に対象ホストを記載
- `host_vars/` に各ホストのホスト名・DNSサーバー・ssh_password_authを記載
- `group_vars/all.yml` で共通変数（サーチドメイン・SSHユーザー・公開鍵パスなど）を管理
- `files/id_rsa.pub` は**ローカル専用**（本物の公開鍵）。**リポジトリには含めず、`.gitignore`で除外**します。
- `files/id_rsa.pub.dummy` は**ダミー公開鍵**で、リポジトリに含めます。

> **運用例**: 本物の公開鍵は `files/id_rsa.pub` としてローカルに配置し、リポジトリには `files/id_rsa.pub.dummy` のみを含めてください。

### 3. Ansible Vaultパスワードファイルの準備（Vault利用時のみ）
```bash
echo '（Vaultパスワード）' > ~/.vault_pass.txt
chmod 600 ~/.vault_pass.txt
```

### 4. プレイブックの実行
```bash
ansible-playbook -i inventory playbook.yml --vault-password-file ~/.vault_pass.txt
```

- パスワードファイル名やパスは環境に合わせて変更してください。
- Vault未使用の場合は `--vault-password-file` オプション不要です。

### 5. 実行結果の確認
- 各ノードでsshdが再起動され、SSH設定が反映されていることを確認してください。
- エラーが出た場合は、VaultパスワードファイルやSSH鍵、インベントリ設定を見直してください。

---

## SSH認証方式の切り替え運用例

1. **初回セットアップ**
   - `host_vars/` の `ssh_password_auth` を `yes` に設定
   - パスワード認証で公開鍵を配布
   - プレイブックを実行

2. **公開鍵配布後**
   - `ssh_password_auth` を `no` に変更
   - 再度プレイブックを実行し、パスワード認証を無効化

> ※この手順で安全にSSH公開鍵認証へ移行できます。

---

## 主な自動化内容
- ホスト名変更
- /etc/hosts の自動修正
- DNSサーバー・サーチドメイン設定（/etc/dhcpcd.conf）
- piユーザーのsudoパスワード不要化
- SSH公開鍵配布
- SSH公開鍵認証有効化・パスワード認証無効化
- 必要時のみ自動再起動
- **全ノードで明示的にsshd再起動**
- **パッケージアップデート（roles/update）**

## 変数例

### group_vars/all.yml
```yaml
search_domain: enulabs.local
ssh_pubkey_file: files/id_rsa.pub
ssh_user: pi
```

### host_vars/pi4-master01.yml
```yaml
hostname: pi4-master01.enulabs.local
dns_nameservers:
  - 192.168.4.254
ssh_password_auth: 'no'
```

## 注意事項
- sudoers設定やSSH設定は慎重にご利用ください。
- 公開鍵（id_rsa.pub）は必ず正しいものを**ローカル専用で配置**してください。
- **秘密鍵（id_rsa等）やVaultパスワードファイルは絶対にリポジトリに含めないでください。**
- コミット前に `git status` で不要なファイルが含まれていないか必ず確認してください。
- 本リポジトリはベストプラクティスに基づいていますが、運用環境に応じて適宜カスタマイズしてください。
- **推奨Ansibleバージョン: 2.10以上**

---

## Ansible Vaultの使い方まとめ

### ファイルの暗号化
```bash
ansible-vault encrypt group_vars/all.yml
```

### ファイルの復号化
```bash
ansible-vault decrypt group_vars/all.yml
```

### 暗号化ファイルの編集
```bash
ansible-vault edit group_vars/all.yml
```

### 暗号化ファイルの内容を表示
```bash
ansible-vault view group_vars/all.yml
```

### プレイブック実行時（Vaultパスワードを聞かれる場合）
```bash
ansible-playbook -i inventory playbook.yml --ask-vault-pass
```

---

ご質問・ご要望はIssueやPRでお知らせください。