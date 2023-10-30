# x.509

```bash
openssl x509 -in /etc/kubernetes/pki/ca.crt -text -noout
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

```bash
openssl genrsa -out myuser.key 2048
```

```bash
openssl req -new -key myuser.key -out myuser.csr -subj "/CN=myuser"
```

```bash
base64 myuser.csr -w 0
```

`myuser-csr.yaml`
```yaml
...
spec:
  request: <PASTE>
...
```

```bash
kubectl create -f myuser-csr.yaml
```

```bash
kubectl get csr
```

```bash
kubectl certifiacte approve myuser
```

```bash
kubectl get csr
```

```bash
kubectl get csr myuser -o jsonpath='{.status.certificate}' | base64 -d > myuser.crt
```

```bash
openssl x509 -in myuser.crt -text -noout
```

```bash
kubectl create -f role-admin-pod.yaml -f rbind-myuser.yaml
```

```bash
kubectl config view
```

```bash
kubectl config get-clusters
kubectl config get-users
kubectl config get-contexts
```

```bash
kubectl config set-credentials myuser --client-certificate=myuser.crt --client-key=myuser.key --embed-certs
```

```bash
kubectl config get-users
kubectl config view
```

```bash
kubectl config set-context myuser@cluster.local --cluster=cluster.local --user=myuser --namespace=default
```

```bash
kubectl config get-contexts
kubectl config view
```

```bash
kubectl config use-context myuser@cluster.local
```

