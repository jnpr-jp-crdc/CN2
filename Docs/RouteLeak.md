# Route Leak
## Route Leak - Same Route Target
- VNRを使用せず、Virtual NetworkのRoute Targetを揃えることでVirtual Network間接続が可能
- Route TargetによるRoute LeakはVirutal Network間だけでなく、外部ルータの経路もImport可能

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/RouteLeak1.png" width="50%">

[Route Leak Same RT sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/RouteLeak-sameRT.yaml)

## Route Leak - Route Target Import
- VNRを使用せず、Virtual NetworkのRoute TargetをExport/ImportすることでVirtual Network間接続が可能
- Route TargetによるRoute LeakはVirutal Network間だけでなく、外部ルータの経路もImport可能

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/RouteLeak2.png" width="50%">

[Route Leak RT Import sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/RouteLeak-RT-Import.yaml)
