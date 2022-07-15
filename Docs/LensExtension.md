# Lens Extension
- Lens(K8S IDE)のContrail Extensionをインストールすることで、ContrailのオペレーションやリソースモニタリングをLens GUIから利用可能
- リソースはYaml形式の他にForm形式で設定が可能
<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/Lens.png">

## Lens Extension インストール
1. Lens SoftwareをClientPCにインストール(Contrail R22.2時点ではLens 5.4.4推奨)
    - https://k8slens.dev/ 
2. Cluster追加
   - Catalog->Categories->ClustersからClusterを追加
   - Kubeconfigは”/root/.kube/config”をコピペ(serve箇所のIPを環境に合わせて変更)
3. Contrail ExtensionをLensにインストール
    - Lens->Extensionsにてcn2_lens_extension-X.X.X.tgzをインストール
   - CN2 Tabが見えない場合、Lensを再起動
