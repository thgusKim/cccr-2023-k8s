# NFS Subdir External Provisioner 구성

저장소 가져오기

```bash
cd ~
git clone https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner
```

디렉토리 이동

```bash
cd ~/nfs-subdir-external-provisioner/deploy/
```

파일 수정

`deployments.yaml`
```bash
...
      containers:
        ...
          env:
            - name: PROVISIONER_NAME
              value: k8s-sigs.io/nfs-subdir-external-provisioner
            - name: NFS_SERVER
              value: 192.168.56.11
            - name: NFS_PATH
              value: /srv/nfs-volume
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.56.11
            path: /srv/nfs-volume
```

리소스 배포

```bash
kubectl apply -k .
```

기본 스토리지클래스 구성

```bash
kubectl patch storageclasses.storage.k8s.io nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```
