# Readiness Probe

```bash
kubectl exec <POD_NAME> -- <COMMAND>
```

```bash
kubectl exec myapp-rs-abcde -- touch /var/ready
```

```bash
kubectl exec -it myapp-rs-abcde -- sh
```

