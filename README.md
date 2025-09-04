## 事前準備
### コンテナの確認

```
docker ps
```

### NSO のインストール

```
cd NSO-6.5-free
ansible-playbook nso.yml
```

### NSO のアクセス

```
cd
source NSO-6.5/ncsrc
ncs_cli -C -u admin
```

### WebUI の有効化

VSCode で ncs-run/ncs.conf を編集し、330 行目の下に下記 server-alias を追加

```
<server-alias>198.18.134.27</server-alias>
```

Linux に戻り、NSO の停止と起動を行います

```
cd ncs-run
ncs --stop
ncs
```

WebUI へのアクセス（下記 URL を右クリックで新しい TAB で開いてください）
- http://198.18.134.27:8080/
- username = admin
- password = admin


## Scheduler

スケジューラの設定

```
config t
scheduler task periodic-check-sync
/devices/device[name='xr-1']
check-sync
schedule "*/5 * * * *"
commit
```

確認

```
show scheduler
```

## サンプルアクション double の実行

```
ncs-make-package --service-skeleton python --action-example custom-action
```

## サンプルアクション double のカスタマイズ

YANG の出力を sting に変更後、main.py のアクションロジックを下記のように変更します。

```
output.result = f'{str(input.number)} の 2 倍は {str(input.number * 2)} です。'
```


## Action によるスクリプトの実行

サンプルアクションに下記を追加します。

```
    tailf:action script {
      tailf: exec "action.sh"
      output result {
        type string;
      }
    }
```

以下のスクリプトを action.sh という名前で $NCS_DIR/bin に配置します。

```
#!/usr/bin/bash
echo result '"'
vmstat
echo '"'
```

curl での確認

```
curl -u admin:admin http://198.18.134.27:8080/restconf/operations/custom-action:action/script -X POST
```

## 余裕のある人向け

```
git clone https://github.com/mekawaba/NSO-custom-action-2.git
```





