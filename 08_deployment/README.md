# Deployment

## Monitoring

```bash
watch -n1 -d kubectl get deploy,rs,pod -l app=myapp-deploy
```

```bash
watch -n1 -d curl -s http://192.168.56.200
```

## Version 1 Deployment

```bash
kubectl apply -f myapp-deploy-v1.yaml
```

```bash
kubectl rollout status deploy myapp-deploy
```

```bash
kubectl rollout history deploy myapp-deploy
```

## From V1 to V2

```bash
kubectl set image deploy myapp-deploy myapp=ghcr.io/c1t1d0s7/go-myweb:v2.0
```

```bash
kubectl rollout status deploy myapp-deploy
```

```bash
kubectl rollout history deploy myapp-deploy
```

## From V2 to V3

```bash
kubectl apply -f myapp-deploy-v3.yaml
```

```bash
kubectl rollout status deploy myapp-deploy
```

```bash
kubectl rollout history deploy myapp-deploy
```

## From V3 to V2(Rollback)

```bash
kubectl rollout undo deploy myapp-deploy --to-revision 2
```

```bash
kubectl rollout status deploy myapp-deploy
```

```bash
kubectl rollout history deploy myapp-deploy
```
