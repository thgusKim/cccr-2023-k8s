# Kubernetes Dashboard

```bash
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
helm repo update
```

`dashboard-param.yaml`
```yaml
service:
  type: LoadBalancer
```

```bash
helm install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard -n kube-system -f dashboard-param.yaml
```

```bash
kubectl get all -n kube-system -l app.kubernetes.io/component=kubernetes-dashboard
```

```bash
kubectl get sa -n kube-system kubernetes-dashboard
```

`kubernetes-dashboard.yaml`
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: kubernetes-dashboard
    namespace: kube-system
```

```bash
kubectl create -f kubernetes-dashboard.yaml
```

```bash
kubectl create token -n kube-system kubernetes-dashboard
```

> https://192.168.56.X
