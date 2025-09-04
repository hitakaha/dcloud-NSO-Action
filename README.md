# 2025 年 9 月 29 日 NSO サクセスコミュニティハンズオンシナリオ

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

VSCode で ncsrun/ncs.conf を編集し、330 行目の下に下記 server-alias を追加

```
<server-alias>198.18.134.27</server-alias>
```

NSO の停止と起動

```
cd ncsrun
ncs --stop
ncs
```

WebUI へのアクセス
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
