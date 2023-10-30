# Requests & Limits

```bash
kubectl create -f myapp-pod-limits.yaml
```

```bash
kubectl top pods
```

```bash
kubectl top nodes
```

```bash
kubectl describe node kube-control1
kubectl describe node kube-node1
kubectl describe node kube-node2
kubectl describe node kube-node3
```

```bash
watch -n1 -d kubectl top pods
```

```bash
kubectl exec myapp-pod-lim -- sha256sum /dev/zero
```

