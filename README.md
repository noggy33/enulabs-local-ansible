# enulabs-local-ansible

Raspberry Pi & x86/Ubuntu混在クラスタ向け Ansible 構成例

## 概要

このリポジトリは、Raspberry Pi（Debian系OS）やx86/Ubuntuノードを対象としたK3sクラスタ自動構築・運用用のAnsibleプレイブックです。
- ホスト名・DNS・SSH・sudo・パッケージ・cgroup・ZFS/NFS・K3s・Helm・ラベル/テイント・監視系チャート等、クラスタ運用に必要な初期設定・基盤・アプリ自動化を網羅します。
- roles/配下のロールは全て原子性・保守性を重視し、サービス単位でファイル分割されています。

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
│   ├── pi4-worker01.yml
│   └── ubuntu-worker01.yml
├── files/                   # 配布用ファイル（公開鍵・k3s.yaml等）
│   ├── id_rsa.pub
│   ├── id_rsa.pub.dummy
│   ├── k3s.yaml.dummy
│   ├── kube-prometheus-stack-values.yaml
│   ├── loki-stack-values.yaml
│   ├── timescaledb-values.yaml
│   ├── jetstream-values.yaml
│   └── node-token.sample
├── roles/
│   ├── lab_os_ubuntu/       # Ubuntuノード初期化・強化
│   │   ├── tasks/{main,network,ssh,update,packages,swap,kernel,timezone}.yml
│   │   └── handlers/main.yml
│   ├── lab_os_pi4/          # Piノード初期化・強化
│   │   ├── tasks/{main,network,ssh,update,cgroup}.yml
│   ├── k3s_core_node/       # ZFS/NFS基盤・ラベル/テイント
│   │   ├── tasks/{main,packages,zfs,nfs,label,taint}.yml
│   │   └── handlers/main.yml
│   ├── k3s_core_helm/       # NFS CSI, Prometheus, Loki, TimescaleDB, JetStream等
│   │   └── tasks/{main,helm_repos,nfs_csi,kube_prometheus_stack,loki_stack,timescaledb,jetstream}.yml
│   ├── k3s_node_server/     # K3sサーバ（マスター）
│   │   └── tasks/main.yml
│   ├── k3s_node_agent/      # K3sエージェント（ワーカー）
│   │   └── tasks/main.yml
│   ├── k3s_ingress_helm/    # Ingress系Helmチャート
│   │   └── tasks/{main,metallb,ingress-nginx,traefik_cleanup}.yml
│   ├── k3s_ingress_node/    # Ingressノード用ラベル/テイント
│   │   └── tasks/{main,label,taint}.yml
│   └── k3s_cleanup/         # K3s/Helmリソース等のクリーンアップ
│       └── tasks/main.yml
└── README.md
```

## 主な自動化内容
- OS初期化（ネットワーク・SSH・パッケージ・swap・sysctl・cgroup・timezone）
- ZFS/NFS基盤自動構築
- K3sサーバ/エージェント自動セットアップ
- NFS CSI, Prometheus, Loki, TimescaleDB, JetStream等のHelm/kubectl自動化
- ノードラベル/テイント自動付与
- クリーンアップ/再構築用ロール
- 全タスク・ロールは原子性・保守性重視で分割

## 使い方

1. inventory, host_vars, group_vars, files/ を編集し、対象ノード・クラスタ構成・公開鍵等を定義
2. .envファイルで ANSIBLE_SSH_PASS, ANSIBLE_BECOME_PASS を設定し、環境変数として読み込む
   ```bash
   export $(cat .env | xargs)
   ```
3. playbook.yml を実行
   ```bash
   ansible-playbook -i inventory playbook.yml
   ```
4. 各ノードの初期化・K3sクラスタ・Helmリソース等が自動構築されます

## 注意事項
- SSH鍵・Vaultパスワード・k3s.yaml等の秘匿情報は必ずローカル専用で管理し、リポジトリには含めないでください
- .envファイルは.gitignoreで除外し、.env.dummyのみ配布してください
- roles/やtasks/の構成・分割は運用方針やクラスタ構成に応じて適宜カスタマイズしてください
- 詳細な運用例・変数例・トラブルシュートは旧READMEやplaybook内コメントも参照
- k3s.yaml.dummy, id_rsa.pub.dummy などダミーファイルを活用し、実運用時は本番用ファイルに差し替えてください