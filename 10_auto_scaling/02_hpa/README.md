# HPA

```bash
kubectl create -f myapp-deploy-hpa.yaml -f myapp-hpa-cpu.yaml
```

```bash
watch -n1 -d kubectl get hpa
```

```bash
watch -n1 -d kubectl top pods
```

```bash
kubectl exec myapp-deploy-hpa-aaaa -- sha256sum /dev/zero
kubectl exec myapp-deploy-hpa-bbbb -- sha256sum /dev/zero
```

