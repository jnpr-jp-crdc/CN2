# サポート構成
- Single Kubernetes Cluster (Shared Network)
  - Kubernetes/Contrail共に共通のInterfaceを使用
  ![Single-Shared-Network](https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-k8s-install-and-lcm/images/jn-000286.png)
- Single Kubernetes Cluster (Multi Network)
  - KubernetesとContrailの通信を分離
  ![Single-Multi-Network](https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-k8s-install-and-lcm/images/jn-000356.png)
- Multi Kubernetes Cluster
  - 複数のKubernetes Clusterを1つのContrailで管理
  - Center ClusterにのみContrail Controllerをデプロイし、他Clusterにはデプロイしない
  - Center ClusterのContrail Controllerから他ClusterのvRouterを管理
  ![Multi-Cluster](https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-k8s-install-and-lcm/images/jn-000287.png)

# Contrail Requirements
[サポートOS、Kubernetes Version](https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-k8s-install-and-lcm/topics/reference/cn-cloud-native-system-requirements.html)

[最小スペック](https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-k8s-install-and-lcm/topics/reference/cn-cloud-native-system-requirements.html)

# インストール ステップ
1. Free Trialサイトにアクセスし、hub.juniper.netのアカウント取得

    [Free Trial](https://www.juniper.net/jp/ja/forms/cn2-free-trial.html)

2. Kubernetesのインストール

  - Kubernetesのインストール方法に制限はありません。

  - Kubernetesのインストールが初めての方は、以下を参考にしてください

    [Kubernetesインストール(JuniperSite)](https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-k8s-install-and-lcm/topics/task/cn-cloud-native-k8s-create-kubernetes-cluster.html)
    [Kubernetesインストール(kubernetes.io)](https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/)
    
3. 以下のサイトからContrail Deployment ManifestsのDownload

    [Download](https://support.juniper.net/support/downloads/?p=contrail-networking)

    権限のない方はJuniper担当者にお問い合わせください
  
4. Contrail インストール

    “kubectl –f apply deployer.yaml”でContrailがインストールされます。
  
    詳細なステップは以下を参照してください
  
    [Single Kubernetes Cluster (Shared Network)](https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-k8s-install-and-lcm/topics/topic-map/cn-cloud-native-k8s-install-single-cluster-one-net.html)
  
    [Single Kubernetes Cluster (Multi Network)](https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-k8s-install-and-lcm/topics/topic-map/cn-cloud-native-k8s-install-single-cluster-multi-net.html)

    [Multi Kubernetes Cluster](https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-k8s-install-and-lcm/topics/topic-map/cn-cloud-native-k8s-install-multi-cluster-one-net.html)
  
5. Contrail Analytics インストール (Option)

    [Contrail Analytics インストール](https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-k8s-install-and-lcm/topics/topic-map/cn-cloud-native-k8s-install-contrail-analytics.html)

## インストール後のPOD一覧
以下はCN2及びContrail-Analyticsをインストールした初期状態です
```
root@cn2-master1:~/manifests# kubectl get pods -A
NAMESPACE            NAME                                                     READY   STATUS      RESTARTS   AGE
contrail-analytics   alertmanager-prometheus-alertmanager-0                   2/2     Running     0          4m48s
contrail-analytics   ambassador-d865d6c54-hhtbn                               1/1     Running     0          7m15s
contrail-analytics   collector-75fd9d769b-5hvbd                               1/1     Running     1          7m18s
contrail-analytics   dashboard-6fd9fff84b-x27s5                               1/1     Running     0          7m18s
contrail-analytics   fluentbit-9dj6m                                          0/1     Running     0          7m18s
contrail-analytics   fluentbit-cqsrw                                          0/1     Running     0          7m18s
contrail-analytics   fluentbit-lghf9                                          0/1     Running     0          7m18s
contrail-analytics   fluentd-7fdc44944b-96lns                                 1/1     Running     6          7m14s
contrail-analytics   grafana-74f8c467bf-7z5hd                                 3/3     Running     0          7m13s
contrail-analytics   grafana-influxdb-datasource-init-wvzbz                   0/1     Completed   0          7m10s
contrail-analytics   influxdb-0                                               1/1     Running     0          7m10s
contrail-analytics   installutil-2zn7c                                        0/1     Completed   0          5m13s
contrail-analytics   installutil-tg9h8                                        0/1     Completed   0          6m3s
contrail-analytics   introspect-5c576f9946-kgc65                              1/1     Running     0          7m18s
contrail-analytics   kruise-cp-secret-22.2.0-rev0bb9dbbd1-c628j               0/1     Completed   0          7m10s
contrail-analytics   multicluster-proxy-76cbdd4544-mvxx8                      1/1     Running     0          7m18s
contrail-analytics   node-exporter-b8l2r                                      1/1     Running     0          7m18s
contrail-analytics   node-exporter-d6w2j                                      1/1     Running     0          7m18s
contrail-analytics   node-exporter-m6cng                                      1/1     Running     0          7m18s
contrail-analytics   opensearch-0                                             1/1     Running     0          7m10s
contrail-analytics   opensearch-dashboards-78fd5974d7-hlrsw                   1/1     Running     0          7m11s
contrail-analytics   opensearch-dashboards-install-index-pattern-fv9zp        0/1     Completed   0          7m9s
contrail-analytics   portal-59cb59c775-pf8ld                                  1/1     Running     0          7m17s
contrail-analytics   prometheus-operator-664bff78f5-mv5xx                     1/1     Running     0          7m11s
contrail-analytics   prometheus-prometheus-prometheus-0                       2/2     Running     0          4m47s
contrail-analytics   state-metrics-644fbcf74-f5nkv                            1/1     Running     0          7m13s
contrail-analytics   telemetry-operator-controller-manager-6c77486684-rpnjc   2/2     Running     0          7m16s
contrail-deploy      contrail-k8s-deployer-ddc899df5-bzp8x                    1/1     Running     0          6d21h
contrail-system      contrail-k8s-apiserver-868c9b7cc5-jmxf8                  1/1     Running     0          6d21h
contrail-system      contrail-k8s-controller-859548cd6f-ckxx4                 1/1     Running     1          6d21h
contrail             contrail-control-0                                       2/2     Running     0          6d21h
contrail             contrail-k8s-kubemanager-79574f76c7-ckf4j                1/1     Running     2          6d21h
contrail             contrail-vrouter-nodes-njnbv                             4/4     Running     2          6d21h
contrail             contrail-vrouter-nodes-nw6p9                             4/4     Running     2          6d21h
contrail             contrail-vrouter-masters-ft24d                           3/3     Running     0          6d21h
kruise-system        kruise-controller-manager-57769cdd8-44n76                1/1     Running     0          7m13s
kruise-system        kruise-controller-manager-57769cdd8-9vmjv                1/1     Running     0          7m13s
kruise-system        kruise-daemon-78jbd                                      1/1     Running     0          7m18s
kruise-system        kruise-daemon-84tkl                                      1/1     Running     0          7m18s
kruise-system        kruise-daemon-cnr5b                                      1/1     Running     0          7m18s
kube-system          coredns-657959df74-2dtm2                                 1/1     Running     36         29d
kube-system          coredns-657959df74-r2nd7                                 1/1     Running     8          7d23h
kube-system          dns-autoscaler-b5c786945-jk8m7                           1/1     Running     1          29d
kube-system          kube-apiserver-cn2-master1                               1/1     Running     3          29d
kube-system          kube-controller-manager-cn2-master1                      1/1     Running     15         29d
kube-system          kube-proxy-2tf5v                                         1/1     Running     11         29d
kube-system          kube-proxy-64ck2                                         1/1     Running     1          29d
kube-system          kube-proxy-hv7pn                                         1/1     Running     6          29d
kube-system          kube-scheduler-cn2-master1                               1/1     Running     16         29d
kube-system          nginx-proxy-cn2-worker1                                  1/1     Running     11         29d
kube-system          nginx-proxy-cn2-worker2                                  1/1     Running     7          29d
```
