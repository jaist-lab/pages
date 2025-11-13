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


### 1.4 アーキテクチャ

#### Fluent Bit、Fluentd、OpenSearch の構成

[mermaid]
graph TB
    subgraph "Kubernetes Cluster (vessel + nodes)"
        subgraph "logging namespace"
            FB1[Fluent Bit Pod<br/>DaemonSet]
            FB2[Fluent Bit Pod<br/>DaemonSet]
            FB3[Fluent Bit Pod<br/>DaemonSet]
        end
        
        subgraph "default namespace"
            APP1[Application Pod]
            APP2[Application Pod]
        end
        
        subgraph "Node Filesystem"
            LOGS[/var/log/containers/*.log]
        end
        
        APP1 --> LOGS
        APP2 --> LOGS
        LOGS --> FB1
        LOGS --> FB2
        LOGS --> FB3
    end
    
    subgraph "anchor Server (172.16.100.200)"
        subgraph "Docker Compose Stack"
            FD[Fluentd Container<br/>Port: 24224<br/>network_mode: host or bridge]
            
            subgraph "log-net network"
                OS[OpenSearch Container<br/>Port: 9200]
                OSD[OpenSearch Dashboards<br/>Port: 5601]
            end
            
            FD -->|HTTP Bulk API| OS
            OSD -->|Query| OS
        end
        
        subgraph "Volumes"
            OSData[(opensearch-data)]
            FDBuffer[(fluentd-buffer)]
        end
        
        OS -.->|Persist| OSData
        FD -.->|Buffer| FDBuffer
    end
    
    subgraph "User Access"
        Browser[Web Browser<br/>http://localhost:5601]
    end
    
    FB1 -->|Forward Protocol<br/>TCP 24224| FD
    FB2 -->|Forward Protocol<br/>TCP 24224| FD
    FB3 -->|Forward Protocol<br/>TCP 24224| FD
    
    Browser -->|SSH Port Forward or Direct| OSD
    
    style FB1 fill:#e1f5ff
    style FB2 fill:#e1f5ff
    style FB3 fill:#e1f5ff
    style FD fill:#fff4e6
    style OS fill:#e8f5e9
    style OSD fill:#f3e5f5
    style LOGS fill:#fff9c4
    
[/mermaid]


#### データフロー詳細図

[mermaid]
sequenceDiagram
    participant App as Application Pod
    participant Log as /var/log/containers/
    participant FB as Fluent Bit
    participant FD as Fluentd
    participant OS as OpenSearch
    participant OSD as OpenSearch Dashboards
    participant User as User Browser

    App->>Log: Write logs to stdout/stderr
    Note over Log: Container runtime writes to log file
    
    FB->>Log: Tail log files
    FB->>FB: Parse logs (multiline, JSON)
    FB->>FB: Add Kubernetes metadata
    Note over FB: namespace, pod_name, container_name, etc.
    
    FB->>FD: Forward logs (TCP 24224)
    Note over FB,FD: Protocol: Fluentd forward protocol<br/>Connection: 172.16.100.200:24224
    
    FD->>FD: Buffer logs to disk
    Note over FD: Path: /fluentd/log/opensearch_buffer
    
    FD->>OS: Bulk insert (HTTP POST)
    Note over FD,OS: Index: kubernetes-YYYYMMDD<br/>Format: Logstash format
    
    OS->>OS: Index and store logs
    
    User->>OSD: Access dashboard (http://localhost:5601)
    OSD->>OS: Query logs
    OS->>OSD: Return results
    OSD->>User: Display logs
[/mermaid]

#### ネットワーク構成図
[mermaid]
graph LR
    subgraph "Network: 172.16.100.0/24"
        K8S[Kubernetes Nodes<br/>172.16.100.x]
        ANCHOR[anchor Server<br/>172.16.100.200]
    end
    
    subgraph "Kubernetes Internal (10.96.0.0/12)"
        SVC[Service Network]
        POD[Pod Network<br/>10.235.0.0/16]
    end
    
    subgraph "Docker Network on anchor"
        BRIDGE[Bridge Network: log-net<br/>172.18.0.0/16]
        HOST[Host Network<br/>172.16.100.200]
    end
    
    K8S -.->|CNI: Calico| POD
    POD -->|Forward logs| ANCHOR
    
    ANCHOR -->|Option 1: Bridge Mode| BRIDGE
    ANCHOR -->|Option 2: Host Mode| HOST
    
    style K8S fill:#e3f2fd
    style ANCHOR fill:#fff3e0
    style POD fill:#f1f8e9
    style BRIDGE fill:#fce4ec
    style HOST fill:#e0f2f1
[/mermaid]

#### コンポーネント詳細図

[mermaid]
graph TB
    subgraph "Fluent Bit Configuration"
        FBINPUT[INPUT: tail<br/>/var/log/containers/*.log]
        FBFILTER[FILTER: kubernetes<br/>Add metadata]
        FBOUTPUT[OUTPUT: forward<br/>172.16.100.200:24224]
        
        FBINPUT --> FBFILTER
        FBFILTER --> FBOUTPUT
    end
    
    subgraph "Fluentd Configuration"
        FDSOURCE[SOURCE: forward<br/>Port: 24224<br/>Bind: 0.0.0.0]
        FDMATCH[MATCH: **<br/>Type: opensearch]
        FDBUFFER[BUFFER: file<br/>timekey: 1d<br/>chunk_limit: 5M]
        
        FDSOURCE --> FDMATCH
        FDMATCH --> FDBUFFER
    end
    
    subgraph "OpenSearch Index"
        INDEX[Index Pattern:<br/>kubernetes-YYYYMMDD]
        FIELDS[Fields:<br/>@timestamp<br/>kubernetes.namespace_name<br/>kubernetes.pod_name<br/>kubernetes.container_name<br/>log]
    end
    
    FBOUTPUT -->|Forward Protocol| FDSOURCE
    FDBUFFER -->|Bulk API| INDEX
    
    style FBINPUT fill:#e1f5ff
    style FBFILTER fill:#e1f5ff
    style FBOUTPUT fill:#e1f5ff
    style FDSOURCE fill:#fff4e6
    style FDMATCH fill:#fff4e6
    style FDBUFFER fill:#fff4e6
    style INDEX fill:#e8f5e9
    style FIELDS fill:#e8f5e9
[/mermaid]

#### トラブルシューティングフロー図
[mermaid]
graph TD
    START[Logs not appearing in<br/>OpenSearch Dashboards]
    
    CHECK1{Fluent Bit Pods<br/>running?}
    CHECK2{Fluent Bit can<br/>connect to Fluentd?}
    CHECK3{Fluentd listening<br/>on IPv4?}
    CHECK4{Fluentd can connect<br/>to OpenSearch?}
    CHECK5{OpenSearch<br/>healthy?}
    CHECK6{Index created?}
    
    FIX1[Check ArgoCD sync<br/>kubectl get pods -n logging]
    FIX2[Check network connectivity<br/>telnet 172.16.100.200 24224]
    FIX3[Use host network mode or<br/>explicit IPv4 binding]
    FIX4[Check compose.yaml network<br/>config and DNS resolution]
    FIX5[Check OpenSearch logs<br/>docker-compose logs opensearch]
    FIX6[Wait for data or check<br/>Fluentd buffer]
    
    SUCCESS[Create Index Pattern<br/>kubernetes-*<br/>in OpenSearch Dashboards]
    
    START --> CHECK1
    CHECK1 -->|No| FIX1
    CHECK1 -->|Yes| CHECK2
    CHECK2 -->|No| FIX2
    CHECK2 -->|Yes| CHECK3
    CHECK3 -->|No| FIX3
    CHECK3 -->|Yes| CHECK4
    CHECK4 -->|No| FIX4
    CHECK4 -->|Yes| CHECK5
    CHECK5 -->|No| FIX5
    CHECK5 -->|Yes| CHECK6
    CHECK6 -->|No| FIX6
    CHECK6 -->|Yes| SUCCESS
    
    FIX1 --> CHECK1
    FIX2 --> CHECK2
    FIX3 --> CHECK3
    FIX4 --> CHECK4
    FIX5 --> CHECK5
    FIX6 --> CHECK6
    
    style START fill:#ffebee
    style SUCCESS fill:#e8f5e9
    style FIX1 fill:#fff9c4
    style FIX2 fill:#fff9c4
    style FIX3 fill:#fff9c4
    style FIX4 fill:#fff9c4
    style FIX5 fill:#fff9c4
    style FIX6 fill:#fff9c4
[/mermaid]

#### ポート・プロトコル一覧

[mermaid]
graph LR
    subgraph "Protocols and Ports"
        P1[Fluent Bit → Fluentd<br/>Protocol: Forward<br/>Port: 24224/TCP<br/>Format: MessagePack]
        
        P2[Fluentd → OpenSearch<br/>Protocol: HTTP/REST<br/>Port: 9200/TCP<br/>API: _bulk]
        
        P3[OpenSearch Dashboards → OpenSearch<br/>Protocol: HTTP/REST<br/>Port: 9200/TCP<br/>API: _search, _cat, etc.]
        
        P4[User → OpenSearch Dashboards<br/>Protocol: HTTP<br/>Port: 5601/TCP<br/>Web UI]
    end
    
    style P1 fill:#e1f5ff
    style P2 fill:#fff4e6
    style P3 fill:#e8f5e9
    style P4 fill:#f3e5f5
[/mermaid]
