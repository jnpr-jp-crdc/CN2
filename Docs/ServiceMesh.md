# Service Mesh
CN2はIstio, Linkerd, Consulと共存が可能となり、CN2環境でサービスメッシュを利用できます。

各PODのSidecarProxy(Envoy)がL4-L7のルーティングやロードバランシングを行うのに対し、CN2はSidecarProxyに対してL2-L4の接続を提供します。

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/ServiceMesh.png" width="90%">

## Istio Install
最新版のistioctlをダウンロード
```
curl -L https://istio.io/downloadIstio | sh -
cd istio-X.X.X
export PATH=$PWD/bin:$PATH
```

istioctlのVersion確認
```
# istioctl version
no running Istio pods in "istio-system"
1.14.2
```

Kubernetesクラスタ環境のIstioインストール条件を満たしているか確認
```
# istioctl x precheck
✔ No issues found when checking the cluster. Istio is safe to install or upgrade!
  To get started, check out https://istio.io/latest/docs/setup/getting-started/
```

Istioインストールオプションの確認
```
# istioctl profile list
Istio configuration profiles:
    default
    demo
    empty
    external
    minimal
    openshift
    preview
    remote
```

チュートリアル実行用の"demo"を使用してIstioをインストール
```
# istioctl install --set profile=demo -y
✔ Istio core installed
✔ Istiod installed
✔ Ingress gateways installed
✔ Egress gateways installed
✔ Installation complete                                                                                                                                                                                                                                                         Making this installation the default for injection and validation.
2022-07-27T07:52:16.374170Z	error	klog	couldn't get resource list for introspect.k8s.io/v1beta1: Got empty response for: introspect.k8s.io/v1beta1

Thank you for installing Istio 1.14.  Please take a few minutes to tell us about your install/upgrade experience!  https://forms.gle/yEtCbt45FZ3VoDT5A
```

Istioリソースの確認
```
# kubectl get all -n istio-system
NAME                                        READY   STATUS    RESTARTS   AGE
pod/istio-egressgateway-5c7666b8-9jvrs      1/1     Running   0          51s
pod/istio-ingressgateway-65468675b6-tlf9j   1/1     Running   0          51s
pod/istiod-7bc6dcddcb-pc54m                 1/1     Running   0          74s

NAME                           TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                      AGE
service/istio-egressgateway    ClusterIP      10.233.3.90     <none>        80/TCP,443/TCP                                                               50s
service/istio-ingressgateway   LoadBalancer   10.233.27.221   100.0.0.4     15021:31954/TCP,80:31522/TCP,443:30940/TCP,31400:32295/TCP,15443:32732/TCP   49s
service/istiod                 ClusterIP      10.233.22.28    <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP                                        73s

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/istio-egressgateway    1/1     1            1           51s
deployment.apps/istio-ingressgateway   1/1     1            1           51s
deployment.apps/istiod                 1/1     1            1           74s

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/istio-egressgateway-5c7666b8      1         1         1       51s
replicaset.apps/istio-ingressgateway-65468675b6   1         1         1       51s
replicaset.apps/istiod-7bc6dcddcb                 1         1         1       74s
```
※default-externalnetworkを設定していれば、Istio IngressGatewayにExternalIPが付与される

※default-externalnetworkの設定は[LoadBalancer](https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Loadbalancer.md)を参照


インストールしたIstioの正常性確認
```
root@cn2-master1:~/istio-1.14.2# istioctl verify-install
1 Istio control planes detected, checking --revision "default" only
2022-07-27T08:06:12.868104Z	error	klog	couldn't get resource list for introspect.k8s.io/v1beta1: Got empty response for: introspect.k8s.io/v1beta1
✔ Deployment: istio-ingressgateway.istio-system checked successfully
--- snip ---
Checked 15 custom resource definitions
Checked 3 Istio Deployments
✔ Istio is installed and verified successfully
```

## サンプルアプリ(Bookinfo)のデプロイ
### Isolated Namespace作成
[Isolated Namespace Sample](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/IsolatedNamespace.yaml)

### Bookinfoデプロイ
```
kubectl apply -f ./samples/bookinfo/platform/kube/bookinfo.yaml -n ns1
```

