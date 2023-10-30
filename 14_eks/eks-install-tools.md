# EKS 관련 도구 설치

## 1. Windows

### 1) Chocolatey 패키지 관리자

Powershell 관리자 권한으로 설치

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

### 2) awscli

```bash
choco install awscli
```

### 3) aws-iam-authenticator

```bash
choco install aws-iam-authenticator
```

### 4) kubectl

```bash
choco install kubernetes-cli --version=1.27.X
```

### 5) eksctl

```bash
choco install eksctl
```

### 6) helm

```bash
choco install kubernetes-helm
```

## 2. MacOS

### 1) Homebrew 패키지 관리자

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 2) awscli

```bash
brew install awscli
```

### 3) aws-iam-authenticator

```bash
brew install aws-iam-authenticator
```

### 4) kubectl

```bash
curl -LO "https://dl.k8s.io/release/v1.27.X/bin/darwin/arm64/kubectl"
```

```bash
sudo install kubectl /usr/local/bin
```

### 5) helm

```bash
brew install helm
```
