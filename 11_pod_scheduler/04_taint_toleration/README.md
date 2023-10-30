# Taints & Toleration

```bash
kubectl get nodes kube-control1 -o jsonpath='{.spec.taints}'
kubectl get nodes kube-node1 -o jsonpath='{.spec.taints}'
kubectl get nodes kube-node2 -o jsonpath='{.spec.taints}'
kubectl get nodes kube-node3 -o jsonpath='{.spec.taints}'
```

```bash
kubectl taint node kube-node1 env=production:NoSchedule
```

