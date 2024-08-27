# Secret リソース

`Secret` は、`ConfigMap` と同様に`Pod` に環境変数を設定するためのリソース  
設定内容を`base64` 形式で保存するため、機密情報を設定する場合に適している

## Secret リソースの設定ファイルの作成

以下のコマンドで、ひな型を作成できる

```bash
kubectl create secret generic secret-test001 --dry-run=client -o yaml > secret-test001.yml
```

ファイルから設定値を読み込んで`Secret` を作成する場合は、以下のように設定する

この例では、`secret-data.csv` というファイルを`base64` エンコードした内容を`Secret` の値として追加している  
キー名はそのまま`secret-data.csv` として設定している

```yaml
kubectl create secret generic secret-test001 --from-file=secret-data.csv=secret-data.csv
```

## Secret リソースの設定項目

Secret リソース特有の項目のうち代表的なもの

- `type` : Secret の種類を指定する
  - `Opaque` : バイナリデータを格納する場合に使用
  - `kubernetes.io/service-account-token` : サービスアカウントのトークンを格納する場合に使用
  - `kubernetes.io/dockercfg` : Docker レジストリの認証情報を格納する場合に使用
- `data` : Secret に格納するデータを指定する
  - `key: value` の形式で指定する
  - `value` には、base64 エンコードされた値を指定する
- `stringData` : `data` と同様にデータを指定するが、base64 エンコードする必要がない
  - `key: value` の形式で指定する
  - 設定ファイルに内容が平文で乗るので、取り扱いに注意が必要

## Secret リソースの追加

`Secret` を追加するには、`kubectl create secret` コマンドを使用する

```bash
kubectl create secret generic secret-from-command
```

コマンドから`Secret` を追加する場合は、 `--from-literal` オプションを使用して設定値を記載することもできる  

自動でbase64 エンコードされるが、コマンド履歴には平文で残るので、機密情報を設定する場合は注意が必要

```bash
kubectl create secret generic secret-from-command --from-literal=SECRET=secret001
```

`--from-literal` で、複数の環境変数を設定することもできる

```bash
kubectl create secret generic secret-from-command --from-literal=SECRET=secret001 --from-literal=SECRET2=secret002
```

オプションで、設定ファイルから`Secret` リソースを作成する場合は、以下のコマンドを使用する

```bash
kubectl create -f secret-test001.yaml
```

## Secret リソースの状態確認

```bash
kubectl get secret
```

`kubectl get secret` の引数にリソース名を指定することもできる

```bash
kubectl get secret secret-test001
```

ファイル名を指定して表示する場合は、`kubectl get secret` コマンドの引数にファイル名を指定する

```bash
kubectl get -f secret-test001.yaml 
```

## Secret リソースの詳細確認

指定の`Secret` に設定された環境変数について詳細を確認するには、`kubectl describe secret` コマンドを使用する

```bash
kubectl describe secret secret-from-command
```

## Secret の環境変数の変更

`Secret` に設定された環境変数を変更するには、`kubectl edit secret` コマンドを使用する

```bash
kubectl edit secret secret-from-command
```

`kubectl edit secret` コマンドを実行すると、エディタが起動するので、環境変数の値を変更する

## Secret リソースの削除

`Secret` を削除するには、`kubectl delete secret` コマンドを使用する

```bash
kubectl delete secret secret-from-command
```

設定ファイルから追加した`Secret` を削除する場合は、以下のコマンドを使用する

```bash
kubectl delete -f secret-test001.yaml
```

## Secret の暗号化

`Secret` は、`base64` エンコードされているだけで、暗号化されていない  
`base64` エンコードは、データを可読性のある形式に変換するだけで、セキュリティを提供するものではない  
`Secret` に機密情報を設定する場合は、暗号化されたファイルを使用するか、`HashiCorp Vault` などのシークレット管理ツールを使用することを検討する

## Pod で Secret を使用する

### Pod でSecret を環境変数として使用する例

`Pod` で`Secret` を使用するには、`Pod` の`spec.containers.envFrom` に`Secret` を指定する

```yaml
spec:
  image: busybox
  containers:
  - name: test-container
    envFrom:
      - secretRef:
        name: secret-test001
```

### Pod でSecret を読み取り専用ボリュームとして使用する例

`Pod` で`Secret` を読み取り専用ボリュームとして使用するには、`Pod` の`spec.volumes` に`Secret` を指定する  

さらに、`spec.containers.volumeMounts` でマウントしたSecret 名を指定し、`mountPath` でマウントするパスを指定する

```yaml
spec:
  volumes:
    - name: secret-volume
     secret:
        secretName: secret-test002
  image: busybox
  containers:
    - name: test-container
      volumeMounts:
        - name: secret-volume
          mountPath: /etc/secret-volume
```

## 補足: base64 のエンコードとでコード

### base64 エンコード

```bash
echo -n "secret" | base64
```

### base64 デコード

```bash
echo -n "c2VjcmV0" | base64 -d
```
