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

1. 依存パッケージのインストール

```bash
sudo apt update
sudo apt install -y ansible
```

2. インベントリ・変数の編集
- `inventory` に対象ホストを記載
- `host_vars/` に各ホストのホスト名・DNSサーバーを記載
- `group_vars/all.yml` で共通変数（サーチドメイン・SSHユーザーなど）を管理

3. 公開鍵の配置
- `files/id_rsa.pub` に配布したいSSH公開鍵を配置

4. 実行

```bash
ansible-playbook -i inventory playbook.yml
```

## 主な自動化内容
- ホスト名変更
- /etc/hosts の自動修正
- DNSサーバー・サーチドメイン設定（/etc/dhcpcd.conf）
- piユーザーのsudoパスワード不要化
- SSH公開鍵配布
- SSH公開鍵認証有効化・パスワード認証無効化
- 必要時のみ自動再起動

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
```

## 注意事項
- sudoers設定やSSH設定は慎重にご利用ください。
- 公開鍵（id_rsa.pub）は必ず正しいものを配置してください。
- 本リポジトリはベストプラクティスに基づいていますが、運用環境に応じて適宜カスタマイズしてください。

---

ご質問・ご要望はIssueやPRでお知らせください。