# WebUI
- R23.1からWebUIがサポートされました。
- 使用するにはContrail Analyticsのインストールが必要。

## WebUI アクセス方法
https://CONTRAIL-MASTER-IP:30443

Kubeconfファイル(.kube/config)をLocalPCにDownloadし、Login

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/WebUI-kubeconfig.png" width="50%">

## Dashboard
Top画面であるDashboardは自由にカスタマイズ可能

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/WebUI1.png" width="50%">

## Monitoring
- Central / Distributed Cluster毎のNode / POD / Namespace / Service / Virtual Network リソースモニタリング
- Cluster / Node / POD の正常性モニタリング
- Node / POD のCPU / Memory / Disk Util
- Top X CPU / Memory / Disk per Cluster / Node / POD

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/WebUI2.png" width="50%">

## Configure Kubernetes
- POD, Service, Volume等、Contrailに関連する設定以外にKubernetesに関する設定もCN2 WebUIから実施可能
- 設定はWebUIからManifestsファイルを編集

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/WebUI3.png" width="50%">

## Configure CN2
- Contrail関連の設定をForm形式で設定可能

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/WebUI4.png" width="50%">

- Virtual Network RouterのTopology表示が可能

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/WebUI5.png" width="50%">