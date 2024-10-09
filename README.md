# Grafana

## Loki

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

```bash
kubectl create namespace grafana
```

```bash
helm install loki -n grafana grafana/loki \
  --set minio.enabled=true \
  --set write.replicas=1 \
  --set read.replicas=1 \
  --set backend.replicas=1 \
  --set loki.auth_enabled=false \
  --set loki.commonConfig.replication_factor=1 \
  --set loki.useTestSchema=true
```

```bash
helm install promtail -n grafana grafana/promtail
```

```bash
helm install grafana -n grafana grafana/grafana --set service.type=LoadBalancer
```

```bash
kubectl get secret --namespace grafana grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

```bash
kubectl apply -f https://k8s.io/examples/debug/counter-pod.yaml
```
