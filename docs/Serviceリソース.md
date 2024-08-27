# Service リソース

`Service` は、サービスのエンドポイントを公開するための方法を定義するリソース  
複数のPodをまとめて、一つのエンドポイントとして公開することができる  
Pod に対する負荷分散やDNS名の解決を行うために使用される  

Pod はローリングアップデートやスケーリングによって差し替えされるため、永続的なリソースではない  
Pod のIP アドレスに対して直接アクセスするようになっていると、利用できなくなる可能性がある  
そのため、`Service` リソースを使用して、Pod に対して一定のエンドポイントを提供する  
サービスに対してアクセスすることで、Pod の IP アドレスが変更されても、継続的にアクセスすることができる  

## Service リソースの作成

`Service` リソースを作成するには、`kubectl create` コマンドを使用する

```bash
kubectl create service clusterip my-service-from-command --tcp=80:80
```

`--tcp` オプションには、`<ポート番号>:<ターゲットポート番号>` の形式で指定する  
`<ポート番号>` は、サービスのポート番号を指定する  
`<ターゲットポート番号>` は、Pod のポート番号を指定する

`--dry-run=client` オプションを指定することで、リソースを作成せずに、設定ファイルを作成することができる

```bash
kubectl create service clusterip service-test001 --tcp=80:80 --dry-run=client -o yaml > service-test001.yml
```

設定ファイルから`Service` リソースを作成する場合は、以下のように設定する

```bash
kubectl apply -f service-test001.yml
```

## Service リソースの設定項目

`Service` リソース特有の項目のうち代表的なもの

- `spec.selector` : サービスが対象とするPod を指定する
  - `key: value` の形式で指定する
  - Pod のラベルセレクタを指定する
- `spec.ports` : サービスが公開するポートを指定する
  - `name` : サービスポートの名前を指定する
  - `port` : サービスポートのポート番号を指定する
  - `targetPort` : ターゲットポートのポート番号を指定する
  - `protocol` : プロトコルを指定する
- `spec.type` : サービスのタイプを指定する
  - `ClusterIP` : クラスタ内からのみアクセス可能なサービス
  - `NodePort` : クラスタ外からアクセス可能なサービス
  - `LoadBalancer` : クラスタ外からアクセス可能なサービス
- `spec.clusterIP` : クラスタ内のIP アドレスを指定する

## Service リソースの削除

`Service` リソースを削除するには、`kubectl delete` コマンドを使用する

```bash
kubectl delete service service-test001
```

設定ファイルから`Service` リソースを削除する場合は、以下のように設定する

```bash
kubectl delete -f service-test001.yml
```

## Service リソースの確認

`Service` リソースを確認するには、`kubectl get` コマンドを使用する

```bash
kubectl get service
```

`Service` リソースの詳細情報を確認するには、`kubectl describe` コマンドを使用する

```bash
kubectl describe service service-test001
```

## Service リソースの更新

`Service` リソースを更新するには、`kubectl apply` コマンドを使用する

```bash
kubectl apply -f service-test001.yml
```

## Service リソースのポートフォワーディング

`Service` リソースのポートフォワーディングを行うには、`kubectl port-forward` コマンドを使用する

```bash
kubectl port-forward service/service-test001 8080:80
```

ポート指定の順番は、`ローカルポート:リモートポート` で指定する  
今回は、ローカルポート8080 に、Service リソースのリモートポート80 を割り当てる

`service/` はショートハンドとして`svc/` と記述できる

```bash
kubectl port-forward svc/service-test001 8080:80
```

<http://localhost:8080> にアクセスすることで、`Service` リソースにアクセスすることができる

## Service リソースの挙動確認

1. 管理対象のDeployment を追加する

    ```bash
    kubectl apply -f deployment-service-test001.yml
    ```

2. 管理対象ではないDeployment も追加しておく

    ```bash
    kubectl apply -f deployment-test001.yml
    ```

3. 起動しているPod の確認

    Pod に設定されているラベルを表示するには、`--show-labels` オプションを指定する  
    Pod に割り当てられているIP アドレスを表示するには、`-o wide` オプションを指定する

    ```bash
    kubectl get pod --show-labels -o wide
    ```

    または、ラベルを指定してPodを絞り込んで表示することもできる

    ```bash
    kubectl get pod --selector app=deployment-service-test001 --show-labels -o wide
    ```

    以下のように表示される

    ```plaintext
    NAME                                         READY   STATUS    RESTARTS   AGE     IP          NODE             NOMINATED NODE   READINESS GATES   LABELS
    deployment-service-test001-d8b4df7cf-kxvvz   1/1     Running   0          26s     10.1.0.65   docker-desktop   <none>           <none>            app=deployment-service-test001,pod-template-hash=d8b4df7cf
    deployment-service-test001-d8b4df7cf-xfrsc   1/1     Running   0          2m30s   10.1.0.61   docker-desktop   <none>           <none>            app=deployment-service-test001,pod-template-hash=d8b4df7cf
    ```

    この場合、`app=deployment-service-test001` というラベルが設定されているPod が2つ起動していて、IPはそれぞれ `10.1.0.65` と `10.1.0.61` である

4. Service リソースを追加する

    ```bash
    kubectl apply -f service-test001.yml
    ```

5. Service リソースとEndpoint リソースの確認

    ```bash
    kubectl get service
    ```

    Service リソースを作成すると、同時にEndpoint リソースも作成される

    ```bash
    kubectl get endpoints service-test001
    ```

    以下のように表示される

    ```plaintext
    NAME              ENDPOINTS                   AGE
    service-test001   10.1.0.61:80,10.1.0.65:80   110s
    ```

    `ENDPOINTS` には、Service リソースが指し示すPod のIP アドレスとポート番号が表示される  
    これは、`kubectl get pod -o wide` で表示されたIPアドレスと一致している

6. ポートフォワードを設定する

    ```bash
    kubectl port-forward service service-test001 8080:80
    ```

7. ブラウザでアクセスする

    <http://localhost:8080>

8. Deployment の再起動

    起動中のPod を削除して、Deployment 内のPod のIP アドレスを更新する

    ```bash
    kubectl delete pod --selector app=deployment-service-test001
    ```

    ```bash
    kubectl get pod --selector app=deployment-service-test001 --show-labels -o wide
    ```

    以下のように表示され、IP アドレスが変更されていることが確認できる

    ```plaintext
    NAME                                         READY   STATUS    RESTARTS   AGE   IP          NODE             NOMINATED NODE   READINESS GATES   LABELS
    deployment-service-test001-d8b4df7cf-hgj87   1/1     Running   0          10s   10.1.0.67   docker-desktop   <none>           <none>            app=deployment-service-test001,pod-template-hash=d8b4df7cf
    deployment-service-test001-d8b4df7cf-s947v   1/1     Running   0          10s   10.1.0.66   docker-desktop   <none>           <none>            app=deployment-service-test001,pod-template-hash=d8b4df7cf
    ```

9. Service と Endpoint の確認

    ```bash
    kubectl get endpoints service-test001
    ```

    以下のように表示される

    ```plaintext
    service-test001   10.1.0.66:80,10.1.0.67:80   18m
    ```
