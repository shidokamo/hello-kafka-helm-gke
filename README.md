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
