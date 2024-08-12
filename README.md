# grafana-alert-cpu-and-memory
Grafana alerts provide real-time notifications based on the metrics and data visualized in your Grafana dashboards. These alerts are essential for proactively managing and responding to issues before they impact your systems.


Grafana Alert Queries:

1. POD Availability Alert
```
A: sum(kube_deployment_spec_replicas{namespace=~"<namespace name>"}) by (deployment,namespace,node)
B: sum(kube_deployment_status_replicas_available{namespace=~"<namespace name>"}) by (deployment,namespace,node)
C: reduce(A)
D: reduce(B)
E: Math ( $C - $D > 0 )

Annotations:
<b>Available/Desired</b> POD count for <b>{{ $labels.namespace }}/{{ $labels.deployment }}</b> is <b>{{ $values.D.Value }}/{{ $values.C.Value }}</b>
```
2. POD Degraded State Alert
```
A: min_over_time(sum by (namespace,pod,phase,node) (kube_pod_status_phase{phase!~"Running",namespace=~"<namespace name>"})[5m:1m]) > 0
B: Reduce (A)
C: Threshold (B) > 0

Annotation:
POD <b>{{ $labels.namespace }} / {{ $labels.pod }}</b>  was in <b>{{ $labels.phase }}</b> state.
```
3. Pod CrashLoopBack Alert
```
A: max_over_time(kube_pod_container_status_waiting_reason{reason=~"CrashLoopBackOff",namespace=~"<namespace name>"}[2m])
B: Reduce (A)
C: Threshold (B) > 0

Annotations:

POD <b>{{ $labels.namespace }} / {{ $labels.pod }} / {{ $labels.container }}</b> is in <b>{{ $labels.reason}}</b> state.
```
4. POD CPU Usage Alert
```
A: sum (rate (container_cpu_usage_seconds_total{image!="",kubernetes_io_hostname=~"^.*$",namespace=~"<namespace name>",container!~""}[10m])) by (pod,namespace)
B: Reduce (A)
C: Threshold (B) > 2.1

Annotations:
<b>Current CPU Usage</b> of Pod <b>{{ $labels.namespace }} / {{ $labels.pod }}</b> is <b>{{ printf "%.2f"  $values.B.Value  }} core</b> and is  exceeding the limit of <b>2 core.</b>
 
```
5. POD Memory Usage Alert
```
A: sum(container_memory_working_set_bytes{namespace=~"<namespace name>",container!="",image!="",kubernetes_io_hostname=~"^.*$"}) by (pod,namespace) / (1024*1024*1024)
B: Reduce (A)
C: Threshold (B) > 5.1

Annotations:
<b>Current Memory Usage</b> of Pod <b>{{ $labels.namespace }} / {{ $labels.pod }}</b> is <b>{{ printf "%.2f"  $values.B.Value  }} GB</b> and is  exceeding the limit of <b>5 GB.</b>
```
