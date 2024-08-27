# ReplicaSet リソース

`ReplicaSet` は、Pod のレプリカを管理するためのワークロードリソース  
指定した数の Pod が常に実行されるように管理する  
Pod のレプリカ数が増減した場合に、Pod の追加や削除を行う

`ReplicaSet` を直接管理することは基本的にはなく、`Deployment` で管理することが一般的  
`Deployment` は `ReplicaSet` を内部で管理しており、`ReplicaSet` は `Deployment` によって自動的に作成される  
`Deploument` には、スケールアウトやローリングアップデートなどの機能があり、`ReplicaSet` はこれらがない  
実際に運用するにあたっては、`Deployment` を利用することが推奨される

## ReplicaSet の設定項目

代表的なものは以下

- `metadata.name` : ReplicaSet の名前
- `spec.replicas` : Pod のレプリカ数。この数だけ Pod が常に実行される
- `spec.selector` : ReplicaSet が管理する Pod を特定するためのラベルセレクタ
- `spec.selector.matchLabels` : ReplicaSet が管理する Pod に適用されるラベル
- `spec.template` : ReplicaSet が管理する Pod のテンプレート。書き方は Pod の設定と同じ
- `spec.template.metadata.labels` : ReplicaSet が管理する Pod に適用されるラベル。`spec.selector.matchLabels` と同じラベルを指定する

## ReplicaSet の追加

```bash
kubectl apply -f replicaset-test001.yml
```

`--dry-run=client -o yaml` オプションをつけることで、実施する内容をyaml形式で確認できる

```bash
kubectl create replicaset -f --dry-run=client -o yaml
```

## ReplicaSet の状態確認

```bash
kubectl get replicaset -n team-b
```

設定ファイル `replicaset-test001.yml` を利用して、そのファイルに含まれる ReplicaSet の状態を確認することができる

```bash
kubectl get -f replicaset-test001.yml
```

## ReplicaSet の更新

設定ファイルの`spec.replicas` を修正して、`apply` コマンドで更新する

```bash
kubectl apply -f replicaset-test001.yml
```

または、`scale` コマンドでレプリカ数を変更することも可能

```bash
kubectl scale replicaset nginx-rs --replicas=4 -n team-b
```

ショートハンド `rs/レプリカセット名` で指定することも可能

```bash
kubectl scale replicaset rs/nginx-rs --replicas=4 -n team-b
```

## ReplicaSet の削除

```bash
kubectl delete -f replicaset-test001.yml
```

または、`delete` コマンドで削除することも可能

```bash
kubectl delete replicaset nginx-rs -n team-b
```
