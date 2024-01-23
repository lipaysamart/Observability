## Observability

>Build Your Kubernetes Monitoring System

![Grafana](https://img.shields.io/badge/Grafana-%23F46800?logo=grafana&logoColor=white) ![Redis](https://img.shields.io/badge/Redis-%23DC382D?logo=redis&logoColor=white) ![Minio](https://img.shields.io/badge/Minio-%23C72E49?logo=minio&logoColor=white) ![Loki](https://img.shields.io/badge/Loki-Grafana-%23F46800) ![Promtail](https://img.shields.io/badge/Promtail-Grafana-%23F46800) 

### 目录结构

___
```sh
├─Export    # 日志类的 Export
│  └─k8sEvenExport
├─Loki    # 日志记录引擎，负责存储日志和处理查询
├─Minio   # 代理，负责收集日志并将其发送给 loki
├─Promtail    # 代理，负责收集日志并将其发送给 loki
├─Redis   # 缓存数据库
└─Rules   # 告警规则
```  

### 项目配置

___
>相关文件是由 `helm` 渲染 `grafana/loki-simple-scalable` 仓库后转换出来的 `yaml` 文件。对比原生 `helm` 安装增加了一些配置，和完善避免部署原生时踩过的坑。
>>每个文件内都有清晰的注解说明，请仔细查看。

|    Explain    |          Path           |
| :-----------: | :---------------------: |
|   Loki 配置   |  Loki/loki-config.yaml  |
| Promtail 配置 | Promtail/configmap.yaml |

### 项目部署

___
>`Loki` 使用**读写分离模式部署**, 这种模式下每天可承载的日志量约 100GB，还可以通过横向扩展副本数来处理 `TB` 级的日志数据
>>在这种模式下，Loki 的组件微服务被绑定到两个目标中：`-target=read` 和 `-target=write`
>>`Loki` 前面还需有一个负载均衡器，它将 `/loki/api/v1/push` 流量路由到写入节点，所有其他请求都转到读取节点，流量以循环方式发送。
>>>*Read* 节点数量不能超过 *Write* 节点数量

```sh
k create -f Minio/ -f Redis/
k create -f Loki/
k create -f Promtail/
k create -f Grafana/
```

### TIPS

___

#### 清理 **Loki**

```sh
k delete -f Loki/
k delete -f Promtail/
k delete -f Minio/ -f Redis/
k delete -f Grafana/
```

#### 配置 ruler 组件

>将 `rule` 规则挂载进 Loki `/var/loki/rules` 目录下
>>- 单租户 `rules/fake/xxx.yaml`
>>- 多租户 `rules/<tenant_id>/xxx.yaml`

```yaml
    ...
          volumeMounts:
            - name: example-rule
              mountPath: /etc/loki/rules/fake  # 挂载到/etc/loki/rules/下fake目录中，
        ...
      volumes:
        - name: example-rule                 # 添加告警规则 configmap
          configMap:
            defaultMode: 0640
            name: fake-rule
```

> 查看 loki 日志，日志关键字 

```
$ k -nmonitoring logs -f loki-read-0 | grep rule
{"caller":"mapper.go:47","level":"info","msg":"cleaning up mapped rules directory","path":"/tmp/rules","ts":"2024-01-22T07:10:51.773495269Z"}
{"caller":"module_service.go:82","level":"info","module":"ruler","msg":"initialising","ts":"2024-01-22T07:10:53.055784043Z"}
{"caller":"ruler.go:499","level":"info","msg":"ruler up and running","ts":"2024-01-22T07:10:53.055831252Z"}
```

### TroubleShooting

___
