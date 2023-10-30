# eksctl을 이용한 AWS EKS 관리

## 1. AWS EKS 클러스터

### 1) AWS EKS 클러스터 생성

EKS 클러스터 생성

```
eksctl create cluster --name myeks --nodegroup-name mynodes
```

EKS 클러스터 확인

```
eksctl get clusters
```

[EKS - 클러스터](https://ap-northeast-2.console.aws.amazon.com/eks/home?region=ap-northeast-2#/clusters) 콘솔에서 확인

### 2) 노드 그룹 관리

노드 확인

```
kubectl get nodes
```

```
kubectl get nodes -o wide
```

- INTERNAL-IP: 노드의 내부 네트워크 IP
- EXTERNAL-IP: 노드의 외부 네트워크 IP
  - 노드를 외부에 직접 노출 시키는 것을 보안상 문제를 야기할 수 있어 권장하지 않음

```
eksctl get nodegroup --cluster myeks
```

노드 그룹 스케일링

```
eksctl scale nodegroup --cluster myeks --name mynodes --nodes 3 --nodes-max 3
```

노드 그룹 확인

```
eksctl get nodegroup --cluster myeks
```

노드 그룹 삭제

```
eksctl delete nodegroup --cluster myeks --name mynodes
```

노드 확인

```
kubectl get nodes -o wide
```

노드 그룹 확인

```
eksctl get nodegroup --cluster myeks
```

새로운 노드 그룹 생성

```
eksctl create nodegroup --cluster myeks --name myeks-ng --node-type t3.medium --node-private-networking --nodes-min 1 --nodes 2 --nodes-max 3
```

노드 그룹 확인

```
eksctl get nodegroup --cluster myeks
```

노드 확인

```
kubectl get nodes -o wide
```

- EXTERNAL-IP: 노드는 사설 네트워크에 배치하도록 구성해 EXTERNAL-IP가 할당되지 않음

```
kubectl get nodes --show-labels
```

노드에 설정된 레이블:

- alpha.eksctl.io/cluster-name=myeks
- alpha.eksctl.io/nodegroup-name=myeks-ng
- beta.kubernetes.io/arch=amd64
- beta.kubernetes.io/instance-type=t3.medium
- beta.kubernetes.io/os=linux
- eks.amazonaws.com/capacityType=ON_DEMAND
- eks.amazonaws.com/nodegroup-image=ami-08f9b107f35946d3d
- eks.amazonaws.com/nodegroup=myeks-ng
- eks.amazonaws.com/sourceLaunchTemplateId=lt-0b177ad29457e7aa2
- eks.amazonaws.com/sourceLaunchTemplateVersion=1
- failure-domain.beta.kubernetes.io/region=ap-northeast-2
- failure-domain.beta.kubernetes.io/zone=ap-northeast-2c
- k8s.io/cloud-provider-aws=5553ae84a0d29114870f67bbabd07d44
- kubernetes.io/arch=amd64
- kubernetes.io/hostname=ip-192-168-101-136.ap-northeast-2.compute.internal
- kubernetes.io/os=linux
- node.kubernetes.io/instance-type=t3.medium
- topology.kubernetes.io/region=ap-northeast-2
- topology.kubernetes.io/zone=ap-northeast-2c

### 3) 기본 Addons 추가

> 참고: EKS에서 지원되는 애드온
> 
> https://docs.aws.amazon.com/eks/latest/userguide/eks-add-ons.html

현재 클러스터에 설치된 애드온 확인

```
eksctl get addon --cluster myeks
```

EKS에서 지원하는 애드온 목록 및 이름 확인

```
eksctl utils describe-addon-versions --cluster myeks | grep AddonName
```

kube-system 네임스페이스의 파드 목록

```
kubectl get pods -n kube-system
```

CoreDNS, kube-proxy, VPC-CNI는 설치된 애드온 목록에는 없지만 기본적으로 배포됨

애드온 버전 업데이트 등 애드온 관리를 위해 설치하는 것을 권장

kube-proxy 애드온 설치

```
eksctl create addon --cluster myeks --name kube-proxy
```

AWS IAM 사용자 및 인증을 위해 OpenID Connect 자격 증명 공급자를 연결함

```
eksctl utils associate-iam-oidc-provider --cluster=myeks --approve
```

CoreDNS 애드온 설치

```
eksctl create addon --cluster myeks --name coredns
```

VPC-CNI 애드온 설치

```
eksctl create addon --cluster myeks --name vpc-cni
```

설치된 애드온 확인

```
eksctl get addon --cluster myeks
```

## 2. AWS Load Balancer Controller

### 1) 서비스 및 인그레스 리소스 생성

EKS에서 로드밸런서 타입의 서비스 및 인그레스는 ELB로 구현됨

#### (1) 로드밸런서 타입 서비스

로드밸런서 타입의 서비스 및 디플로이먼트 리소스 생성

```
kubectl apply -f resource-manifest/svc/
```

서비스 확인

```
kubectl get svc
```

[ELB](https://console.aws.amazon.com/ec2/#LoadBalancers:) 확인

- 로드 밸런서 유형: Classic
- 체계: Internet-facing

리소스 삭제

```
kubectl delete -f resource-manifest/svc/
```

#### (2) 인그레스 리소스

인그레스 리소스 생성

```
kubectl apply -f resource-manifest/ing/
```

인그레스 리소스 확인

```
kubectl get ingress
```

```
kubectl describe ingress myapp-ing
```

인그레스 컨트롤러가 없어 생성되지 않음

인그레스 리소스 삭제

```
kubectl delete -f resource-manifest/ing/
```

### 2) AWS Load Balancer Controller 설치

> 참고: AWS Load Balancer Controller 추가 기능 설치
> 
> https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html
> https://kubernetes-sigs.github.io/aws-load-balancer-controller/

AWS Load Balacer Controller는 EKS에서 AWS ELB를 관리함

- Ingress: ALB(Application Load Balancer)
- LoadBalancer 타입의 Service: NLB(Network Load Balancer)

설치 전 필수 조건:

- EKS 클러스터
- 클러스터에 IAM OIDC 프로바이더 연결
- VPC-CNI, kube-proxy, CoreDNS 애드온 추가

사용자 대신 ELB 관련 API를 호출할 수 있는 IAM 정책 다운로드

```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```

정책 생성

```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

> 참고
>
> 이미 존재할 수도 있음

계정 ID 확인

```
aws sts get-caller-identity
```

결과에서 숫자로 된 `Account` 값을 확인

IAM 서비스 계정 생성

```
eksctl create iamserviceaccount \
  --cluster=myeks \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::111122223333:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

> 중요
> 
> `--attach-policy-arn` 옵션의 `111122223333` 값을 앞서 확인한 자신의 Account 번호로 대치

차트 저장소 추가

```
helm repo add eks https://aws.github.io/eks-charts
```

차트 목록 업데이트

```
helm repo update eks
```

`aws-load-balancer-controller` 차트 설치

```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=myeks \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller 
```

AWS Load Balancer Controller 관련 파드 리소스 확인

```
kubectl get po -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller
```

### 3) NLB를 사용하는 로드밸런서 서비스

> 참고
>
> https://docs.aws.amazon.com/eks/latest/userguide/network-load-balancing.html

NLB 구성을 위한 서비스 리소스의 어노테이션:

- service.beta.kubernetes.io/aws-load-balancer-type:
  - external: NLB 요청
- service.beta.kubernetes.io/aws-load-balancer-scheme:
  - internet-facing: 로드 밸런서를 퍼블릭 서브넷에 생성
  - internal: 로드 밸런서를 프라이빗 서브넷에 생성
- service.beta.kubernetes.io/aws-load-balancer-nlb-target-type:
  - instance: EC2 노드를 대상으로 함
  - ip: EC2 노드 또는 Fargate 파드의 IP를 대상으로 함

> 참고
>
> 리소스 생성 후 어노테이션을 수정하지 말것
> 수정이 필요한 경우 삭제 후 다시 생성

NLB를 요청하는 리소스 생성

```
kubectl apply -f resource-manifest/svc/nlb
```

서비스 리소스 확인

```
kubectl get service myapp-svc-nlb
```

[ELB](https://console.aws.amazon.com/ec2/#LoadBalancers:) 콘솔에서 확인

- 로드 밸런서 유형: Network
- 체계: Internet-facing

리소스 삭제

```
kubectl delete -f resource-manifest/svc/nlb/
```

### 4) ALB를 사용하는 인그레스

> 참고
>
> https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html

ALB 구성을 위한 인그레스 리소스의 어노테이션:

- kubernetes.io/ingress.class:
  - alb: ALB 요청
  - 어노테이션 대신 `ing.spec.ingressClassName: alb` 구성으로 대채 가능
- alb.ingress.kubernetes.io/scheme
  - internet-facing: 로드 밸런서를 퍼블릭 서브넷에 생성
  - internal: 로드 밸런서를 프라이빗 서브넷에 생성
- alb.ingress.kubernetes.io/target-type
  - instance: EC2 노드를 대상으로 함
  - ip: EC2 노드 또는 Fargate 파드의 IP를 대상으로 함

ALB를 요청하는 리소스 생성

```
kubectl apply -f resource-manifest/ing/alb/
```

서비스 리소스 확인

```
kubectl get ingress myapp-alb-ing
```

[ELB](https://console.aws.amazon.com/ec2/#LoadBalancers:) 콘솔에서 확인

- 로드 밸런서 유형: Application
- 체계: Internet-facing

리소스 삭제

```
kubectl delete -f resource-manifest/ing/alb/
```

## 3. AWS EBS 볼륨

AWS EBS는 접근 모드가 ReadWriteOnce(RWO)만 가능 함

### 1) EBS 볼륨 리소스 테스트

기본으로 제공되는 gp2 기본 스토리지클래스를 사용하는 볼륨 리소스 테스트

기본 스토리지클래스 확인

```
kubectl get sc

NAME            PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
gp2 (default)   kubernetes.io/aws-ebs   Delete          WaitForFirstConsumer   false                  137m
```

파드 및 PVC 리소스 생성

```
kubectl apply -f resource-manifest/ebs/
```

리소스 확인

```
kubectl get pod,pv,pvc
```

PV가 생성되지 않아 파드 및 PVC의 상태는 Pending

리소스 삭제

```
kubectl delete -f resource-manifest/ebs/
```

### 2) AWS EBS CSI 드라이버 애드온 설치

AWS EBS CSI 드라이버 애드온 설치

```
eksctl create addon --cluster myeks --name aws-ebs-csi-driver
```

AWS EBS CSI 드라이버 애드온 관련 파드 확인

```
kubectl get po -n kube-system -l app.kubernetes.io/name=aws-ebs-csi-driver
```

### 3) EBS 볼륨 리소스 테스트

파드 및 PVC 리소스 생성

```
kubectl apply -f resource-manifest/ebs/
```

리소스 확인

```
kubectl get pod,pv,pvc
```

PVC가 요청한 PV가 생성되며 파드에 마운트 됨

[볼륨](https://console.aws.amazon.com/ec2/#Volumes:) 콘솔에서 볼륨 확인

- 유형: gp2
- 크기: 1 GiB

리소스 삭제

```
kubectl delete -f resource-manifest/ebs/
```

## 4. AWS EFS 볼륨

AWS EFS는 관리형 NFS 파일 시스템으로 PV로 모든 접근 모드(ROX, RWO, RWX)로 구성 가능

### 1) AWS EFS CSI 드라이버 애드온 설치

AWS EFS CSI 드라이버 애드온 설치

```
eksctl create addon --cluster myeks --name aws-efs-csi-driver
```

AWS EFS CSI 드라이버 애드온 관련 파드 확인

```
kubectl get po -n kube-system -l app.kubernetes.io/name=aws-efs-csi-driver
```

### 2) EKS를 위한 AWS EFS 파일 시스템 생성

> 참고: aws 명령을 이용한 EFS 파일 시스템 생성
> 
> https://github.com/kubernetes-sigs/aws-efs-csi-driver/blob/master/docs/efs-create-filesystem.md

#### (1) EKS에서 사용하는 VPC ID 확인

```
vpc_id=$(aws eks describe-cluster --name myeks --query "cluster.resourcesVpcConfig.vpcId" --output text) && echo $vpc_id
```

#### (2) VPC의 CIDR 확인

```
cidr_range=$(aws ec2 describe-vpcs --vpc-ids $vpc_id --query "Vpcs[].CidrBlock" --output text) && echo $cidr_range
```

#### (3) EFS용 보안 그룹 생성 및 ID 확인

```
security_group_id=$(aws ec2 create-security-group --group-name myefs-sg --description "My EFS security group" --vpc-id $vpc_id --output text) && echo $security_group_id 
```

#### (4) 보안 그룹 인바운드 규칙 생성

```
aws ec2 authorize-security-group-ingress --group-id $security_group_id --protocol tcp --port 2049 --cidr $cidr_range
```

[보안 그룹](https://console.aws.amazon.com/ec2/#SecurityGroups:) 콘솔에서 `myefs-sg` 보안 그룹 및 인바운드 규칙 확인

#### (5) EFS 파일 시스템 생성 및 EFS 파일 시스템 ID 확인

```
file_system_id=$(aws efs create-file-system --region ap-northeast-2 --performance-mode generalPurpose --query 'FileSystemId' --output text) && echo $file_system_id
```

[EFS](https://ap-northeast-2.console.aws.amazon.com/efs?region=ap-northeast-2#/file-systems) 콘솔에서 생성된 EFS 파일 시스템 확인

#### (6) EFS 파일 시스템에 접근하기 위한 프라이빗 서브넷 확인

```
aws ec2 describe-subnets --filters "Name=vpc-id,Values=$vpc_id" --query 'Subnets[*].{SubnetId: SubnetId,AvailabilityZone: AvailabilityZone,CidrBlock: CidrBlock}' --output table
```

EKS의 EC2 워커 노드가 배치된 프라이빗 서브넷 ID 확인

- 퍼블릭 서브넷
  - 192.168.0.0/19
  - 192.168.32.0/19
  - 192.168.64.0/19
- 프라이빗 서브넷: 워커 노드가 배치됨
  - 192.168.96.0/19
  - 192.168.128.0/19
  - 192.168.160.0/19

#### (7) 프라이빗 서브넷 ID를 지정해 EFS 탑제 대상 설정

```
aws efs create-mount-target --file-system-id $file_system_id --security-groups $security_group_id --subnet-id subnet-1111111111111111
aws efs create-mount-target --file-system-id $file_system_id --security-groups $security_group_id --subnet-id subnet-2222222222222222
aws efs create-mount-target --file-system-id $file_system_id --security-groups $security_group_id --subnet-id subnet-3333333333333333
```

--subnet-id 옵션의 값을 각 프라이빗 서브넷 ID로 대치해 각각 실행

[EFS](https://ap-northeast-2.console.aws.amazon.com/efs?region=ap-northeast-2#/file-systems) -> 파일 시스템 ID 링크 선택 -> `네트워크` 탭에서 확인

#### (8) EFS 스토리지를 위한 스토리지클래스 생성

스토리지클래스 리소스에 사용할 EFS 파일 시스템 ID 확인

```
echo $file_system_id
```

efs-sc 스토리지 클래스

`resource-manifest/efs-sc/efs-sc.yaml`

```yml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
parameters:
  provisioningMode: efs-ap
  fileSystemId: fs-0123456789ABCDEFG
  directoryPerms: "700"
```

- sc.parameters.fileSystemId: 위에서 확인한 파일 시스템 값으로 대치

스토리지클래스 리소스 생성

```
kubectl apply -f resource-manifest/efs-sc/
```

스토리지클래스 리소스 확인

```
kubectl get sc efs-sc
```

### 3) EFS 스토리지클래스를 이용한 리소스 테스트

EFS 파일 시스템을 요청하는 PVC 및 Deployment 리소스 생성

```
kubectl apply -f resource-manifest/efs/
```

파드, PV, PVC 리소스 확인

```
kubectl get pod,pv,pvc
```

[EFS 액세스 포인트](https://ap-northeast-2.console.aws.amazon.com/efs?region=ap-northeast-2#/access-points) 콘솔에서 액세스 포인트가 생성되었는지 확인

각 파드가 동일한 파일시스템에 데이터를 쓸수 있고 같은 데이터를 읽는지 확인

```
kubectl exec efs-deploy-aaaaaaaaaa-bbbbb -- cat /data/out
kubectl exec efs-deploy-aaaaaaaaaa-ccccc -- cat /data/out
```

리소스 삭제

```
kubectl delete -f resource-manifest/efs/
```

### 4) EFS 파일시스템 및 보안 그룹 삭제

#### (1) EFS 파일시스템 탑제 대상 ID 확인

```
aws efs describe-mount-targets --file-system-id $file_system_id --output text --query "MountTargets[].MountTargetId"
```

#### (2) 각 탑제 대상 삭제

```
aws efs delete-mount-target --mount-target-id fsmt-11111111111111111
aws efs delete-mount-target --mount-target-id fsmt-22222222222222222
aws efs delete-mount-target --mount-target-id fsmt-33333333333333333
```

#### (3) EFS 파일시스템 삭제

```
aws efs delete-file-system --file-system-id $file_system_id
```

#### (4) EFS 파일시스템을 위한 보안 그룹 삭제

```
aws ec2 delete-security-group --group-id $security_group_id
```

## 5. 메트릭 서버

### 1) 리소스 사용량 확인

```
kubectl top pods -A
kubectl top nodes
```

`error: Metrics API not available` 오류 발생

### 2) 메트릭 서버 설치

> 참고
> 
> https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html

메트릭 서버 리소스 설치

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

메트릭 서버 관련 리소스 파드 확인

```
kubectl get po -n kube-system -l k8s-app=metrics-server
```

### 3) 리소스 확인

```
kubectl top pods -A
kubectl top nodes
```

## 6. Cloud Watch 로깅

### 1) Control Plane 로깅

EKS에서 컨트롤 플레인은 직접 관리할 수 없으며, 관련 로깅 기능을 활성화 해서 CloudWatch에서 확인 가능

모든 로그 타입 활성화

```
eksctl utils update-cluster-logging --enable-types=all --cluster=myeks --approve
```

> 참고: 로그 타입
> 
> - api
> - audit
> - autheticator
> - controllerManager
> - scheduler

1. [CloudWatch - 로그그룹](https://ap-northeast-2.console.aws.amazon.com/cloudwatch/home?region=ap-northeast-2#logsV2:log-groups)
2. `/aws/eks/<CLUSTE_NAME>/cluster` 로그 그룹에서 확인가능

### 2) Worker Node 및 Pod 로깅

> 참고: FluentBit를 이용해 CloudWatch로 로그 전송
> 
> https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-logs-FluentBit.html
> https://artifacthub.io/packages/helm/aws/aws-for-fluent-bit

#### (1) 권한 부여

EC2 인스턴스가 CloudWatch에 로그 그룹을 생성하고 이벤트 전송이 가능하도록 `CloudWatchAgentServerPolicy` 권한 부여

1. [EC2 - 인스턴스](https://ap-northeast-2.console.aws.amazon.com/ec2/home?region=ap-northeast-2#Instances:v=3;$case=tags:true%5C,client:false;$regex=tags:false%5C,client:false) 콘솔
2. EKS용 인스턴스 선택
3. 아래 `보안` 탭
   1. `IAM 역할` 링크 선택
4. 아래 `권한` 탭
   1. `권한 추가` -> `정책 연결`
   2. `CloudWatchAgentServerPolicy` 선택 -> `권한 추가` 선택

#### (2) AWS for Fluent Bit 차트 설치

차트 저장소 추가

```
helm repo add eks https://aws.github.io/eks-charts
```

차트 사용자화 파일 작성

`helm-values/aws-for-fluent-bit.yaml`

```yml
cloudWatchLogs:
  enabled: true
  region: ap-northeast-2
  logGroupName: /aws/eks/myeks/workload
  logGroupTemplate: /aws/eks/myeks/workload/$kubernetes['namespace_name']
  logStreamTemplate: $kubernetes['pod_name'].$kubernetes['container_name']
  autoCreateGroup: true
```

차트 설치

```
helm install aws-for-fluent-bit --namespace amazon-cloudwatch --create-namespace -f helm-values/aws-for-fluent-bit.yaml eks/aws-for-fluent-bit
```

설치된 리소스 확인

```
kubectl get pod -n amazon-cloudwatch -l app.kubernetes.io/instance=aws-for-fluent-bit
```

#### (3) 로그 확인

[CloudWatch - 로그 그룹](https://ap-northeast-2.console.aws.amazon.com/cloudwatch/home?region=ap-northeast-2#logsV2:log-groups) 콘솔 -> `/aws/eks/<CLUSTER_NAME>/workload/<NAMESPACE>`

default 네임스페이스에 파이값을 계산하는 잡 리소스 생성

```
kubectl apply -f resource-manifest/log/job.yaml
```

[CloudWatch - 로그 그룹](https://ap-northeast-2.console.aws.amazon.com/cloudwatch/home?region=ap-northeast-2#logsV2:log-groups) 콘솔 -> `/aws/eks/myeks/workload/default` 로그 그룹 -> `pi-abcde.pi` 로그 스트림 확인

> 참고: 로그 스트림 이름
>
> <POD_NAME>.<CONTAINER_NAME>

## 7. Container Insights 모니터링

> 참고
> 
> https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/deploy-container-insights-EKS.html
> https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-metrics.html
> https://artifacthub.io/packages/helm/aws/aws-cloudwatch-metrics

차트 저장소 추가

```
helm repo add eks https://aws.github.io/eks-charts
```

차트 설치

```
helm install aws-cloudwatch-metrics \
    --namespace amazon-cloudwatch --create-namespace eks/aws-cloudwatch-metrics \
    --set clusterName=myeks
```

[CloudWatch - 로그 그룹](https://ap-northeast-2.console.aws.amazon.com/cloudwatch/home?region=ap-northeast-2#logsV2:log-groups) 콘솔 -> `	/aws/containerinsights/<CLUSTER_NAME>/performance`

[CloudWatch - Container Insights](https://ap-northeast-2.console.aws.amazon.com/cloudwatch/home?region=ap-northeast-2#container-insights:infrastructure) 콘솔에서 성능 지표 확인 가능

## 8. Cluster Autoscaler

> 참고
> 
> https://github.com/kubernetes/autoscaler
> https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md

### 1) 클러스터 오토스케일러 설치

#### (1) EC2 인스턴스에 IAM 권한 추가

> 참고
> 
> https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md#full-cluster-autoscaler-features-policy-recommended

EC2 인스턴스에 AutoScalingGrooup을 찾을 수 있는 IAM 권한 추가

1. [EC2 - 인스턴스](https://ap-northeast-2.console.aws.amazon.com/ec2/home?region=ap-northeast-2#Instances:v=3;$case=tags:true%5C,client:false;$regex=tags:false%5C,client:false) 콘솔
2. EKS용 인스턴스 선택
3. 아래 `보안` 탭
   1. `IAM 역할` 링크 선택
4. 아래 `권한` 탭
   1. `권한 추가` -> `인라인 정책 생성`
   2. `JSON` 선택 후 기존 코드 삭제 후 아래 코드 붙혀넣기 -> `다음` 선택
   3. 정책 이름: `EKS_ASG_Autodiscovery` -> `정책 생성` 선택

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "autoscaling:DescribeAutoScalingGroups",
        "autoscaling:DescribeAutoScalingInstances",
        "autoscaling:DescribeLaunchConfigurations",
        "autoscaling:DescribeScalingActivities",
        "autoscaling:DescribeTags",
        "ec2:DescribeInstanceTypes",
        "ec2:DescribeLaunchTemplateVersions"
      ],
      "Resource": ["*"]
    },
    {
      "Effect": "Allow",
      "Action": [
        "autoscaling:SetDesiredCapacity",
        "autoscaling:TerminateInstanceInAutoScalingGroup",
        "ec2:DescribeImages",
        "ec2:GetInstanceTypesFromInstanceRequirements",
        "eks:DescribeNodegroup"
      ],
      "Resource": ["*"]
    }
  ]
}
```

#### (2) cluster-autoscaler 차트 설치

차트 저장소 추가

```
helm repo add cluster-autoscaler https://kubernetes.github.io/autoscaler
```

차트 설치

```
helm install cluster-autoscaler cluster-autoscaler/cluster-autoscaler \
    --namespace kube-system \
    --set autoDiscovery.clusterName=myeks \
    --set awsRegion=ap-northeast-2
```

클러스터 오토스케일러 관련 파드 리소스 확인

```
kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-cluster-autoscaler
```

### 3) 클러스터 오토스케일링을 위한 리소스 테스트

디플로이먼트 리소스 생성

```
kubectl apply -f resource-manifest/asg/
```

파드 확인

```
kubectl get pods -l app=myapp-deploy-asg
```

노드가 사용하고 있는 리소스 확인

```
kubectl get nodes
```

[CloudWatch - Container Insights](https://ap-northeast-2.console.aws.amazon.com/cloudwatch/home?region=ap-northeast-2#container-insights:infrastructure) 콘솔 -> 성능 모니터링 -> EKS Nodes -> 예약된 CPU 컴퓨팅 용량 확인

복제본 추가

```
kubectl scale deployment myapp-deploy-asg --replicas 15
```

노드가 추가 되었는지 확인

```
kubectl get nodes
```

노드 그룹 정보 확인

```
eksctl get nodegroups --cluster myeks --name myeks-ng
```

리소스 삭제

```
kubectl delete -f resource-manifest/asg/
```

## 9. Fargate

EC2 인스턴스 기반의 VM을 사용하지 않고 파드를 실행하는 Serverless 제품

### 1) Fargate 프로파일 생성

Fargate 프로파일 생성

```
eksctl create fargateprofile --cluster myeks --name myeks-fg --namespace dev --labels "env=fg"
```

파드를 Fargate에 생성하는 조건:

- 파드의 네임스페이스: dev
- 파드의 레이블(옵션): env=fg

위 조건이 맞으면 파드는 EC2 인스턴스 기반의 노드 그룹에 배포하지 않고 서버리스로 배포함

생성된 Fargate 프로파일 확인

```
eksctl get fargateprofile --cluster myeks
```

### 2) Fargate를 사용하는 리소스 테스트

네임스페이스 및 디플로이먼트 생성

```
kubectl apply -f resource-manifest/fg/
```

파드 및 노드 확인

```
kubectl get pod -n dev -l env=fg -o wide
kubectl get nodes
```

리소스 삭제

```
kubectl delete -f resource-manifest/fg/
```

파드 및 노드 확인

```
kubectl get pod -n dev -l env=fg -o wide
kubectl get nodes
```

## 10. 클러스터 삭제

eksctl 도구 이외에 생성한 리소스는 가능하면 수동으로 삭제

```
eksctl delete cluster --name myeks --disable-nodegroup-eviction --force
```

체크 리스트:

- EC2 인스턴스
- VPC: 서브넷, 보안그룹, 네트워크 인터페이스 등
- EBS 볼륨
- ELB
- EFS 파일시스템
- CloudWatch 로그 그룹
- EKS 클러스터

> 네트워크 인터페이스 관련 오류가 발생할 수 있음