デプロイ後のPOD, Serviceの状態は以下となります。
```
# kubectl get all -n ns1
NAME                                  READY   STATUS    RESTARTS   AGE
pod/details-v1-7d88846999-hhstb       1/1     Running   0          2m1s
pod/productpage-v1-7795568889-l5r9x   1/1     Running   0          118s
pod/ratings-v1-754f9c4975-hj6v2       1/1     Running   0          2m1s
pod/reviews-v1-55b668fc65-kb2s6       1/1     Running   0          2m
pod/reviews-v2-858f99c99-zwxz7        1/1     Running   0          119s
pod/reviews-v3-7886dd86b9-4rh77       1/1     Running   0          119s

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/details       ClusterIP   10.233.42.222   <none>        9080/TCP   2m2s
service/productpage   ClusterIP   10.233.32.254   <none>        9080/TCP   119s
service/ratings       ClusterIP   10.233.6.141    <none>        9080/TCP   2m1s
service/reviews       ClusterIP   10.233.21.245   <none>        9080/TCP   2m1s

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/details-v1       1/1     1            1           2m1s
deployment.apps/productpage-v1   1/1     1            1           118s
deployment.apps/ratings-v1       1/1     1            1           2m1s
deployment.apps/reviews-v1       1/1     1            1           2m
deployment.apps/reviews-v2       1/1     1            1           119s
deployment.apps/reviews-v3       1/1     1            1           119s

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/details-v1-7d88846999       1         1         1       2m1s
replicaset.apps/productpage-v1-7795568889   1         1         1       118s
replicaset.apps/ratings-v1-754f9c4975       1         1         1       2m1s
replicaset.apps/reviews-v1-55b668fc65       1         1         1       2m
replicaset.apps/reviews-v2-858f99c99        1         1         1       119s
replicaset.apps/reviews-v3-7886dd86b9       1         1         1       119s
```

デプロイ後の構成は以下となります。

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/bookinfo1.png" width="90%">

### サイドカープロキシ(Envoy)の挿入
サイドカープロキシを挿入したManifestsの作成
```
# istioctl kube-inject -f ./samples/bookinfo/platform/kube/bookinfo.yaml > ./samples/bookinfo/platform/kube/bookinfo2.yaml
```
サイドカープロキシのデプロイ
```
# kubectl apply -f ./samples/bookinfo/platform/kube/bookinfo2.yaml -n ns1
```

デプロイ後のPODの状態は以下となり、Readyが2/2となり、サイドカープロキシがデプロイされていることを確認
```
# kubectl get pods -n ns1
NAME                              READY   STATUS    RESTARTS   AGE
details-v1-66d8f545cb-vrvv9       2/2     Running   0          84s
productpage-v1-74fbdcdf7c-jbx28   2/2     Running   0          83s
ratings-v1-8598f7cbcb-dbrkf       2/2     Running   0          84s
reviews-v1-7cc99bb549-9t8rh       2/2     Running   0          84s
reviews-v2-769664c4b-sl6f5        2/2     Running   0          84s
reviews-v3-59b896c68c-mg98r       2/2     Running   0          83s
```

作成したNamespaceでPODをデプロイした際に自動でサイドカープロキシを挿入するようにするには、以下のようにlabelを事前に設定することで都度kube-injectの実施は不要
```
kubectl label namespace ns1 istio-injection=enabled
```

デプロイ後の構成は以下となります。

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/bookinfo2.png" width="90%">

### アクセス確認
ratings PODからproductpageにHTTPアクセス
```
# kubectl exec "$(kubectl get pod -n ns1 -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -n ns1 -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
<title>Simple Bookstore App</title>
```

