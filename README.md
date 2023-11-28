## Observability

>Build Your Kubernetes Monitoring System

![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=Prometheus&logoColor=white) ![Grafana](https://img.shields.io/badge/grafana-%23F46800.svg?style=for-the-badge&logo=grafana&logoColor=white) 	![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white) ![VictoriaMetrics](https://img.shields.io/badge/VictoriaMetrics-%23512BD4?style=for-the-badge&logo=VictoriaMetrics&logoColor=white)

### 项目背景

___
>记录自己搭建整套可观测性的过程。

### 目录结构

```
├─Log
│  ├─Export   # 日志类的 Export
│  │  └─k8sEvenExport
│  ├─Loki   # 日志记录引擎，负责存储日志和处理查询
│  ├─Minio    # s3 持久化存储日志
│  ├─Promtail   # 代理，负责收集日志并将其发送给 loki
│  ├─Redis    # 缓存数据库
│  └─Rules    # 日志告警规则
├─Metric
│  ├─Alertmanager   # 告警平台
│  ├─ExternalRules    # 扩展的告警规则
│  ├─Kube-Prometheus     # Kube-Prometheus-Operator 项目版本
│  │  └─Release-0.12
│  │      └─manifests
│  │          └─setup
│  ├─PrometheusAlert  # 告警通知渠道，格式化模板
│  │  └─wx-tpl    # 告警模板简单示例
│  └─ScrapMonitor   # 监控抓取配置
│      └─exporter   # Export 配置
└─UI
    └─Grafana   # 数据展示
```  

### 项目说明

>日志系统方案使用 `PLAG` 结构，由 `Promtail` 数据采集 + `Loki` 做数据读/写/存 + `Alertmanager` 告警 + `Grafana` 展示组成。 

- `Loki` 日志记录引擎，负责存储日志和处理查询，查询时基于标签特性
- `Promtail` 代理，负责收集日志并将其发送给 `loki`，是专门为 `Loki` 量身定制的日志采集器，支持多种格式并进行结构化日志
- `Grafana` **UI** 界面，用于可视化和监控数据的工具，与 `Kibana` 相似。它提供了丰富的图表和仪表盘，可以帮助用户实时监控和分析日志数据。


>指标系统方案使用 `PVAG` 结构，由 `Prometheus` 数据采集 + `VictoriaMetrics` 统一存储 + `Alertmanager` 告警 + `Grafana` 展示组成。

- `Victoria-Metrics` **(VM)** 是一个高性能、可扩展的时序数据库和长期存储解决方案。
- 它专门针对 `Prometheus` 数据进行了优化，可以有效地存储和查询大规模的时间序列数据。
- `Victoria-Metrics` 支持 `Prometheus` 的查询语言 `PromQL`，并提供了额外的功能，数据压缩和快速数据恢复。


### 项目部署

- [LOG](./Log/README.md)
- [METRICS](./Metric/README.md)

>查看**目录**下的 *README.me* 安装步骤进行创建 `yaml`。
