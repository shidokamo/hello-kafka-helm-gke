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

## デプロイ：外部IPを使わない場合
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

### Kafka のテスト
まず、kafka の pod 名とポート番号を確認する。

```
$ kubectl get pod,svc -n kafka
NAME                       READY   STATUS    RESTARTS   AGE
pod/my-kafka-0             1/1     Running   1          50m
pod/my-kafka-zookeeper-0   1/1     Running   0          50m
pod/my-kafka-zookeeper-1   1/1     Running   0          49m
pod/my-kafka-zookeeper-2   1/1     Running   0          48m

NAME                                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
service/my-kafka                      ClusterIP   10.27.240.3    <none>        9092/TCP                     50m
service/my-kafka-headless             ClusterIP   None           <none>        9092/TCP                     50m
service/my-kafka-zookeeper            ClusterIP   10.27.245.36   <none>        2181/TCP                     50m
service/my-kafka-zookeeper-headless   ClusterIP   None           <none>        2181/TCP,3888/TCP,2888/TCP   50m
```

`my-kafka-0` の 9092 番ポートを localhost にフォワードする。

```
kubectl -n kafka port-forward my-kafka-0 9092
```

`kafkacat` というユーティリティツールがなければインストールする。

```
sudo apt-get install -y kafkacat
```

なんらかのログファイル（一定間隔で追記されているものが望ましい）をトピックに投げる。
ログローテートを使っている可能性も考慮して `tail` には、`-F` オプションをつけるのが望ましい。

```
tail -F ./your_appending.log | kafkacat -P -t your_topic_name -b localhost:9092
```

トピックをサブスクライブする。うまくデプロイができていれば、メッセージを受信する様子が確認できます。

```
kafkacat -C -t your_topic_name -b localhost:9092
```

## デプロイ：LoadBalancer で外部IPへ公開する場合
リージョン IP アドレスを作成する
```
gcloud compute addresses create YOUR_IP_NAME --region us-central1
```

リージョン IP アドレスを確認する
```
gcloud compute addresses describe YOUR_IP_NAME --region us-central1
```

デプロイを更新
```
helm install my-kafka-lb -f values.yaml --set external.loadBalancerIP=YOUR_IP_ADDRESS incubator/kafka
```

## デプロイ：NodPort で外部IPへ公開する場合（GCP推奨）

```
  --set external.type=NodePort \
  --set external.annotations."kubernetes\.io/ingress\.global-static-ip-name"=YOUR_IP_NAME
```
