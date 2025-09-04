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

ncs_cli -C -u admin で NSO CLI に入り、下記のようにスケジューラを設定します

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

## サンプルアクションの作成

Linux に戻り、下記のコマンドを用いてパッケージのスケルトンを作成します。

```
cd
cd ncs-run/packages
ncs-make-package --service-skeleton python --action-example custom-action
```

次にパッケージを make します

```
cd custom-action/src
make
```

その後 NSO でパッケージを読み込みます

```
ncs_cli -C -u admin
packages reload
```

## サンプルアクション double のカスタマイズ

YANG の出力を sting に変更後、main.py のアクションロジックを下記のように変更します。

```
output.result = f'{str(input.number)} の 2 倍は {str(input.number * 2)} です。'
```

その後 make し、パッケージを読み込みます。

```
make
ncs_cli -C -u admin
packages reload
```

## Action によるスクリプトの実行

ncs-run/packages/custom-action/src/yang/custom-action.yang ファイルを編集し、サンプルアクションに下記を追加します。

```
    tailf:action script {
      tailf:exec "action.sh";
      output {
        leaf result {
          type string;
        }
      }
    }
```

上記のあと忘れずに make しておきます。

```
cd /home/cisco/ncs-run/packages/custom-action/src
make
```

その後、スクリプト配置のため、パスの通ったフォルダに移動します。

```
cd $NCS_DIR/bin
```

その後、下記をタイプすると入力待ちになります。

```
cat > action.sh
```

その状態で下記をコピペしてください。

```
#!/usr/bin/bash
echo result '"'
vmstat
echo '"'

```

その後 CTRL+D を押すことで action.sh が $NCS_DIR/bin に配置されます。

最後に実行属性を付与します。

```
chmod +x action.sh
```

下記を実行し、スクリプトが動作することを確認します。

```
./action.sh
```

最後に packages reload で完了です。

```
ncs_cli -C -u admin
packages reload
```


### curl での確認

Linux で下記を実行してください。

```
curl -u admin:admin http://198.18.134.27:8080/restconf/operations/custom-action:action/script -X POST
```

## 余裕のある人向け

Linux に戻りパッケージを clone します。

```
cd /home/cisco/ncs-run/packages
git clone https://github.com/mekawaba/NSO-custom-action-2.git
```

その後、一度 make clean, make all を実行します。

```
make clean
make all
```

その後パッケージを読み込みます。

```
ncs_cli -C -u admin
packages reload
```


Linux に戻り、 IOS および ASA のテンプレートを読み込むためにフォルダを移動します。
その後 NSO CLI に入ります。

```
cd /home/cisco/NSO-6.5-free
ncs_cli -C -u admin
```

次にテンプレートを読み込みます。

```
config t
load merge devices-ios-asa.xml
commit
end
```

その後、デバイスが sync できることを確認します。

```
devices sync-from
```

その後、devices/device が拡張されていることを確認できたら完了です。

ハンズオンは以上です。お疲れ様でした。





