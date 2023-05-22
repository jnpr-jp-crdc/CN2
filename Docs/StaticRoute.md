# Static Route
CN2環境では通常PODはVirtual Networkに設定されたDefault GW経由で別Virtual Network及び外部へアクセスを行うが、Virtual NetworkにStatic Routeを設定することにより、特定の宛先を特定のPOD向けにリダイレクトさせることが可能となる。
※PODのDefault GWはVirtual NetworkのDefault GWのままでよい

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/StaticRoute1.png" width="50%">

## Static Route to Virtual Network
- Static Routeを定義し、Virtual Networkにアサインする方式

※ API Versionがv3であることに注意
### Route Table
```
apiVersion: core.contrail.juniper.net/v3
kind: RouteTable
metadata:
  name: static-rt
  namespace: ns1
spec:
  routes:
    route:
      - nextHop: 10.0.0.3
        nextHopType: ip-address
        prefix: 20.0.0.0/24
        communityAttributes:
          communityAttribute:
            - accept-own
            - no-advertise
```
### Virtual NetworkにStatic Routeをアサイン
```
apiVersion: core.contrail.juniper.net/v3
kind: VirtualNetwork
metadata:
  namespace: ns1
  name: vn1
spec:
  v4SubnetReference:
    apiVersion: core.contrail.juniper.net/v3
    kind: Subnet
    namespace: ns1
    name: sub1-v4
  routeTableReferences:
    - apiVersion: core.contrail.juniper.net/v3
      kind: RouteTable
      namespace: ns1
      name: static-rt
```

[Static Route Sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/StaticRoute.yaml)

## Static Route to Virtual Machine Interface
- Static Routeを定義し、Virtual Machine Interface(PODのInterface)にアサインする方式
- Static RouteのNexthop IPを明示していする必要がなく、PODのInterfaceへリダイレクトされる

※ API Versionがv3であることに注意

### Route Table
```
apiVersion: core.contrail.juniper.net/v3
kind: RouteTable
metadata:
  name: static-rt
  namespace: ns1
spec:
  routes:
    route:
      - nextHopType: ip-address
        prefix: 20.0.0.0/24
        communityAttributes:
          communityAttribute:
            - accept-own
```
### Virtual Machine Interface
```
apiVersion: v3
kind: VirtualMachineInterface
metadata:
  name: static-route-pod
  namespace: static-route
  annotations:
     core.juniper.net/interface-route-table: '[{"name": "static-rt", "namespace": "ns1"}]'
```