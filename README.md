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
2. 