サイドカープロキシを経由しているか通信ログを確認
```
root@cn2-master1:~/istio-1.14.2# kubectl logs -n ns1 "$(kubectl get pod -n ns1 -l app=productpage -o jsonpath={.items..metadata.name})" -c istio-proxy | tail -5
[2022-07-28T05:12:53.673Z] "GET /reviews/0 HTTP/1.1" 200 - via_upstream - "-" 0 438 878 878 "-" "curl/7.52.1" "bcc505d1-c954-9b74-88c9-17ef0f137e64" "reviews:9080" "10.233.66.20:9080" outbound|9080||reviews.ns1.svc.cluster.local 10.233.66.21:53406 10.233.0.250:9080 10.233.66.21:37600 - default
[2022-07-28T05:12:53.594Z] "GET /productpage HTTP/1.1" 200 - via_upstream - "-" 0 5290 1010 1009 "-" "curl/7.52.1" "bcc505d1-c954-9b74-88c9-17ef0f137e64" "productpage:9080" "10.233.66.21:9080" inbound|9080|| 127.0.0.6:59145 10.233.66.21:9080 10.233.66.17:42306 outbound_.9080_._.productpage.ns1.svc.cluster.local default
[2022-07-28T05:24:43.260Z] "GET /details/0 HTTP/1.1" 200 - via_upstream - "-" 0 178 9 8 "-" "curl/7.52.1" "6b9e92cb-93f4-9e6f-84d0-d9f61279db73" "details:9080" "10.233.66.16:9080" outbound|9080||details.ns1.svc.cluster.local 10.233.66.21:47280 10.233.43.71:9080 10.233.66.21:49272 - default
[2022-07-28T05:24:43.275Z] "GET /reviews/0 HTTP/1.1" 200 - via_upstream - "-" 0 358 786 785 "-" "curl/7.52.1" "6b9e92cb-93f4-9e6f-84d0-d9f61279db73" "reviews:9080" "10.233.66.18:9080" outbound|9080||reviews.ns1.svc.cluster.local 10.233.66.21:51838 10.233.0.250:9080 10.233.66.21:56836 - default
[2022-07-28T05:24:43.244Z] "GET /productpage HTTP/1.1" 200 - via_upstream - "-" 0 4294 857 856 "-" "curl/7.52.1" "6b9e92cb-93f4-9e6f-84d0-d9f61279db73" "productpage:9080" "10.233.66.21:9080" inbound|9080|| 127.0.0.6:42713 10.233.66.21:9080 10.233.66.17:42306 outbound_.9080_._.productpage.ns1.svc.cluster.local default
```

CN2環境ではサイドカープロキシ間の通信はCN2で管理しているVirtual NetworkによるOverlay接続構成となります。

以下の構成図はCN2環境でService Meshを構成した図となります。

CN2 vRouterがPOD毎にデプロイされているような図となりますが、実際にはCN2 vRouterは各WorkerNodeに1台のみデプロイされ、Virtual Network内外通信及び、WorkerNode間のVXLANによるOverlay接続を担います。

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/bookinfo3.png" width="90%">


### Istio IngressGatewayを利用した外部公開
外部からのアクセスはIstio Ingress Gatewayを経由したLoadBalancing接続となります。

CN2環境ではIstio Ingress GatewayにExetnal IPが付与され、外部ルータに対してAdvertiseを行なっているため、外部からアクセスが可能です。

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/bookinfo4.png" width="90%">

Gatewayリソース(Istio ingressgatewayの設定)とVirtualServiceリソース(Istio Ingress Gatewayで受けたトラヒックの宛先設定)の設定を行います。

サンプルで用意されている以下のManifestsを使用します。
```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "*"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
```

Gateway, VirtualServiceをデプロイ
```
# kubectl apply -f ./samples/bookinfo/networking/bookinfo-gateway.yaml -n ns1

# kubectl get gateway -n ns1
NAME               AGE
bookinfo-gateway   62s

# kubectl get virtualservice -n ns1
NAME       GATEWAYS               HOSTS   AGE
bookinfo   ["bookinfo-gateway"]   ["*"]   72s
```

### 外部からのアクセス確認
Istio Ingress Gatewayに設定されたExternal IP宛てにHTTPアクセス
```
# curl 100.0.0.4/productpage | grep "<title>.*</title>"
    <title>Simple Bookstore App</title>
```

## Virtual Networkを使用したサービスメッシュのデプロイ
上記はCN2で管理しているIsolated Namespace内のdefault-podnetwork上でサービスメッシュが動作していますが、別途Virtual Networkを作成し、その上でサービスメッシュを動作させることが可能です。
次期Version:22.3を予定

## Kialiによる可視化
```
kubectl apply -f ./samples/addons/kiali.yaml
kubectl apply -f ./samples/addons/prometheus.yaml
kubectl apply -f ./samples/addons/grafana.yaml

kubectl port-forward --address K8S_MASTER_IP service/kiali -n istio-system 20001:20001
```
<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/kiali.png" width="80%">
