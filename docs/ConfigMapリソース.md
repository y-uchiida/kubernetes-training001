# ConfigMap リソース

`ConfigMap` は、Pod に設定ファイルや環境変数を提供するためのリソース  
`Configmap` の設定ファイルを作成し、それをPod のコンテナ(`spec.containers.envFrom`)で読み込ませることで、Pod 共通利用するべきデータを提供することができる  
または、Podのコンテナに読み取り専用ボリュームとして追加(`spec.volumes`) することで、Pod に設定ファイルを提供することもできる

`ConfigMap` は設定内容の暗号化は行わないため、機密情報を設定する場合は `Secret` リソースを使用する

## Configmap の設定ファイルの作成

以下のコマンドで、ひな型を作成できる

```bash
kubectl create configmap configmap-test001 --dry-run=client -o yaml > configmap-test001.yml
```

## ConfigMap リソースの追加

`ConfigMap` を追加するには、`kubectl create configmap` コマンドを使用する

```bash
kubectl create configmap configmap-from-command
```

コマンドの引数で、`ConfigMap` の設定ファイルを指定することもできる

```bash
kubectl create configmap configmap-from-command --from-file=configmap-test001.yml
```

設定ファイルから `ConfigMap` を作成する場合は、以下のコマンドを使用する

```bash
kubectl create -f configmap-test001.yaml
```

## ConfigMap リソースの状態確認

```bash
kubectl get configmap
```

ファイルを指定して表示する場合は、`kubectl get configmap` コマンドの引数にファイル名を指定する

```bash
kubectl get configmap configmap-test001
```

表示する`ConfigMap` のリソース名を指定することもできる

```bash
kubectl get configmap configmap-from-command
```

## Configmap リソースの詳細確認

指定の`ConfigMap` に設定された環境変数について詳細を確認するには、`kubectl describe configmap` コマンドを使用する

```bash
kubectl describe configmap configmap-from-command
```

## Configmap の環境変数の変更

`ConfigMap` の環境変数を変更するには、`kubectl edit configmap` コマンドを使用する

```bash
kubectl edit configmap configmap-from-command
```

`ConfigMap` のyaml がエディターで開かれるので、ここに設定内容の変更を記述して保存する

## Configmap の削除

`ConfigMap` を削除するには、`kubectl delete configmap` コマンドを使用する

```bash
kubectl delete configmap configmap-from-command
```

設定ファイルから追加した `ConfigMap` を削除する場合は、以下のコマンドを使用する

```bash
kubectl delete -f configmap-test001.yml
```

## Pod でConfigMap を使用する

### Pod でConfigMap を環境変数のとして読み込む例

設定を一部抜粋

```yaml
spec:
  containers:
    - name: test-container
  image: busybox
  envfrom:
    - configMapRef:
      name: configmap-test001
```

### Pod でConfigmap を読み取り専用ボリュームとして追加する例

```yaml
spec:
  containers:
    - name: test-container
  image: busybox
  volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: configmap-test002
```

## busybox イメージの実行結果を確認する

1. busybox の Pod を作成する  

    ```bash
    kubectl apply -f pod-load-configmap-test.yml
    ```

2. Pod のログを確認する

    Pod の設定ファイルで記述した `spec.containers.command` で実行されたコマンドの結果が表示される

    ```bash
    kubectl logs pod-load-configmap
    ```

3. Pod にログインして、環境変数を確認する

    対象のPod がRunnning 状態であれば、以下のコマンドでログインできる  
    今回は busybox イメージで実行コマンドに `sleep 600` を含めているので、Podの追加後10分間は接続可能

    ```bash
    kubectl exec -it pod-load-configmap -- /bin/sh
    ```

    ```bash
    echo $TEST_ENV
    ```

    ```bash
    cat /etc/config/data.csv
    ```
