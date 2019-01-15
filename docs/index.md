<!-- markdownlint-disable MD012 -->
<!-- markdownlint-disable MD036 -->

# Ansible Role

## show running-configの出力をファイルに保存するロール

Takamitsu IIDA (@takamitsu-iida)

---

### インストール方法

```bash
ansible-galaxy install -p ./roles git+https://github.com/takamitsu-iida/ansible-role-cisco_show_run.git
```

フォルダの名前をご希望のロール名に変えてください。

---

## 使い方

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

## 変数

