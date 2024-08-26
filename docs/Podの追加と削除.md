# Podの追加と削除

## Podの追加

設定ファイルのパスを指定して追加・削除することができる

```bash
kubectl create -f pod-test001.yml
```

## Podの削除

```bash
kubectl delete -f pod-test001.yml
```

## Podの状態確認

```bash
kubectl get pods
```

設定ファイルのパスを指定して、そのファイルに含まれるPodの状態を確認することができる

```bash
kubectl get -f pod-test001.yml
```
