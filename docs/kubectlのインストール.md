# kubectl のインストール

以下の公式ページの手順に沿ってインストールする

<https://kubernetes.io/ja/docs/tasks/tools/install-kubectl-linux/>

`ネイティブなパッケージマネージャーを使用してインストールする` の手順で実施した

パッケージの更新について、注意書きがある

> kubectlを他のマイナーリリースにアップグレードするためには、apt-get updateとapt-get upgradeを実行する前に、/etc/apt/sources.list.d/kubernetes.listの中のバージョンを上げる必要があります。 この手順についてはChanging The Kubernetes Package Repositoryに詳細が記載されています。

`/etc/apt/sources.list.d/kubernetes.list` は、以下のようになっている

```plaintext
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /
```

URLの中の `v1.30` の部分を差し替えることで、インストールするバーションを変更できる
