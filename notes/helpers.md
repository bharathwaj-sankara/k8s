# Helper commands


## Docker Related Helpers

### List all repos in registry

https://`{{ registry }}`/v2/_catalog

### List all tags for an image

https://`{{ registry }}`/v2/`{{ repo-name }}`/tags/list

## Argento Build Helpers

### Build Argento

```
make build -j5 CLUSTER=bh-abs-metric1 K8S_CLUSTER=dev TO_CISCO=true
```

### Deploy Argento

```
make deploy TAG=wsankara.1571531322 -j5 CLUSTER=bh-abs-metric1 K8S_CLUSTER=dev TO_CISCO=1
```

### Delete Argento App

```
make purge  CLUSTER=bh-abs-metric1 K8S_CLUSTER=dev
```

### Delete Helm chart for prometheus-operator:

```
helm del --purge k8s-monitoring
kubectl delete crd prometheusrules.monitoring.coreos.com
kubectl delete crd servicemonitors.monitoring.coreos.com
kubectl delete crd alertmanagers.monitoring.coreos.com
kubectl delete crd alertmanagers.monitoring.coreos.com
kubectl delete crd prometheuses.monitoring.coreos.com
kubectl delete crd podmonitors.monitoring.coreos.com
kubectl get crd
```

### Install helm chart

```
helm install --name k8s-monitoring --namespace k8s-monitoring stable/prometheus-operator --version=6.7.3 --values=prom-upgrade.yaml
```

## Access Related Helpers

### Port-forward promethues dashboard 

```
kubectl -n k8s-monitoring port-forward svc/k8s-monitoring-prometheus-prometheus 9090:9090
```
