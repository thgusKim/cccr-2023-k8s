# Vagrant 설치 및 사용법

[Vagrant 공식 사이트](https://www.vagrantup.com/)

## 1. 패키지 관리자 및 Vagrant 패키지 설치

### 1) Windows

#### Chocolatey 패키지 관리자 설치

https://chocolatey.org/install

- PowerShell v2+
- .NET Framework 4.8+

Powershell을 관리자 권한으로 실행

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

#### Vagrant 설치

```
choco install vagrant
```

### 2) macOS

#### Homebrew 패키지 관리자 설치

https://brew.sh/index_ko

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

#### Vagrant 설치

```
brew install vagrant
```

### 3) Debian/Ubuntu

#### 패키지 저장소 추가

```
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
```

```
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
```

#### Vagrant 설치

```
sudo apt update && sudo apt install vagrant
```

## 2. VirtualBox 다운로드 및 설치

### 1) 설치 파일 및 패키지

https://www.virtualbox.org/wiki/Downloads

### 2) Windows

```
choco install virtualbox virtualbox-guest-additions-guest.install
```

### 3) macOS

```
brew install virtualbox virtualbox-extension-pack
```

### 4) Linux - Ubuntu

> https://www.virtualbox.org/wiki/Linux_Downloads

```
echo -e "deb [arch=amd64] https://download.virtualbox.org/virtualbox/debian bionic contrib" | sudo tee -a /etc/apt/sources.list
```

```
wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
```

```
wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -
```

```
sudo apt-get update
```

```
sudo apt-get install virtualbox virtualbox-guest-utils virtualbox-ext-pack
```

## 3. Vagrant 사용법

### 1) vagrant 디렉토리 생성

```
cd ~ 
mkdir test
cd test
```

### 2) vagrant 초기화

```
vagrant init ubuntu/jammy64
```

### 3) VM 배포

```
vagrant up [VM]
```

### 4) VM 상태확인

```
vagrant status [VM]
```

### 5) VM 접속

```
vagrant ssh [VM]
```

### 6) 재부팅

```
vagrant reload [VM]
```

### 7) VM 중지

```
vagrant halt [VM]
```

### 8) 삭제
```
vagrant destroy [VM]
```
