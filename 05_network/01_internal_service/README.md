# Internal Service

```bash
kubectl run nettool -it --image ghcr.io/c1t1d0s7/network-multitool --rm
```

```bash
> host myapp-svc.default.svc.cluster.local
```

```bash
> curl http://myapp-svc.default.svc.cluster.local
> curl http://myapp-svc
```
