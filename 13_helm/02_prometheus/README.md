# Prometheus

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

```bash
kubectl create ns monitor
```

```bash
helm install prometheus prometheus-community/kube-prometheus-stack -n monitor -f prometheus-grafana.yaml
```

```bash
kubectl get svc -n monitor
```

> http://192.168.56.X

- ID: admin
- PWD: prom-operator

