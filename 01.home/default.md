---
title: SDLCF Research page 
---
# Research

---

## Project

### NEDO
- [「ポスト5G情報通信システム基盤強化研究開発事業／ポスト5G情報通信システムの開発（委託）」に係る実施体制の決定について(2023/12/05)](https://www.nedo.go.jp/koubo/IT3_100292.html)
- [超高効率AI計算基盤の研究開発](https://www.meti.go.jp/policy/mono_info_service/joho/post5g/231205_gaiyou.pdf)

### Press release
- [PFN、IIJ、JAISTが共同で超高効率AI計算基盤の研究開発を開始(2023/12/05)](https://www.jaist.ac.jp/whatsnew/press/2023/12/05-1.html)
- [直接水冷方式の高密度AIサーバおよびハイブリッド冷却データセンターによる 超高効率AI計算基盤技術の実証実験を開始(2025/09/11)](https://www.jaist.ac.jp/whatsnew/press/2025/09/11-1.html)

---

## Applications

|Application|URL|login||
|---|---|---|---|
|**Contents management ** |||
|Grav | [http://150.65.146.97/](http://150.65.146.97/)|admin/Jaileon02||
|**Vertial machine management ** |||
|ProxmoxVE r760xs1 | [http://172.16.100.11:8006](http://172.16.100.11:8006)|root/jaileon02|Firefox推奨|
|ProxmoxVE r760xs2 | [http://172.16.100.12:8006](http://172.16.100.12:8006)|root/jaileon02|Firefox推奨|
|ProxmoxVE r760xs3 | [http://172.16.100.13:8006](http://172.16.100.13:8006)|root/jaileon02|Firefox推奨|
|ProxmoxVE r760xs4 | [http://172.16.100.14:8006](http://172.16.100.14:8006)|root/jaileon02|Firefox推奨|
|ProxmoxVE r760xs5 | [http://172.16.100.15:8006](http://172.16.100.15:8006)|root/jaileon02|Firefox推奨|
|**Ceph Dashboard** |||
|Ceph Dashboard | [https://172.16.100.11:8443](https://172.16.100.11:8443)|admin/jaileon02|https!|
|**VPN software ** |||
|wireGuard | [vpn access]()|||
|**CI/CD software ** |||
|ArgoCD | [http://172.16.100.131:32443](https://172.16.100.131:32443)|admin/jaileon02||
---

## Servers

|IP|hostname|用途|
|---|---|---|
|**Servers and netowork** |||
|172.16.100.1|F310|ルータ、DHCPサーバ|
|172.16.100.11|r760xs1|SDLCFサーバ|
|172.16.100.12|r760xs2|SDLCFサーバ|
|172.16.100.13|r760xs3|SDLCFサーバ|
|172.16.100.14|r760xs4|SDLCFサーバ|
|172.16.100.15|r760xs5|SDLCFサーバ|
|172.16.100.31|dlcsv1|SDLCFサーバ（水冷）|
|172.16.100.32|dlcsv2|SDLCFサーバ（水冷）|
|172.16.100.33|dlcsv3|SDLCFサーバ（水冷）|
|172.16.100.34|dlcsv4|SDLCFサーバ（水冷）|
|**Production K8s node**|||
|172.16.100.101|master01|K8sマスタ|
|172.16.100.102|master02|K8sマスタ|
|172.16.100.103|master03|K8sマスタ|
|172.16.100.104|node01|K8sワーカーノード|
|172.16.100.105|node02|K8sワーカーノード|
|**Development1 K8s node**|||
|172.16.100.121|dev-master01|K8sマスタ|
|172.16.100.122|dev-master02|K8sマスタ|
|172.16.100.123|dev-master03|K8sマスタ|
|172.16.100.124|dev-node01|K8sワーカーノード|
|172.16.100.125|dev-node02|K8sワーカーノード|
|**Development2 K8s node**|||
|172.16.100.131|dev-master21|K8sマスタ|
|172.16.100.132|dev-master22|K8sマスタ|
|172.16.100.133|dev-master23|K8sマスタ|
|172.16.100.134|dev-node21|K8sワーカーノード|
|172.16.100.135|dev-node22|K8sワーカーノード|

---

