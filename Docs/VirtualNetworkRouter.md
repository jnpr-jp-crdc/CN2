# Virtual Network Router (VNR)
- VNRはRouteLeakによりVirtual Network間を接続
- 1つのVNRに複数のVirtual Networkを接続可能
- Virtual Networkは複数のVNRに接続可能
- VNR Type
  - Mesh: 
    - 全てのPODは互いに通信可能
  - Hub: 
    - Spoke/Hub間のVNRは互いに通信可能
    - Type Spokeとの間でRouteLeakされる
    - Hub内のVirtual Netowrk間は通信不可
  - Spoke:
    - Spoke/Hub間のVNRは互いに通信可能
    - Type Spoke同士はRouteLeakされない
    - Spoke内のVirtual Network間は通信不可

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/vnr.png" width="50%">

# Virtual Network Router作成
## Mesh Case1
- 同一Namespace内にあるVN1, VN2をVNR1(Mesh)にて接続
- Virtual Networkに設定したLabelにマッチした経路をImport

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/vnr-mesh1.png" width="50%">

[VNR Mesh Case1 sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/vnr-mesh1.yaml)

## Mesh Case2
- 同一Namespace内で異なるVNRに接続されているVirtual Network間を接続
- VNRにてVNRに設定したLabelをImportすることでVNR間を接続

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/vnr-mesh2.png" width="50%">

[VNR Mesh Case2 sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/vnr-mesh2.yaml)

## Mesh Case3
- 異なるNamespaceで異なるVNRに接続されているVirtual Network間を接続
- VNRにてNamespaceに設定したLabelをImportすることで異なるNamespaceのVNR間を接続

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/vnr-mesh3.png" width="50%">

[VNR Mesh Case3 sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/vnr-mesh3.yaml)

## Hub & Spoke Case1
- 同一Namespace内でHub&Spoke
- VN1, VN2はSpoke VNRに接続、VN3はHub VNRに接続
- VN1 / VN3間及び VN2 / VN3間は互いに通信可能
- VN1 / VN2間は通信不可

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/vnr-hubspoke1.png" width="50%">

[VNR Hub&Spoke Case1 sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/vnr-hubspoke1.yaml)

## Hub & Spoke Case2
- 異なるNamespace間でHub&Spoke
- VN1, VN2はSpoke VNRに接続、VN3はHub VNRに接続
- VN1 / VN3間及び VN2 / VN3間は互いに通信可能
- VN1 / VN2間は通信不可

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/vnr-hubspoke2.png" width="50%">

[VNR Hub&Spoke Case2 sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/vnr-hubspoke2.yaml)


## Hub & Spoke & Mesh
- 異なるNamespace間でHub&Spoke
- VN1, VN2はSpoke VNRに接続、VN3, VN4はHub VNRとMesh VNRに接続
- VN1 - VN3/VN4間、VN2 - VN3/VN4間は互いに通信可能
- VN3, VN4をMesh VNRにも接続することにより、VN3-VN4間も接続可能となる

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/vnr-full.png" width="50%">

[VNR Hub&Spoke&Mesh sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/vnr-hubspokemesh.yaml)

## Default VNR
- Default Cluster Namespace, Contrail NamespaceにはDefaultで定義されているVNRがあり、default-podnetwork, default-servicenetwork間の接続、Fabric Networkへの接続が定義されている
- Isolated Namespace作成時にもDefaultでVNRが作成される

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/vnr-default.png" width="80%">
