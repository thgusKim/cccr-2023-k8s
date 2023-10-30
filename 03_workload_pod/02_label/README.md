# Label

```bash
kubectl get pods --show-labels
```

```bash
kubectl get pods -l env=dev
kubectl get pods -l env=dev,tier=frontend
kubectl get pods -l env=dev,tier!=backend
kubectl get pods -l env
kubectl get pods -l 'env in (dev,prod)'
kubectl get pods -A -l tier=control-plane,component=kube-apiserver
```

```bash
kubectl label pod myapp-pod-label version=0.1
kubectl label pod myapp-pod-label version=0.2 --overwrite
kubectl label pod myapp-pod-label version-
```
