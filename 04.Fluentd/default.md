---
title: fluentd
---
# fluentd/fluent-bit/OpenSearchDashboard

## 1.1 構築目標

- **各Kubernetesクラスタ**: fluent-bitでログ収集・保存
- **anchorサーバ**: ログ集約・保存
- **データ収集**: fluent-bitによるログ収集　 24224/tcp -> 0.0.0.0:24224


### 1.2 対象環境

| 環境 | クラスタ名 | Prometheus役割 | 保存期間 | Master IP | ストレージ |
|------|-----------|---------------|---------|----------|-----------|
| Production | production-cluster | ローカル収集 | 7日 | 172.16.100.101 | Ceph RBD |
| Development | development-cluster | ローカル収集 | 7日 | 172.16.100.121 | Ceph RBD |
| Sandbox | sandbox-cluster | ローカル収集 | 7日 | 172.16.100.131 | Ceph RBD |
| anchor | - | 長期保存 | 90日 | 172.16.200.200 | Local |




### 1.3 アクセス先一覧

| 環境 | URL | 認証情報 |
|------|-----------|---------|---------|
| fluentd | [http://172.16.100.200:5601](http://172.16.100.200:5601) |  　|
| OpenSearchDashboard | [http://172.16.100.200:5601](http://172.16.100.200:5601) | 認証なし|


http://172.16.100.200:5601/

### 1.4 アーキテクチャ

#### Fluent Bit、Fluentd、OpenSearch の構成

[mermaid]

graph TD
    subgraph "Kubernetes Cluster (Production/Sandbox)"
        subgraph "各ノード"
            A[Container Logs<br>/var/log/containers/*.log]
            B[Fluent Bit DaemonSet<br>Pod]
            A -->|tail| B
        end
        
        subgraph "Fluent Bit 処理フロー"
            B --> C[INPUT<br>tail plugin]
            C --> D[FILTER<br>kubernetes plugin]
            D --> E[OUTPUT<br>forward plugin]
        end
        
        E -->|TCP 24224<br>Forward Protocol| F
    end
    
    subgraph "anchor Server (172.16.100.200)"
        F[Fluentd<br>Port 24224]
        G[OpenSearch<br>Port 9200]
        H[OpenSearch Dashboards<br>Port 5601]
        
        F -->|parse & transform| F1[Fluentd Processing]
        F1 -->|HTTP/REST API| G
        G -->|query| H
    end
    
    subgraph "User"
        I[Web Browser]
        I -->|HTTP 5601| H
    end
    
    style A fill:#e1f5ff
    style B fill:#b3e5fc
    style C fill:#81d4fa
    style D fill:#4fc3f7
    style E fill:#29b6f6
    style F fill:#ffccbc
    style F1 fill:#ffab91
    style G fill:#a5d6a7
    style H fill:#ce93d8
    style I fill:#fff9c4

[/mermaid]
