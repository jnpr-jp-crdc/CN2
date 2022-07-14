# Cloud Native Contrail Networking(CN2)概要
CN2は、Kubernetes, OpenShiftで利用可能なCNI(Container Network interface)です。

EVPN/VXLAN, BGP/MPLSをベースとしたアーキテクチャとなり、Kubernetes/OpenShift Cluster内はもちろん、Cluster間接、Cluster外のDataCenterGatewayRouterとの接続やVPN網接続、など柔軟なネットワーク接続を提供します。

CN2特徴

・ハイブリッド・マルチクラウド環境での一貫したネットワーク

・マルチクラスターフェデレーション - 単一のContrail controllerで複数のKubernetes Clusterを管理

・Kubernetes / OpenStack ハイブリッドSDN *future support

・ハイパフォーマンス - DPDK/SmartNIC対応

・GitOpsによるCI/CDパイプライン

# Free Trial
[申請フォーム](https://www.juniper.net/jp/ja/forms/cn2-free-trial.html)

申請後、CN2インストールに必要なhub.juniper.netのアカウントの発行及び、deployer.yamlがダウンロード可能となります。

# Contrail Networking歴史
Juniper Networks社のContrail Networkingは、2012年にContrail Systems社を買収し、リリース以降、パブリッククラウド基盤、プライベートクラウド基盤、キャリアクラウド基盤などさまざまな環境で利用されており、OpenStack分野では最も使われているネットワークプラットフォームとなっています。[OpenContrail Ranked #1 Again!](https://forums.juniper.net/t5/Service-Provider-Transformation/OpenContrail-Ranked-1-Again-Juniper-Brings-Its-A-Game-with-a/ba-p/290851)

2018年にはオープンソースで提供していた「OpenContrail」が、「[Tungsten Fabric](https://tungsten.io/)」と名称を変え、Linux Foundationによってホストされ 、よりニュートラルなオープンソースプロジェクトとなっています。

2019年にはContrail Enterprise Multicloudを発表し、Kubernetes/OpenShift/OpenStackのOverlay接続に加え、DataCenter Fabric(Underlay)も一元管理できるようになりました。

2022年、Contrail NetworkingはCloud Native Contrail Networking(CN2)と名称を変え、Cloud Native環境とより親和性を上げるためにアーキテクチャの変更が行われました。

# チュートリアル
0. CN2 Install
1. Namespace
 1. Isolated Namespace
 2. Pod作成(default-podnetwork)
 3. Fabric SNAT
 4. Fabric Forwarding
2. Virtual Network
 1. Network Attachment Definition
 2. Pod作成(Contrail Virtual Network)
 3. Pod作成(Contrail Multi Virtual Network)
 4. Subnet
 5. Virtual Network
3. Virtual Netowrk Router
 1. Mesh Case1
 2. Mesh Case2
 3. Mesh Case3
 4. Hub&Spoke Case1
 5. Hub&Spoke Case2
 6. Mesh&Hub&Spoke
4. Route Leak
 1. Same Route Target
 2. Router Target Import/Export
5. BGPaaS
6. VLAN Sub Interface
7. Allowed Address Pair (VIP)
8. Network Policy
9. 外部Router接続
 1. Fabric Forwarding
 2. Virtual Network
10. ClusterIP
11. NodePort
12. Load Balancer
13. Ingress
14. KubeVirt
15. DPDK
16. Port base Mirroring
17. Contrail Status
18. Analytics
19. Lens Extention
20. Contrail Pipeline
21. Multi-Cluster

