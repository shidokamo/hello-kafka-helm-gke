# Hello Kafka on GKE
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
