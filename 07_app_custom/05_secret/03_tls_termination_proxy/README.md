# TLS Termination Proxy

```bash
mkdir ingress-tls
```

```bash
openssl genrsa -out ingress-tls/ingress-tls.key 2048
```

```bash
openssl req -new -x509 -key ingress-tls/ingress-tls.key -out ingress-tls/ingress-tls.crt -days 365 -subj /CN=myapp.example.com
```

```bash
base64 ingress-tls/ingress-tls.crt -w 0
base64 ingress-tls/ingress-tls.key -w 0
```
> myapp-secret-https.yaml 

```bash
kubectl create -f .
```

```bash
curl --resolve myapp.example.com:443:192.168.56.21 -k https://myapp.example.com
curl --resolve myapp.example.com:443:192.168.56.21 -k https://myapp.example.com -v
```

