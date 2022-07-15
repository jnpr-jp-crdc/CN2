# Network Policy
- Namespace, Virtual Networkを跨る通信及び、Namespace, Virtual Network内通信、外部通信に対してNetwork Policy(Firewall)を適用
- Network PolicyはPODのLabelに紐付く
- Ingress, Egressの設定が可能(指定しない場合はIngressが適用される)

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/NetworkPolicy.png" width="50%">

[Network Policy sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/NetworkPolicy.yaml)

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: policy1
  namespace: ns1
spec:
  podSelector:
    matchLabels:
      role: webserver                # PODに設定したLabelにマッチした場合、NetworkPolicyがPODに適用される 
  policyTypes:
  - Ingress
  - Egress
  ingress:                           # Network Policyを適用するPOD向けの通信制御
  - from:                            # 送信元にマッチする条件を指定
    - ipBlock:                       # ipBlock, namespaceSelector, podSelectorはor条件 
        cidr: 10.0.0.0/24
        except:
        - 10.0.0.3/32
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: client
    ports:                           # fromとportsはand条件(両方がマッチしたトラヒックが許可される) 
    - protocol: TCP
      port: 80                       # Network Policyを適用するPOD向け通信の送信Port 
  egress:                            # Network Policyを適用するPOD発信の通信制御
  - to:
    - podSelector:
        matchLabels:
          role: webserver
    ports:                           # toとportsはand条件(両方がマッチしたトラヒックが許可される) 
    - protocol: TCP
      port: 80                       # Network Policyを適用するPOD発信の送信Port 
```
## Network Policy Kubernetes / Contrail オブジェクトマッピング
<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/NetworkPolicy-mapping.png">

## Policy作成時に生成されるContrail オブジェクト
### Address Group
```
# kubectl get addressgroups -n ns1
NAME          SUBNETS   STATE     AGE
10.0.0.0-24   1         Success   39m
10.0.0.3-32   1         Success   39m
```

### Tag
```
# kubectl get tags -n ns1
NAME             TAGTYPE     TAGVALUE       ID        STATE     AGE
tag-7965fbc5d9   role        webserver      1507334   Success   43m
tag-79f7d87f89   role        client         1507335   Success   43m
```

### Firewall policy
```
# kubectl get firewallpolicies -n ns1
NAME            RULES   STATE     AGE
k8s-allow-all   2       Success   49m
k8s-deny-all    2       Success   49m
ns1-policy1     5       Success   49m
```

### Firewall RULES
```
# kubectl get firewallrules -n ns1
NAME                                              SERVICE                ENDPOINT1   DIRECTION   ENDPOINT2   ACTION   STATE     AGE
default-egress-ns1-policy1-denyall                any/0:65535->0:65535               >                       deny     Success   47m
default-ingress-ns1-policy1-denyall               any/0:65535->0:65535               <                       deny     Success   47m
k8s-Namespace-ns1-egress                          any/0:65535->0:65535               >                       pass     Success   47m
k8s-Namespace-ns1-ingress                         any/0:65535->0:65535               <                       pass     Success   47m
ns1-egress-policy1-0-PodSelector-0                tcp/0:65535->80:80                 >                       pass     Success   47m
ns1-ingress-policy1-0-PodSelector-2               tcp/0:65535->80:80                 <                       pass     Success   47m
ns1-ingress-policy1-0-namespaceSelector-1         tcp/0:65535->80:80                 <                       pass     Success   47m
ns1-ingress-policy1-ipBlock-0-cidr--10.0.0.0-24   tcp/0:65535->80:80                 <                       pass     Success   47m
ns1-ingress-policy1-ipBlock-0-cidr--10.0.0.3-32   tcp/0:65535->80:80                 <                       deny     Success   47m
```

## Firewall Ruleシーケンス
```
# kubectl describe firewallpolicies ns1-policy1 -n ns1
Name:         ns1-policy1
--- snip ----
Spec:
  Firewall Rule References:
    API Version:  core.contrail.juniper.net/v1alpha1
    Attributes:
      Sequence:  1.000000
    Fq Name:
      default-domain
      ns1
      ns1-ingress-policy1-ipBlock-0-cidr--10.0.0.3-32
    Kind:              FirewallRule
    Name:              ns1-ingress-policy1-ipBlock-0-cidr--10.0.0.3-32
    Namespace:         ns1
    Resource Version:  5427018
    UID:               8735915c-584e-42f5-917c-19555b6ae8e3
    API Version:       core.contrail.juniper.net/v1alpha1
    Attributes:
      Sequence:  2.000000
    Fq Name:
      default-domain
      ns1
      ns1-ingress-policy1-ipBlock-0-cidr--10.0.0.0-24
    Kind:              FirewallRule
    Name:              ns1-ingress-policy1-ipBlock-0-cidr--10.0.0.0-24
    Namespace:         ns1
    Resource Version:  5427032
    UID:               caf01e6a-b4ee-4b19-8141-db65ec0ff1ec
    API Version:       core.contrail.juniper.net/v1alpha1
    Attributes:
      Sequence:  3.000000
    Fq Name:
      default-domain
      ns1
      ns1-ingress-policy1-0-namespaceSelector-1
      Kind:              FirewallRule
      Name:              ns1-ingress-policy1-0-namespaceSelector-1
      Namespace:         ns1
      Resource Version:  5427035
      UID:               5885364e-2ace-4c71-a62e-772fdc1dae22
      API Version:       core.contrail.juniper.net/v1alpha1
      Attributes:
        Sequence:  4.000000
      Fq Name:
        default-domain
        ns1
        ns1-ingress-policy1-0-PodSelector-2
      Kind:              FirewallRule
      Name:              ns1-ingress-policy1-0-PodSelector-2
      Namespace:         ns1
      Resource Version:  5427042
      UID:               068e72cd-d6f8-49ed-92d9-d1740516cb27
      API Version:       core.contrail.juniper.net/v1alpha1
      Attributes:
        Sequence:  5.000000
      Fq Name:
        default-domain
        ns1
        ns1-egress-policy1-0-PodSelector-0
      Kind:              FirewallRule
      Name:              ns1-egress-policy1-0-PodSelector-0
      Namespace:         ns1
      Resource Version:  5427047
      UID:               76982eee-0e5c-491c-9fb5-63d4c58c1478
    Fq Name:
      default-domain
      ns1
      ns1-policy1
```
