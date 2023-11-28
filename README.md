## Observability

>Build Your Kubernetes Monitoring System

![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=Prometheus&logoColor=white) ![Grafana](https://img.shields.io/badge/grafana-%23F46800.svg?style=for-the-badge&logo=grafana&logoColor=white) 	![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white) ![VictoriaMetrics](https://img.shields.io/badge/VictoriaMetrics-%23512BD4?style=for-the-badge&logo=VictoriaMetrics&logoColor=white)

### 目录结构

```
├─Log
│  ├─Export   # 日志类的 Export
│  │  └─k8sEvenExport
│  ├─Loki   # 日志记录引擎，负责存储日志和处理查询
│  ├─Minio    # s3 持久化存储日志
│  ├─Promtail   # 代理，负责收集日志并将其发送给 loki
│  ├─Redis  # 缓存数据库
│  └─Rules  # 告警规则
└─Metric
    ├─Alertmanager    # 告警平台
    ├─ExternalRules   # 扩展的告警规则
    ├─Grafana   # 数据展示
    ├─Kube-Prometheus   # Kube-Prometheus-Operator 项目版本
    │  └─Release-0.12
    │      └─manifests
    │          └─setup
    ├─PrometheusAlert   # 告警通知渠道，格式化模板
    │  └─wx-tpl   # 告警模板简单示例
    └─ScrapMonitor    # 监控抓取配置
        └─exporter    # Export 配置
```  

### 项目说明

>日志系统 `Loki + Grafana + Promtail` 

```sh
git clone -b PLG-Release-0.1 https://github.com/lipaysamart/Observability.git
```

>指标系统 `Prometheus + Grafana + Alertmanager + PrometheusAlert`

```sh
git clone -b Kube-Release-0.12 https://github.com/lipaysamart/Observability.git
```