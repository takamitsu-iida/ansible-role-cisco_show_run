cisco_show_run
==============

Cisco IOS装置に対してshow runningコマンドを実行し、出力結果をファイルに保存します。

ansible-galaxyコマンドでダウンロードしたあと、フォルダの名前をお好みのロール名に変えてください。

```bash
ansible-galaxy install -p ./roles git+https://github.com/takamitsu-iida/ansible-role-cisco_show_run.git
```

実際のログ取りではshow running-configだけではなく、いろんなコマンドを打ち込むことになると思います。
その際にテキスト形式で残すだけではなく、CSVに加工して保存したりすると便利だと思います。

応用編はこちら。

<https://github.com/takamitsu-iida/ansible-role-cisco_show_commands>

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

LOG_PATH: ログを残すフォルダです。デフォルトはカレントディレクトリ直下のlogフォルダ `"{{ lookup('env', 'PWD') + '/log' }}"` です。

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
        name: cisco_show_run

    - 設定変更するタスク

    - include_role:
        name: cisco_show_run
```

実行すると、このようにログファイルが生成されます。

```bash
log
├── r1_show_run_2019-01-11@12:07:13.txt
├── r1_show_run_2019-01-11@12:07:30.txt
├── r2_show_run_2019-01-11@12:07:13.txt
├── r2_show_run_2019-01-11@12:07:30.txt
├── r3_show_run_2019-01-11@12:07:13.txt
├── r3_show_run_2019-01-11@12:07:30.txt
├── r4_show_run_2019-01-11@12:07:13.txt
└── r4_show_run_2019-01-11@12:07:30.txt
```

ファイル名に時刻情報が含まれるので、どのファイルが作業前でどのファイルが作業後なのか、わからなくもないですが・・・

プレイブックを何度も実行すると、何が何だかわからなくなってしまうと思います。

作業前と作業後でログ置き場を変えたい場合は、ロール変数LOG_PATHを変更します。

```yml
- name: show running-config
  hosts: routers
  gather_facts: false

  tasks:
    - include_role:
        name: cisco_show_run
      vars:
        LOG_PATH: "{{ lookup('env', 'PWD') + '/log/before' }}"

    - 設定変更するタスク

    - include_role:
        name: cisco_show_run
      vars:
        LOG_PATH: "{{ lookup('env', 'PWD') + '/log/after' }}"
```

実行すると、作業前のログはbeforeに、作業後はafterにファイルが作られます。

```bash
log
├── after
│   ├── r1_show_run_2019-01-11@12:10:53.txt
│   ├── r2_show_run_2019-01-11@12:10:53.txt
│   ├── r3_show_run_2019-01-11@12:10:53.txt
│   └── r4_show_run_2019-01-11@12:10:54.txt
└── before
    ├── r1_show_run_2019-01-11@12:10:37.txt
    ├── r2_show_run_2019-01-11@12:10:37.txt
    ├── r3_show_run_2019-01-11@12:10:37.txt
    └── r4_show_run_2019-01-11@12:10:37.txt
```

ログファイルの置き場は同じにして、ファイル名の先頭で区別を付けたい場合は、このようにします。

```yml
- name: show running-config
  hosts: routers
  gather_facts: false

  tasks:
    - include_role:
        name: cisco_show_run
      vars:
        PREFIX: before_

    - 設定変更するタスク

    - include_role:
        name: cisco_show_run
      vars:
        PREFIX: after_
