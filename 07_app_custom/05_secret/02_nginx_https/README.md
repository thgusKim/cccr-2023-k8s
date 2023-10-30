# Nginx HTTPS

```bash
mkdir nginx-tls
```

```bash
openssl genrsa -out nginx-tls/nginx-tls.key 2048
```

```bash
openssl req -new -x509 -key nginx-tls/nginx-tls.key -out nginx-tls/nginx-tls.crt -days 365 -subj /CN=myapp.example.com
```

```bash
base64 nginx-tls/nginx-tls.crt -w 0
base64 nginx-tls/nginx-tls.key -w 0
```
> nginx-secret-https.yaml 

```bash
kubectl create -f .
```

```bash
kubectl port-forward nginx-pod-https 8443:443
```

```bash
curl -k https://localhost:8443
curl -k https://localhost:8443 -v
```

