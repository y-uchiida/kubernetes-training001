# Deployment リソース

Deployment リソースは、Pod のデプロイとスケーリングを管理するリソース  
ReplicaSet と Pod を組み合わせたものと考えることができる  

定期実行すジョブ以外は基本的にDeployment を利用するくらい使用頻度が高い

## Deployment の設定項目

Deployment 特有のフィールド

- `spec.strategy` : デプロイのルールセットのようなもの
- `spec.strategy.type` : デプロイ時のPodの操作方針。`Recreate` または `RollingUpdate` を指定
  - `Recreate` : 一度に全てのPodを新しいものに置き換える。ダウンタイムが発生する
  - `RollingUpdate` : 旧バージョンのPodを残しつつ、一部分ずつ新しいものに置き換える
  - デフォルトは `RollingUpdate`
- `spec.strategy.rollingUpdate` : `RollingUpdate` でのデプロイ時の設定
  - `maxSurge` : 一度に増やすPodの数
  - `maxUnavailable` : 一度に減らすPodの数
- `spec.strategy.revisionHistory`: 履歴を保持するリビジョン数
  - 過去のバージョンとして保持しておくReplicaSetの世代数。この数までロールバックできる
  - デフォルトは `10`
- `spec.paused` : デプロイの一時停止
  - `true` に設定すると、新しいReplicaSetの作成を一時停止
- `spec.progressDeadlineSeconds` : デプロイの進捗状況のタイムアウト。この時間を超えるとデプロイ失敗と判定される
  - デフォルトは `600` 秒

## Deployment の追加

```bash
kubectl apply -f deployment-test001.yml
```

## Deployment の状態確認

```bash
kubectl get deployment
```

設定ファイル `deployment-test001.yml` を利用して、そのファイルに含まれる Deployment の状態を確認することができる

```bash
kubectl get -f deployment-test001.yml
```

## Deployment の更新

設定ファイルを修正して、`apply` コマンドで更新する

```bash
kubectl apply -f deployment-test001.yml
```

または、 `edit` コマンドで編集して更新することも可能

```bash
kubectl edit deployment nginx-deployment
```

image のバージョンを変更する場合は、`set image` コマンドを利用する  

`コンテナ名:イメージ名` の形式で指定する

```bash
kubectl set image deployment/nginx-deployment nginx-dep=nginx:1.15
```

以下コマンドで Pod の様子を逐次表示しておくと動きがわかりやすい

```bash
watch kubectl get pods
```

## Deployment のロールバック

### リビジョンの表示

```bash
kubectl rollout history deployment/nginx-deployment
```

以下のように出力される

```plaintext
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>
```

### ロールバック

ひとつ前のリビジョンにロールバックする

```bash
kubectl rollout undo deployment/nginx-deployment
```

ロールバックするリビジョン番号を指定してロールバックする

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

これを行うと、リビジョン番号 2 の内容をもとに Deployment が更新される

この後、以下のコマンドでリビジョンの履歴を確認すると、ロールバックしたリビジョンが追加されている

```bash
kubectl rollout history deployment/nginx-deployment
```

結果は以下のようになる

```plaintext
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
3         <none>
4         <none>
```

ロールバックしたリビジョン番号 2 が消えて、新しいリビジョン番号 4 が追加されている

`ReplicaSet` は追加されず、古いバージョンのものに巻き戻される  

## Deployment の削除

Deployment のリソース名を指定して削除する場合は以下のコマンド

```bash
kubectl delete deployment nginx-deployment
```

設定ファイル `deployment-test001.yml` を利用して削除する場合は以下のコマンド

```bash
kubectl delete -f deployment-test001.yml
```
