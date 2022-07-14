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

# インストール ステップ
1. Free Trialサイトにアクセスし、hub.juniper.netのアカウント取得

    [Free Trial](https://www.juniper.net/jp/ja/forms/cn2-free-trial.html)

2. Kubernetesのインストール

    - Kubernetesのインストール方法に制限はありません。

    - Kubernetesのインストールが初めての方は、以下を参考にしてください

    [Kubernetesインストール(JuniperSite)](https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-k8s-install-and-lcm/topics/task/cn-cloud-native-k8s-create-kubernetes-cluster.html)
    
    [Kubernetesインストール(kubernetes.io)](https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/)

  - Contrail Requirements

    [サポートOS、Kubernetes Version](https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-k8s-install-and-lcm/topics/reference/cn-cloud-native-system-requirements.html)

    [最小スペック](https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-k8s-install-and-lcm/topics/reference/cn-cloud-native-system-requirements.html)
    
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

