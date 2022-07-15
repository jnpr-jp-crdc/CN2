# Analytics
## Introspection
- Introspectionにより、Contrailのより詳細なDebuggingが可能
- 事前にContrail-Analyticsをインストールしておく必要がある

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/Introspection.png">

### Introspection アクセス方法
1. Google Chrome の ModHeader Extentionをインストール
2. K8S Masterで以下を実行し、TokenをGet
```
SECRET_NAME=$(kubectl get serviceaccount introspect -n contrail-analytics -o jsonpath='{.secrets[0].name}'); TOKEN=$(kubectl get secret $SECRET_NAME -n contrail-analytics -o jsonpath='{.data.token}' | base64 --decode); echo $TOKEN
```
3. ModHeaderに以下を設定
<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/ModHeader.png" width="50%">

4. https://K8S_API_ServerIP:6443/apis/introspect.k8s.io/v1beta1/index にアクセス

## Grafana / Prometheus
- Grafanaにて各種リソース、トラヒックの使用状況のモニタリングが可能
- 事前にContrail-Analyticsをインストールしておく必要がある
<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/Grafana1.png">
<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/Grafana2.png">

### Grafana アクセス方法
1. Port Forward設定 
```
kubectl port-forward --address K8S_MASTER_IP service/grafana -n contrail-analytics 80:80
```
2. http://K8S_MASTER_IPにアクセス (User:admin, Pass:prom-operator)

## vRouter Session Analytics
- vRouter Agentは、user visible entities (UVEs) 及び traffic information (session)をコレクターにExport
- InfluxDBにクエリすることで以下の情報取得が可能 
<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/vRouterSession2.png">
<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/vRouterSession.png">

### vRouter Session Analytics 設定方法
1. Fluentdのcluster_ip確認
```
kubectl get svc -n contrail-analytics | grep fluentd-loadbalancer
```
2. vRouter Agentのコレクタ設定(CLUSTER_IP箇所編集)
```
kubectl -n contrail patch vrouter contrail-vrouter-masters --type=merge -p '{"spec":{"agent":{"default":{"collectors":[”CLUSTER_IP:24224"]}}}}'
kubectl -n contrail patch vrouter contrail-vrouter-nodes --type=merge -p '{"spec":{"agent":{"default":{"collectors":[”CLUSTER_IP:24224"]}}}}'
kubectl -n contrail patch gvc default-global-vrouter-config --type=merge -p '{"spec":{"flowExportRate": 10000}}'
```
3. InfluxDBのPassword取得
```
echo $(kubectl -n contrail-analytics get secret influxdb-auth -o json | jq -M '.data."admin-password"' | cut -f2 -d\" | base64 -d)
```
4. PortForward設定
```
kubectl port-forward --address K8S_MASTER_IP pods/influxdb-0 -n contrail-analytics 8086:8086
```
5. InfluxDBへアクセス
http://K8S_MASTER_IP:8086  (User:admin, Pass:上記3のPass)

6. LogLevel変更(Option)
```
kubectl -n contrail-analytics edit cm collector

data:
  config.yaml: |-
    log:
      caller: true
      level:
        default: debug
```
