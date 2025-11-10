---
title: Servers
---

## Servers 

---

### Nodes

|IP|hostname|用途|
|---|---|---|
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

---

### **Production K8s node**

|IP|hostname|用途|
|---|---|---|
|172.16.100.101|master01|K8sマスタ|
|172.16.100.102|master02|K8sマスタ|
|172.16.100.103|master03|K8sマスタ|
|172.16.100.104|node01|K8sワーカーノード|
|172.16.100.105|node02|K8sワーカーノード|
|**CI/CD software** |||
|ArgoCD | [http://172.16.100.101:32443](https://172.16.100.101:32443)|admin/jaileon02|
---

### **Development K8s node**

|IP|hostname|用途|
|---|---|---|
|172.16.100.121|dev-master01|K8sマスタ|
|172.16.100.122|dev-master02|K8sマスタ|
|172.16.100.123|dev-master03|K8sマスタ|
|172.16.100.124|dev-node01|K8sワーカーノード|
|172.16.100.125|dev-node02|K8sワーカーノード|
|**CI/CD software** |||
|ArgoCD | [http://172.16.100.121:32443](https://172.16.100.121:32443)|admin/jaileon02|
---

### **Sandbox K8s node**

|IP|hostname|用途|
|---|---|---|
|172.16.100.131|sandbox-master21|K8sマスタ|
|172.16.100.132|sandbox-master22|K8sマスタ|
|172.16.100.133|sandbox-master23|K8sマスタ|
|172.16.100.134|sandbox-node21|K8sワーカーノード|
|172.16.100.135|sandbox-node22|K8sワーカーノード|
|**CI/CD software** |||
|ArgoCD | [http://172.16.100.131:32443](https://172.16.100.131:32443)|admin/jaileon02|
---

