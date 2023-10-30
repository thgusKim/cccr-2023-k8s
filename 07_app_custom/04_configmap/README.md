# ConfigMap

```bash
kubectl create -f nginx-cm-compress.yaml -f nginx-pod-compress.yaml
```

```bash
kubectl port-forward nginx-pod-compress 8080:80
```

```bash
curl -I localhost:8080
curl -H "Accept-Encoding: gzip" -I localhost:8080
```
