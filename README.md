# Observability

>Build Your Kubernetes Monitoring System

 ![VictoriaMetrics](https://img.shields.io/badge/VictoriaMetrics-%23000000?logo=victoriametrics&logoColor=wilte)

## 目录结构

```
├─Grafana  # 数据展示
├─PrometheusAlert
│  └─wx-tpl # # 告警模板简单示例
└─VictoriaMetrics
    ├─Ci  
    ├─Components
    ├─Exporter  # 导出器的集合
    └─Internal
        ├─Scrape  # 监控抓取配置
        └─VMAlertmanagerConf # 告警路由配置
```  

## 项目规范

### scrapeTimeout & scrapeInterval

>约定指标抓取的时间间隔，方便对齐

|Flags|Explain|
|:---:|:---:|
|`Blackbox`|HTTP 和 TCP 探测超时统一为 `8s`, ICMP 为 `2s`|
|`Global`|全局默认抓取间隔 `60s`, 超时 `20s`|
|`PodScrape`|默认抓取间隔 `45s` 超时 `15s`|
|`ProbeScrape`|默认抓取间隔 `30s`, 超时 `10s`|


### Ingress 统一域名访问路径

>通过单个域名实现路由管理

|       Path       |      Explain      |
| :--------------: | :---------------: |
|       `/`        | 管理 AgentTarget |
| `/select/0/vmui` |    查询 Metric    |
|    `/alerts`     | 管理 Alertmanager |
|    `/vmalert`    |   查看 AlertRule    |

## 项目部署

### 安装 CRD

```sh
helm upgrade --install vm-operator vm/victoria-metrics-operator --version 0.27.9 -f values.yaml -n vm-operator --create-namespace
```

### 安装 VM-Cluster

```sh
k apply -f Ci/vm-cluster.yaml
```

### 安装 VM-Agent

```sh
k apply -f Components/vm-agent.yaml
```

### 安装 VM-Alert

```sh
k apply -f Components/vm-alert.yaml
```

### 安装 VM-ALertmanager

```sh
k apply -f Components/vm-alertmanager.yaml
```

### 安装 Blackbox-Exporter

```sh
k apply -f Exporter/web-exporter.yaml
```

## Scrap 配置

### Cadvisor Scrape

>VMNodeScrape CRD 提供了抓取 kubernetes 节点指标的发现机制，对于 node_export 监控非常有用
```sh
k apply -f ScrapMonitor/vm-node-scrape.yaml
```

### Probe Scrape

>必须先配置 Blackbox-export 提供探测能力，CRD 提供 静态抓取和服务发现。
```sh
k apply -f Components/web-export.yaml
k apply -f ScrapMonitor/vm-probe-scrape.yaml
```

## TroubleShooting

- Agent 报错 403

>这是一个 RBAC 权限问题, 需要添加抓取对象的权限。 
```sh
unexpected status code returned when scraping \"https://kubernetes.default.svc:443/api/v1/nodes/worker-2/proxy/metrics/cadvisor\": 403; expecting 200
```
