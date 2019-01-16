<!-- markdownlint-disable MD012 -->
<!-- markdownlint-disable MD036 -->

# Ansible Role

<br>

## cisco_show_run

show running-configの出力をファイルに保存するロールです。

<br><br>

Takamitsu IIDA (@takamitsu-iida)

---

### はじめに

ネットワーク機器の設定を変更する場合、作業の前後でログをファイルに保存します。

ログ採取を簡単にできるようにするロールを作っておくと何かと便利です。

---

### ロールのインストール方法

作成したロールをgithubに置いておき、ansible-galaxyでインストールすると便利です。

```bash
ansible-galaxy install -p ./roles
 git+https://github.com/takamitsu-iida/ansible-role-cisco_show_run.git
```

-pでロールを格納するフォルダを指定します。

cisco_show_runというフォルダ名でインストールされますので、ご希望のロール名に合わせてフォルダ名を変えてください。

---

### 使い方

設定変更するタスクの前後に挿入するとよいでしょう。

```yml
- name: show running-config
  hosts: routers
  gather_facts: false

  tasks:

    - include_role:
        name: cisco_show_run

    - 設定変更するタスク

    - include_role:
        name: cisco_show_run
```

---

### 変数

