# StatefulSet Basic

```bash
kubectl create -f myapp-sts.yaml -f myapp-svc-headless.yaml
```

```bash
watch -n1 -d kubectl get sts,pod
```

```bash
kubectl get svc,ep
```

```bash
kubectl scale sts myapp-sts --replicas=X
```

```bash
kubectl delete pod myapp-sts-1
```

```bash
kubectl run nettool --image ghcr.io/c1t1d0s7/network-multitool -it --rm

# host myapp-sts-0.myapp-svc-headless
# host myapp-sts-0.myapp-svc-headless.default.svc.cluster.local
# host myapp-sts-1.myapp-svc-headless
# host myapp-sts-1.myapp-svc-headless.default.svc.cluster.local
```