```

実行するとこのようにファイルが作られます。

```bash
log/
├── after_r1_show_run_2019-01-11@12:05:42.txt
├── after_r2_show_run_2019-01-11@12:05:43.txt
├── after_r3_show_run_2019-01-11@12:05:43.txt
├── after_r4_show_run_2019-01-11@12:05:43.txt
├── before_r1_show_run_2019-01-11@12:05:26.txt
├── before_r2_show_run_2019-01-11@12:05:26.txt
├── before_r3_show_run_2019-01-11@12:05:26.txt
└── before_r4_show_run_2019-01-11@12:05:26.txt
```

License
-------

BSD

Author Information
------------------

Takamitsu IIDA

<br>
<br>

task作成メモ
==========

ログをファイルに残すだけでも、ゼロから作ると意外に面倒です。

タスク作成時のポイントをメモしておきます。

## カレントディレクトリの取得

カレントディレクトリの情報ならfactにあるのでは？と思うのですが、意外にこれが取りづらいのです。
`ansible -m setup localhost` を実行するとこのように `ansible_env.PWD` にカレントディレクトリが入っていることがわかります。

```json
"ansible_env": {
    "PWD": "/Users/iida/git/internal/i-ansible-labo",
```

ですが、ターゲット装置にローカルホストを指定してファクト収集しているわけではないので、プレイブックで取り出すのは面倒です。
仕方ないのでこのロールでは環境変数をlookupで取り出しています。

```yml
LOG_PATH: "{{ lookup('env', 'PWD') }}"
```

## ログを保管するディレクトリの作成

```yml
- name: create log directory if not exists
  delegate_to: localhost
  file:
    path: "{{ LOG_PATH | default('./log') }}"
    state: directory
    recurse: true
  run_once: true
  when: CREATE_LOG
```

ログを残すのは制御ノード上ですので **delegate_to** を使ってローカルホストでファイルを作成します。
意外と忘れがちです。

フォルダの作成は一度やれば十分なので、**run_once** をtrueにします。これも忘れがちです。

## 現在時刻の取得

```yml
- name: get current time
  set_fact:
    # 2018-08-28@11:17:34
    date: "{{ lookup('pipe', 'date +%Y-%m-%d@%H:%M:%S') }}"
```

これもlocalhostでファクトを採取すれば取り出せるのですが・・・

このロールでは **lookup** でpipeを呼び出してdateコマンドを実行しています。
この方がフォーマットの指定もできて楽です。

## 失敗してもログを残す

```yml
-
  block:
    - name: run exec commands on remote nodes bia SSH
      ios_command:
        commands: "{{ CMD_LIST }}"
      register: result

  rescue:
    - name: log failed result
      delegate_to: localhost
      copy:
        content: "{{ result | to_nice_json(indent=2) }}"
        dest: "{{ LOG_PATH + '/' + '_FAILED_' + filename }}"
```

show running-configを打ち込むくらいなら失敗することはないと思うのですが、show tech-supportになると時間もかかりますしエラーになることもあるでしょう。
エラー発生時には戻り値を含め、全て保存しておくと後々のためになるでしょう。
**block** と **rescue** を使って失敗時にファイルを保存しています。

## 複数のコマンドを表示、保存する

```yml
- name: save
  delegate_to: localhost
  copy:
    content: |
      {% for item in CMD_LIST -%}
      === {{ item }} ===
      {{ stdout[loop.index0] }}
      {% endfor %}
    dest: "{{ LOG_PATH + '/' + filename }}"
```

ios_commandモジュールで複数のコマンドを打ち込むと、結果は **stdout配列** に格納されて戻ってきます。
この戻り値に、実行したコマンドは含まれていません。
なのでstdout配列の中身だけをベタに書き込むと、なんのコマンドの出力なんだっけ？ということになってしまいます。

画面に表示したりファイルに保存するときには、
打ち込んだコマンドの配列と、結果のstdout配列の両方をループで回してあげる必要があります。

結果はこのようになります。

```bash
=== show running-config ===
Building configuration...

Current configuration : 3143 bytes
!
! Last configuration change at 13:57:03 UTC Sun Apr 14 2019 by cisco
!
```

区切り（=== コマンド ===）は後から加工しやすいように、もう少し工夫した方がいいかもしれません。

## 画面表示するときのstdout_callback指定

**ios_command** の戻り値(stdout)を画面表示すると、改行されずに"\n"で表示されると思います。
これだと見づらいので、ansible.cfgの設定で以下のように設定します。
なぜこれがデフォルトじゃないのか理解に苦しみます。

```ini
[defaults]

stdout_callback = debug
```

<br/><br/><br/>

### presentation deck

<https://takamitsu-iida.github.io/decks/ansible-role-cisco_show_run/>
