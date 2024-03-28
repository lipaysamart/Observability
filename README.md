# Observability

>Build Your Kubernetes Monitoring System

![VictoriaMetrics](https://img.shields.io/badge/VictoriaMetrics-%23000000?logo=victoriametrics&logoColor=wilte)

## 目录结构

```
├─Docker  # 模拟多租户写入 VMCluster
├─Grafana # 数据展示
├─PrometheusAlert #
│  └─wx-tpl
└─VictoriaMetrics
    ├─Ci
    ├─Components  # VM-Operator 组件
    └─Internal
        ├─BlackboxExporter  # 探针导出器
        ├─ExternalRules
        ├─KubeStateMetrics
        ├─NodeExporter  # 节点导出器
        ├─ScrapeConf  # 监控抓取配置
        │  └─otherExamples  # 其它示例抓取配置
        └─VMAlertmanagerConf  # 告警路由配置
```  

## 项目规范

### ScrapeTimeout & ScrapeInterval

>约定指标抓取的时间间隔，方便对齐

|       Flags        |                    Explain                    |
| :----------------: | :-------------------------------------------: |
|     `Blackbox`     | HTTP 和 TCP 探测超时统一为 `8s`, ICMP 为 `2s` |
|      `Global`      |      全局默认抓取间隔 `60s`, 超时 `20s`       |
|    `PodScrape`     |         默认抓取间隔 `45s` 超时 `15s`         |
|   `ProbeScrape`    |        默认抓取间隔 `30s`, 超时 `10s`         |
| `VM-ServiceScrape` |        默认抓取间隔 `30s`, 超时 `10s`         |
|     `VM-Rule`      |            默认规则评估间隔 `120s`            |

### VM-Rule & VM-Alert

>告警的严重等级在 Emergency - Info 之间。

|   Level   |                                           Explain                                            |
| :-------: | :------------------------------------------------------------------------------------------: |
| Emergency | 最高级别的告警，表示存在紧急情况或系统崩溃。需要立即采取紧急措施来避免进一步的损失或停机时间 |
| Critical  |          这一级别的告警表示存在严重的问题或错误，可能会对系统的正常运行产生重大影响          |
|  Warning  |    这一级别的告警表示存在一些潜在的问题或异常情况，需要引起注意，但不会对系统造成严重影响    |
|   Info    |      这是最低级别的告警，通常用于提供一般性的信息或状态更新。这些告警不需要立即采取行动      |


### VM-Auth 

>统一 `VictoriaMetrics` 微组件访问入口

|            Path            |      Explain      |
| :------------------------: | :---------------: |
|         `/targets`         | 管理 AgentTarget  |
| `/select/<tenant_id>/vmui` |    查询 Metric    |
|         `/alerts`          | 管理 Alertmanager |
|         `/vmalert`         |  查看 AlertRule   |

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

### 安装 VM-Auth

>CRD 对象没有 ingress 配置。相反，可以使用 `VMAuth` 作为 `ingress-controller` 和 `VictoriaMetrics` 组件之间的代理。

```sh
k apply -f Components/vm-auth
```

### 安装 VM-User

>VMUser 描述用户配置、身份验证方法 `basic auth`,`Authorization` 。用户访问权限、以及可能的路由信息。配合 `VM-Auth` 使用。

```sh
k apply -f Components/vm-user
```

>补充说明: 通过 `VM-Auth` 配置的域名进行访问的时候，会去匹配 `VM-User` 中提供的路由端点。如果找不到，会显示 *MissRouter*。配置的路由端点必须有 `ClusterIP`。如果是 `Headless Service` 则需配置静态端点: 

```yaml
# Headless Service
    - static:
        urls: 
          - http://vmselect-vmcluster-0.vmselect-vmcluster.vm-operator.svc.cluster.local:8481
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


## Tips

- Multitenant

>tenant_id 可以是 [0 .. 2^32) 任意数字，每个租户数据都将是隔离的。也就是说在一条查询中不会查询其它租户的数据出来。这个优势在于可以减少查询时的查询量，提高查询速度。
>>`vmcluster` 只需要一套，其它租户加入监控须安装以下组件，`vmagent`、`vmalert`、`vmrule`

```sh
http://<vmselect>:8481/insert/<tenant_id>/prometheus/api/v1/write
```

>`\Docker` 目录下用来模拟多租户场景写入 `VM` 集群，用于调试。

```sh
docker compose up -d
```

>可以通过 url 获取已注册租户的列表

```sh
http://<vmselect>:8481/admin/tenants
```

>通过 externalLabels 来使租户具有可读性

```yaml
global:
  external_labels:
    tenant: "GuangDong"
```

- Recommended Alert Metrics

| 应用监控     | 状态监控           | 数据库监控         | 主机监控       |
| :----------- | :----------------- | :----------------- | :------------- |
| 调用错误次数 | HTTP接口返回状态码 | 数据库调用错误次数 | Cpu/Mem使用率  |
| 调用错误率   | 调用响应时间       | 数据库调用响应时间 | 磁盘空间使用率 |
| 调用响应时间 | ...                | 线程池使用率       | ...            |
| ...          |                    | 队列大小           |                |
| ...          |                    | ...                |                |


- ServiceMonitor — VM-ServiceScrape

>从 ServiceMonitor 迁移到 VMServiceScrape, 需要更改以下几个字段。

|      Before       |        After         |
| :---------------: | :------------------: |
| metricRelabelings | metricRelabelConfigs |
|    relabelings    |    relabelConfigs    |
|        ...        |         ...          |

## TroubleShooting

- Agent 报错 403

>这是一个 RBAC 权限问题, 需要添加抓取对象的权限。 
```sh
unexpected status code returned when scraping \"https://kubernetes.default.svc:443/api/v1/nodes/worker-2/proxy/metrics/cadvisor\": 403; expecting 200
```

- rollup result cache resets

>这是一个在远程写入时的报错，对应告警会有 `PrometheusRemoteWriteDesiredShards` or `PrometheusRemoteStorageFailures` --相关 (issue 4070)

解决方法: 添加 `-search.disableAutoCacheReset` 标志

## Reference

- [issue 4070](https://github.com/VictoriaMetrics/VictoriaMetrics/issues/4070)
