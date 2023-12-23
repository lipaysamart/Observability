## 规范

- 全局抓取间隔是超时时间的 三倍, ( VM-Agent 配置 )

```sh
# VM-Agent
  scrapeTimeout: "20s"
  scrapeInterval: "60s"
```
- HTTP 和 TCP 探测超时统一为 8s, ICMP 为 2s

```sh
...
    modules:
      http_2xx:
        ...
        prober: icmp
        timeout: 8s
      icmp:  # ping 检测服务器的存活
        ...
        prober: icmp
        timeout: 2s
```
- PodScrape 抓取间隔统一 45s 超时 15s

```sh
  podMetricsEndpoints:
    - port: web
      path: /-/metrics
      scheme: http
      interval: 15s
```
- ProbeScrape 抓取间隔统一 30s 超时 10s

- Ingress 统一域名访问路径

|       Path       |      Explain      |
| :--------------: | :---------------: |
|       `/`        | 查看 agent target |
| `/select/0/vmui` |    查询 metric    |
|    `/alerts`     | 管理 alertmanager |
|    `/vmalert`    |   查看告警规则    |

## 安装 CRD

```sh
helm upgrade --install vm-operator vm/victoria-metrics-operator --version 0.27.9 -f values.yaml -n vm-operator --create-namespace
```

## 安装 VM-Cluster

```sh
k apply -f Ci/vm-cluster.yaml
```

## 安装 VM-Agent

```sh
k apply -f Components/vm-agent.yaml
```

### 抓取 Cadvisor

>VMNodeScrape CRD 提供了抓取 kubernetes 节点指标的发现机制，对于 node_export 监控非常有用
```sh
k apply -f ScrapMonitor/vm-node-scrape.yaml
```

### 抓取 Probe

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
