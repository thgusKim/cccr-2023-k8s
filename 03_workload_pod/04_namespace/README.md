# Namespace

```bash
kubectl get namespaces
```

```bash
kubectl create -f myapp-pod.yaml -n kube-public
```

```bash
kubectl get pod myapp-pod -n kube-public
```

```bash
kubectl delete -f myapp-pod.yaml -n kube-public
```

```bash
kubectl create namespace development
```

```bash
kubectl create -f myapp-pod-ns.yaml
```

```bash
kubectl delete -f myapp-pod-ns.yaml
```

```bash
kubectl create namespace development
```

```bash
kubectl create -f qa-ns.yaml
```

```bash
kubectl delete -f qa-ns.yaml
```