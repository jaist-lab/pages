# Prometheus/Grafana
## 1.1 構築目標

- **各Kubernetesクラスタ**: Prometheusで短期メトリクス収集・保存（7日間）
- **anchorサーバ**: 長期メトリクス集約・保存（90日間）
- **GPU監視**: NVIDIA DCGM Exporterメトリクスの収集
- **ストレージ**: Ceph RBDによる動的プロビジョニング

### 1.2 対象環境

| 環境 | クラスタ名 | Prometheus役割 | 保存期間 | Master IP | ストレージ |
|------|-----------|---------------|---------|----------|-----------|
| Production | production-cluster | ローカル収集 | 7日 | 172.16.100.101 | Ceph RBD |
| Development | development-cluster | ローカル収集 | 7日 | 172.16.100.121 | Ceph RBD |
| Sandbox | sandbox-cluster | ローカル収集 | 7日 | 172.16.100.131 | Ceph RBD |
| anchor | - | 長期保存 | 90日 | 172.16.200.200 | Local |

### 1.3 アーキテクチャ

```
Production Cluster → Prometheus (7日/Ceph) → Remote Write ↘
Development Cluster → Prometheus (7日/Ceph) → Remote Write → anchor Prometheus (90日) → Grafana
Sandbox Cluster → Prometheus (7日/Ceph) → Remote Write ↗
```

[mermaid]

graph LR

    subgraph "Sandbox Cluster (172.16.100.131)"
        S1[Prometheus<br/>NodePort: 32090<br/>保存期間: 7日]
        SG1[Grafana<br/>NodePort: 32000]
        SNE1[Node Exporter<br/>9100]
        SKS1[Kube State Metrics<br/>8080]
        
        SNE1 -->|メトリクス収集| S1
        SKS1 -->|メトリクス収集| S1
        S1 -->|データソース| SG1
    end
    
    subgraph "Development Cluster (172.16.100.121)"
        D1[Prometheus<br/>NodePort: 32090<br/>保存期間: 7日]
        DG1[Grafana<br/>NodePort: 32000]
        DNE1[Node Exporter<br/>9100]
        DKS1[Kube State Metrics<br/>8080]
        DDCGM1[DCGM Exporter<br/>GPU監視<br/>9400]
        
        DNE1 -->|メトリクス収集| D1
        DKS1 -->|メトリクス収集| D1
        DDCGM1 -->|GPU メトリクス| D1
        D1 -->|データソース| DG1
    end


    subgraph "Production Cluster (172.16.100.101)"
        P1[Prometheus<br/>NodePort: 32090<br/>保存期間: 7日]
        PG1[Grafana<br/>NodePort: 32000]
        PNE1[Node Exporter<br/>9100]
        PKS1[Kube State Metrics<br/>8080]
        PDCGM1[DCGM Exporter<br/>GPU監視<br/>9400]
        
        PNE1 -->|メトリクス収集| P1
        PKS1 -->|メトリクス収集| P1
        PDCGM1 -->|GPU メトリクス| P1
        P1 -->|データソース| PG1
    end



    subgraph "anchor (172.16.200.200)"
        AG[Grafana 11.4.0<br/>Port: 3000<br/>admin/jaist-monitoring-2025]
        ANE[Node Exporter<br/>Port: 9100]
        AP[Prometheus v2.47.2<br/>Port: 9090<br/>保存期間: 90日<br/>remote-write-receiver有効]
        
        AP -->|データソース| AG
        ANE -->|ホストメトリクス| AP
    end
    
    P1 -.->|Remote Write<br/>HTTP POST<br/>api/v1/write| AP
    D1 -.->|Remote Write<br/>HTTP POST<br/>api/v1/write| AP
    S1 -.->|Remote Write<br/>HTTP POST<br/>api/v1/write| AP
    
    P1 -.->|Federation<br/>GET /federate| AP
    D1 -.->|Federation<br/>GET /federate| AP
    S1 -.->|Federation<br/>GET /federate| AP
    
    style AP fill:#ff9999
    style AG fill:#99ccff
    style P1 fill:#ffcc99
    style D1 fill:#ffcc99
    style S1 fill:#ffcc99
    style PG1 fill:#ccffcc
    style DG1 fill:#ccffcc
    style SG1 fill:#ccffcc
    
    classDef exporter fill:#ffffcc
    class PNE1,PKS1,PDCGM1,DNE1,DKS1,DDCGM1,SNE1,SKS1,SDCGM1,ANE exporter
[/mermaid]


---

## 2. 前提条件とアーキテクチャ

### 2.1 必要な環境

- **Ceph クラスタ**: 稼働中のCephクラスタ
- **Kubernetes**: v1.31.3 (Kubespray v2.28.0)
- **Helm**: v3.x
- **anchor**: Podman環境

### 2.2 ネットワーク構成

```
各クラスタ管理網: 172.16.200.xxx
各クラスタ内部網: 172.16.100.xxx
Ceph Monitor: 172.16.200.11-15:6789
anchor: 172.16.200.200
vessel: 管理サーバ
```

### 2.3 アクセス先一覧

| 環境 | Prometheus | Grafana | 認証情報 |
|------|-----------|---------|---------|
| Production | [http://172.16.100.101:32090](http://172.16.100.101:32090) | http://172.16.100.101:32000 | admin/jaist-prod-monitoring |
| Development | http://172.16.100.121:32090 | http://172.16.100.121:32000 | admin/jaist-dev-monitoring |
| Sandbox | http://172.16.100.131:32090 | http://172.16.100.131:32000 | admin/jaist-sandbox-monitoring |
| anchor | http://172.16.200.200:9090 | http://172.16.200.200:3000 | admin/jaist-monitoring-2025 |

---