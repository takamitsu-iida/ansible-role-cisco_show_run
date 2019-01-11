show_run
=========

Cisco IOS装置に対してshow runningコマンドを実行し、出力結果をファイルに保存します。

ansible-galaxyコマンドでダウンロードしたあと、フォルダの名前をお好みのロール名に変えてください。

```bash
ansible-galaxy install -p ./roles git+https://github.com/takamitsu-iida/ansible-role-cisco_show_run.git
```

Requirements
------------

対象はCisco装置であることを前提にしています。
対象機器にTelnetで接続したい場合は別モジュールが別途必要です。

<https://github.com/takamitsu-iida/ansible-mytelnet>

Role Variables
--------------

CMD_LIST: 実行したいコマンドを配列で指定します。デフォルトはshow running-configです。

CREATE_LOG: ログをファイルに保存するかをブール値で指定します。デフォルトはtrueです。

PREFIX: ログファイルの先頭につける文字列です。デフォルトは''です。

LOG_PATH: ログを残すフォルダです。デフォルトは `"{{ lookup('env', 'PWD') + '/log' }}"` です。

Dependencies
------------

依存するロールはありません。

Example Playbook
----------------

設定変更するタスクの前後に挿入するとよいでしょう。

```yml
- name: show running-config
  hosts: routers
  gather_facts: false

  tasks:
    - include_role:
        name: show_run

    - 設定変更するタスク

    - include_role:
        name: show_run
```

作業前と作業後でログ置き場を変えたい場合は、このようにします。
作業前はbefore、作業後はafterにログが残ります。

```yml
- name: show running-config
  hosts: routers
  gather_facts: false

  tasks:
    - include_role:
        name: show_run
      vars:
        LOG_PATH: "{{ lookup('env', 'PWD') + '/log/before' }}"

    - 設定変更するタスク

    - include_role:
        name: show_run
      vars:
        LOG_PATH: "{{ lookup('env', 'PWD') + '/log/after' }}"
```

ログの置き場は同じにして、ファイル名の先頭になにか付けたい場合は、このようにします。

```yml
- name: show running-config
  hosts: routers
  gather_facts: false

  tasks:
    - include_role:
        name: show_run
      vars:
        PREFIX: before_

    - 設定変更するタスク

    - include_role:
        name: show_run
      vars:
        PREFIX: after_
```

License
-------

BSD

Author Information
------------------

Takamitsu IIDA
