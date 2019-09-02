# Hello Kafka on GKE with Helm
Kafka を GKE にデプロイするサンプルです

* https://github.com/Yolean/kubernetes-kafka
* https://github.com/helm/charts/tree/master/incubator/kafka

上記のレポジトリを元にした小規模なデプロイです。

## 手順
### クラスタの作成
```bash
gcloud container cluster create kafka --disk-size=100 --machine-type=n1-standard-1
```
### Tiller のサービスアカウントの作成
```bash
kubectl apply -f create-helm-service-account.yaml
```

### Tiller のデプロイ
```bash
helm init --history-max 200 --service-account tiller
```

### Tiller の確認
```bash
kubectl get pod -n kube-system | grep tiller
```

### レポジトリの登録
Helm は、stable チャートをデフォルトで登録しているが、Kafka はまだ incubator なので、
レポジトリを登録する。
```bash
helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator
```

### レポジトリの確認
```bash
helm repo list
```

### 名前空間を作る
```bash
kubectl create ns kafka
```

### 標準構成でのデプロイ
```bash
helm install --name my-kafka --namespace kafka incubator/kafka
```

### カスタム構成でのデプロイ
```bash
helm install --name my-kafka --namespace kafka -f values.yaml incubator/kafka
```

### Pod/Service の確認
```bash
kubectl get pod,svc -n kafka
```

### Helm のリリース確認
```bash
helm list
```

### リリースの削除
```bash
helm delete my-kafka
```

### Kafka を使う
