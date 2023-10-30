# Kubespray를 이용한 Production Ready Kubernetes 클러스터 배포

[Kubespray GitHub 저장소](https://github.com/kubernetes-sigs/kubespray)

작성날짜: 2018년 11월 30일  
업데이트: 2023년 10월 10일

> kubespray는 프로덕션 환경에 Kubernetes를 설치할 수 있는 배포방법

## 0. VM 시스템 구성

OS: Ubuntu 22.04 LTS(Jammy)

| Control Plane      | IP               | CPU | Memory |
|--------------------|------------------|-----|--------|
| kube-control1      | 192.168.56.11/24 | 2   | 3072MB |

| Node               | IP               | CPU | Memory |
|--------------------|------------------|-----|--------|
| kube-node1         | 192.168.56.21/24 | 2   | 2560MB |
| kube-node2         | 192.168.56.22/24 | 2   | 2560MB |
| kube-node3         | 192.168.56.23/24 | 2   | 2560MB |

## 1. Requirements

- Ansible, python-netaddr
- Jinja
- 인터넷 연결(도커 이미지 가져오기)
- IPv4 포워딩
- SSH 키 복사
- 배포 중 문제가 발생하지 않도록 방화벽 비활성
- 적절한 권한 상승(non-root 사용자인 경우, passwordless sudo 설정)

- Control Plane
  - Memory: 1500MB
- Node
  - Memory: 1024MB

### SSH 키 생성

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ''
```

### SSH 키 복사

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub vagrant@192.168.56.11
ssh-copy-id -i ~/.ssh/id_ed25519.pub vagrant@192.168.56.21
ssh-copy-id -i ~/.ssh/id_ed25519.pub vagrant@192.168.56.22
ssh-copy-id -i ~/.ssh/id_ed25519.pub vagrant@192.168.56.23
```

> vagrant 이미지의 기본 사용자 vagrant, 기본 패스워드 vagrant

### python3, pip, git 설치

```bash
sudo apt update  
sudo apt install -y python3 python3-pip git
```

## 2. Kubespray 배포

### kubespray Git 저장소 클론

```bash
cd ~
```

```bash
git clone --branch release-2.23 https://github.com/kubernetes-sigs/kubespray.git  
```

### 디렉토리 변경

```bash
cd kubespray
```

### requirements.txt 파일에서 의존성 확인 및 설치

```bash
sudo pip3 install -r requirements.txt  
```

### Ansible 인벤토리 준비

```bash
cp -rfp inventory/sample inventory/mycluster
```

### 인벤토리 수정

`inventory/mycluster/inventory.ini`

```ini
[all]  
kube-control1 ansible_host=192.168.56.11 ip=192.168.56.11
kube-node1    ansible_host=192.168.56.21 ip=192.168.56.21
kube-node2    ansible_host=192.168.56.22 ip=192.168.56.22
kube-node3    ansible_host=192.168.56.23 ip=192.168.56.23

[kube_control_plane]  
kube-control1 

[etcd]  
kube-control1  

[kube_node]  
kube-node1  
kube-node2
kube-node3  

[calico_rr]  

[k8s_cluster:children]  
kube_control_plane  
kube_node  
calico_rr
```

### 파라미터 확인 및 변경

`inventory/mycluster/group_vars/k8s_cluster/addons.yml`

```yaml
...
metrics_server_enabled: true
...
ingress_nginx_enabled: true
...
metallb_enabled: true
...
metallb_protocol: "layer2"
...
metallb_config:
  address_pools:
    primary:
      ip_range:
        - 192.168.56.200-192.168.56.209
      auto_assign: true
  ...
  layer2:
    - primary
...
```

`inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml`

```yaml
...
kube_proxy_strict_arp: true
...
```

### Ansible 통신 가능 확인

```bash
ansible all -i inventory/mycluster/inventory.ini -m ping

kube-control1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
kube-node1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
kube-node3 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
kube-node2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### (옵션) Apt 캐시 업데이트

```bash
ansible all -i inventory/mycluster/inventory.ini -m apt -a 'update_cache=yes' --become

kube-control1 | CHANGED => {
    "cache_update_time": 1584068822,
    "cache_updated": true,
    "changed": true
}
kube-node1 | CHANGED => {
    "cache_update_time": 1584068827,
    "cache_updated": true,
    "changed": true
}
kube-node3 | CHANGED => {
    "cache_update_time": 1584068826,
    "cache_updated": true,
    "changed": true
}
kube-node2 | CHANGED => {
    "cache_update_time": 1584068826,
    "cache_updated": true,
    "changed": true
}
```

### 플레이북 실행

```bash
ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -e kube_version=v1.27.X --become
```

> `kube_version`: 현재 Kubespray v2.23.0 버전의 경우 기본 Kubernetes 버전은 1.27.6

> 호스트 시스템 사양, VM 개수, VM 리소스 및 네트워크 성능에 따라 15~60분 소요

### 자격증명 가져오기

```bash
mkdir ~/.kube
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
```

### kubectl 명령 자동완성

```bash
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl
exec bash
```

### Kubernetes 클러스터 확인

```bash
kubectl get nodes
kubectl cluster-info
```
