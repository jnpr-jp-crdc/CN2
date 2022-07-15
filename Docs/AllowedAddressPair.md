# Allowed Address Pair
- Allowed Address Pair(VIP)をPODに付与
- Address Modeとして以下の2つを設定可能
  - active-active: VIPへのアクセスはECMPにより分散
  - active-standby: VRRP

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/AllowedAddressPair.png" width="50%">

[Allowed Address Pair sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/AllowedAddressPair.yaml)

[Keepalived sample config](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/AAP-Keepalived.conf)
