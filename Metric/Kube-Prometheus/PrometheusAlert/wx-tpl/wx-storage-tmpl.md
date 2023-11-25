{{ $var := .externalURL}}{{ range $k,$v:=.alerts -}}
{{ if eq $v.status "resolved" -}}
## [Prometheus恢复信息]($v.generatorURL)💨

##### 🌟<font color="#02b340">【AlertName】✅</font>[{{$v.labels.alertname}}]({{$var}})✅{{ if $v.labels.severity }}
##### 🌟<font color="#02b340">【Level】</font>
{{- if eq $v.labels.severity "info" }}info
{{- else if eq $v.labels.severity "warning" }}warning
{{- else if eq $v.labels.severity "critical" }}critical
{{- else if eq $v.labels.severity "error" }}error
{{ else }}{{ $v.labels.severity }}
{{ end -}}
{{ end }}
##### 🌟<font color="#02b340">【Status】</font><font color="#67C23A">已恢复</font>
##### 🌟<font color="#02b340">【StartTime】</font>{{GetCSTtime $v.startsAt}}
##### 🌟<font color="#02b340">【EndTime】</font>{{GetCSTtime $v.endsAt}}
##### 🌟<font color="#02b340">【Cluster】</font>{{$v.labels.cluster}}
##### 🌟<font color="#02b340">【Namespace】</font>{{$v.labels.namespace}}
##### 🌟<font color="#02b340">【Claims】</font>{{$v.labels.persistentvolumeclaim}}

**<font color="#02b340">{{$v.annotations.description}}</font>**

## ✨[点我查看当前状态]({{$v.generatorURL}})✨

{{- else -}}
## [Prometheus告警信息]($v.generatorURL)💨

##### 🌟<font color="#FF0000">【AlertName】🔔</font>[{{$v.labels.alertname}}]({{$var}})🔔{{ if $v.labels.severity }}
##### 🌟<font color="#FF0000">【Level】</font>
{{- if eq $v.labels.severity "info"}}info
{{- else if eq $v.labels.severity "warning"}}warning 🔥
{{- else if eq $v.labels.severity "critical"}}critical 💔💔
{{- else if eq $v.labels.severity "error"}}error 🔥🔥❌
{{ else }}{{ $v.labels.severity }}
{{ end -}}
{{ end }}
##### 🌟<font color="#FF0000">【Status】</font><font color="#E6A23C">需要处理</font>
##### 🌟<font color="#FF0000">【StartTime】</font>{{GetCSTtime $v.startsAt}}
##### 🌟<font color="#FF0000">【Cluster】</font>{{$v.labels.cluster}}
##### 🌟<font color="#FF0000">【Namespace】</font>{{$v.labels.namespace}}
##### 🌟<font color="#FF0000">【Claims】</font>{{$v.labels.persistentvolumeclaim}}

**<font color="#E6A23C">{{$v.annotations.description}}</font>**

## ✨[点我去屏蔽告警]({{$var}})✨
{{ end -}}
{{ end }}