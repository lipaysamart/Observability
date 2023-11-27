## Observability

>Build Your Kubernetes Monitoring System

![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=Prometheus&logoColor=white) ![Grafana](https://img.shields.io/badge/grafana-%23F46800.svg?style=for-the-badge&logo=grafana&logoColor=white) 	![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white) ![Prometheusalert](https://img.shields.io/badge/Prometheusalert-E6522C?style=for-the-badge&logoColor=white)

### K8S版本兼容

___

| kube-prometheus stack                                                                      | Kubernetes 1.21 | Kubernetes 1.22 | Kubernetes 1.23 | Kubernetes 1.24 | Kubernetes 1.25 |
| ------------------------------------------------------------------------------------------ | --------------- | --------------- | --------------- | --------------- | --------------- |
| [`release-0.9`](https://github.com/prometheus-operator/kube-prometheus/tree/release-0.9)   | ✔               | ✔               | ✗               | ✗               | ✗               |
| [`release-0.10`](https://github.com/prometheus-operator/kube-prometheus/tree/release-0.10) | ✗               | ✔               | ✔               | ✗               | ✗               |
| [`release-0.11`](https://github.com/prometheus-operator/kube-prometheus/tree/release-0.11) | ✗               | ✗               | ✔               | ✔               | ✗               |
| [`release-0.12`](https://github.com/prometheus-operator/kube-prometheus/tree/release-0.12) | ✗               | ✗               | ✗               | ✔               | ✔               |

### 目录结构

___

```
├─Alertmanager    # 告警平台
├─ExternalRules   # 扩展的告警规则
├─Grafana   # 数据展示
├─PrometheusAlert   # 告警通知，格式化模板
│  └─wx-tpl   # 告警模板简单示例
├─Release-0.12    # Kube-Prometheus-Operator 项目版本
│  └─manifests
│      └─setup
└─ScrapMonitor     # 监控抓取配置
    └─exporter    # Export 配置  
```  

### 项目配置

___

>与原生的 **Kube-Prometheus** 不同。在原生项目的基础上增加了一些配置，和完善避免部署原生时踩过的一些坑。其中一些默认值需要进行修改，例如 `storageClass`, `ingress` 等资源。
>>每个文件内都有清晰的注解说明，请仔细查看。

|          Explain          |                     Path                      |
| :-----------------------: | :-------------------------------------------: |
|      Prometheus 配置      |     manifests/prometheus-prometheus.yaml      |
|       Blackbox 配置       | manifests/blackboxExporter-configuration.yaml |
|       自动发现配置        |     manifests/prometheus-additional.yaml      |
|         RBAC 配置          |     manifests/prometheus-clusterRole.yaml     |
| Alertmanager 告警路由配置 |     Alertmanager/alertmanager-secret.yaml     |
|       Grafana 配置        |          Grafana/grafana-config.yaml          |
|   Prometheusalert 配置    |  Prometheusalert/prometheusAlert-config.yaml  |

### 项目部署

____

>

```
$ kubectl create -f manifests/setup
$ kubectl create -f Grafana/ -f PrometheusAlert/ -f Alertmanager/
$ kubectl create -f manifests/
```

### TIPS

___

#### 配置多个 **AlertmanagerConfig**

如果你有多个告警接收人和路由配置时，不想挤压在一个文件内，可以使用这个Kind资源 `AlertmanagerConfig`, 帮助我们创建管理告警配置。

#### 配置外部 **Alertmanager**

如果你有多套集群发送到一个 **Alertmanager** 时需要添加以下参数 (文件默认在 manifests 下 `alert-center-external-secert.yaml`):
```yaml
kubectl apply -f - <<-EOF
apiVersion: v1
kind: Secret
metadata:
  name: alert-center-external
  namespace: monitoring
stringData:
  alert-config: |-
    - path_prefix: /
      scheme: https
      timeout: 10s
      api_version: v1
      static_configs:
        - targets: ['alertmanager.k8s.local']
EOF
```
```yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: k8s
  namespace: monitoring
spec:
  # 添加这部分内容
  additionalAlertManagerConfigs:
    key: alert-config
    name: alert-center-external
```

#### 配置自定义监控

>在 `Pormetheus-Operator` 中给我们提供了 `ServiceMonitor` 对象,我们可以通过 `ServiceMonitor` 来关联 `Metrics` 数据接口的 `Service` 对象。
>>常见的监控对象在 `PROMETHEUS/scrape`

#### 配置自动发现

>通过添加注解 `prometheus.io/scrape=true`来进行 `service/pod` 发现并进行自动监控,此前我们已经在Prometheus中进行配置 `Prometheus.spec.additionalScrapeConfigs`。

#### 配置告警规则

>想要自定义一个报警规则，只需要创建一个能够被 `prometheus` 对象匹配的 `PrometheusRule` 对象即可。
>>扩展的告警规则在 `ExternalRules/`。

#### 监控 **kube_proxy** 组件

```
# kube-proxy默认监听的地址是127.0.0.1:10249，如果你是1.26的版本则默认不开启。
# 修改监听的端口，按如下方法:
k edit cm kube-proxy -nkube-system
# 将metricsBindAddress这段修改成metricsBindAddress: 0.0.0.0:10249
# 重启kube-proxy:
k get pods -nkube-system | grep kube-proxy | awk '{print $1}' | xargs kubectl delete pods -n kube-system
```

#### 监控 **etcd** 组件

```
# 在 master上修改 /etc/kubernetes/manifests/etcd.yaml 文件
# 将 command 中修改 --listen-metrics-urls=http://127.0.0.1:2381
--listen-metrics-urls=http://0.0.0.0:2381
```

#### 监控 **kube_scheduler** 组件和 **kube_controller_manager** 组件

```
# 在 master 路径下 /etc/kubernetes/manifests 找到这两个文件
# 将 command 中修改 --bind-address=127.0.0.1
--bind-address=0.0.0.0
```

#### 清理 **Prometheus-Operator**

```
k delete -f manifests/
k delete -f manifests/setup
```

### TroubleShooting

___

#### 多副本采集数据不一致

>多副本的情况下正常来说请求是会去轮询访问后端的两个 `Prometheus` 实例, 这样可能就会导致每次请求到的 `Prometheus` 数据都不一样，解决这个问题的方法则可以在创建 `Service` 的时候添加 `sessionAffinity: ClientIP` 这样的属性，然后会根据 `ClientIP` 来做 `session` 亲和性。所以不用担心请求会到不同的副本上去


#### 无法通过 **statefulSet** 对象来删除 **Prometheus** 或 **Alertmanager**

>这是因为它们由 **Operator** 控制

```sh
# 查看Prometheus对象下定义的资源
k get prometheus -nmonitoring

NAME   VERSION   DESIRED   READY   RECONCILED   AVAILABLE   AGE
k8s    2.46.0    2         2       True         True        3h54m

# 删除Prometheus对象
k delete prometheus k8s -nmonitoring 
```