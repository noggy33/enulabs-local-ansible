# My Ansible Playbook

このプロジェクトは、Ansibleを使用して管理対象のホストに対してタスクを自動化するためのプレイブックを提供します。

## プロジェクト構成

- **playbook.yml**: Ansibleのプレイブックファイルで、実行するタスクやロールを定義します。
- **inventory**: 管理対象のホストを定義するインベントリファイルです。ホストのグループや接続情報が含まれます。
- **roles/sample_role/tasks/main.yml**: このファイルには、ロールに関連するタスクが定義されています。Ansibleが実行する具体的な操作が記述されています。
- **roles/sample_role/handlers/main.yml**: このファイルには、タスクの結果に応じて実行されるハンドラーが定義されています。例えば、サービスの再起動などが含まれます。
- **roles/sample_role/templates**: Jinja2テンプレートファイルを格納するディレクトリです。動的に生成される設定ファイルなどが含まれます。
- **roles/sample_role/files**: プレイブックで使用する静的ファイルを格納するディレクトリです。
- **group_vars/all.yml**: 全てのホストに適用される変数を定義するファイルです。
- **host_vars/example_host.yml**: 特定のホストに適用される変数を定義するファイルです。

## 使用方法

1. インベントリファイルを編集して、管理対象のホストを定義します。
2. 必要に応じて、`group_vars`や`host_vars`で変数を設定します。
3. プレイブックを実行するには、以下のコマンドを使用します。

   ```
   ansible-playbook -i inventory playbook.yml
   ```

このプロジェクトを使用して、Ansibleによる自動化を簡単に実現できます。