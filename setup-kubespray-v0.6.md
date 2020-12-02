
## Table of Contents

1. [문서 개요](#1)  
  1.1. [목적](#1.1)  
  1.2. [범위](#1.2)  
  1.3. [시스템 구성도](#1.3)  
  1.4. [참고자료](#1.4)  

2. [Kubespray 설치](#2)  
  2.1. [Prerequisite](#2.1)  
  2.2. [SSH Key 생성 및 배포](#2.2)  
  2.3. [Kubespray 다운로드](#2.3)  
  2.4. [Ubuntu, Python Package 설치](#2.4)  
  2.5. [Kubespray 파일 수정](#2.5)  
  2.6. [Kubespray 설치](#2.6)  
  2.7. [Kubespray 설치 확인](#2.7)  

3. [Kubespray 삭제](#3)  

4. [컨테이너 플랫폼 운영자 생성 및 Token 획득](#4)  
  4.1. [Cluster Role 운영자 생성 및 Token 획득](#4.1)  
  4.2. [Namespace 사용자 Token 획득](#4.2)  
  
5. [Resource 생성 시 주의사항](#5)  

<br>

## <div id='1'> 1. 문서 개요

### <div id='1.1'> 1.1. 목적
본 문서 (Kubespray 설치 가이드) 는 개방형 PaaS 플랫폼 고도화 및 개발자 지원 환경 기반의 Open PaaS에 배포되는 컨테이터 플랫폼을 설치하기 위한 Kubernetes Native를 Kubespray를 이용하여 설치하는 방법을 기술하였다.

PaaS-TA 6.0 버전부터는 Kubespray 기반으로 단독 배포를 지원한다. 기존 Container 서비스 기반으로 설치를 원할 경우에는 PaaS-TA 5.0 이하 버전의 문서를 참고한다.

<br>

### <div id='1.2'> 1.2. 범위
설치 범위는 Kubernetes Native를 검증하기 위한 Kubespray 기본 설치를 기준으로 작성하였다.

<br>

### <div id='1.3'> 1.3. 시스템 구성도
![image](https://user-images.githubusercontent.com/67575226/96062457-b0f89680-0ed0-11eb-8205-89e5d0327a65.png)

<br>

### <div id='1.4'> 1.4. 참고자료
> https://kubespray.io  
> https://github.com/kubernetes-sigs/kubespray  

<br>

## <div id='2'> 2. Kubespray 설치

### <div id='2.1'> 2.1. Prerequisite
본 설치 가이드는 Ubuntu 환경에서 설치하는 것을 기준으로 하였다. Kubespray 설치를 위해서는 Ansible v2.9 +, Jinja 2.11+ 및 python-netaddr이 Ansible 명령을 실행할 시스템에 설치되어 있어야 한다.

Kubespray 설치에 필요한 주요 소프트웨어 및 패키지 Version 정보는 다음과 같다.

|주요 소프트웨어|Version|Python Package|Version
|---|---|---|---|
|Kubespray|v2.14.1|ansible|2.9.6|
|Kubernetes Native|v1.18.6|jinja2|2.11.1|
|Docker|v19.03.12|netaddr|0.7.19|
|||pbr|5.4.4|
|||jmespath|0.9.5|
|||ruamel.yaml|0.16.10|

Kubernetes 공식 가이드 문서에서는 Cluster 배포 시 다음을 권고하고 있다.

- deb / rpm 호환 Linux OS를 실행하는 하나 이상의 머신 (Ubuntu 또는 CentOS)
- 머신 당 2G 이상의 RAM
- control-plane 노드로 사용하는 머신에 2 개 이상의 CPU
- 클러스터의 모든 시스템 간의 완전한 네트워크 연결

<br>

### <div id='2.2'> 2.2. SSH Key 생성 및 배포
Kubespray 설치를 위해서는 SSH Key가 인벤토리의 모든 서버들에 복사되어야 한다. 본 설치 가이드에서는 RSA 공개키를 이용하여 SSH 접속 설정을 진행한다.  

SSH Key 생성 및 배포 이후의 모든 설치과정은 Master Node에서 진행한다.

- Master Node에서 RSA 공개키를 생성한다.
```
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa): [엔터키 입력]
Enter passphrase (empty for no passphrase): [엔터키 입력]
Enter same passphrase again: [엔터키 입력]
Your identification has been saved in /home/ubuntu/.ssh/id_rsa.
Your public key has been saved in /home/ubuntu/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:TDN47SvyTaJ2es9nGrCLDQi6K3F8JYI2IBQ8axx+COg ubuntu@ip-10-0-0-148
The key's randomart image is:
+---[RSA 2048]----+
|+o.              |
|=+     . .       |
|B.=   . = .      |
|.E... .+ +       |
|o.+. o  S .      |
|..o...   o .     |
|.o .. o + +      |
|..    .BoB .o    |
|+.   .+++.=+     |
+----[SHA256]-----+
```

- 사용할 Master, Worker Node에 공개키를 복사한다.
```
# 출력된 공개키 복사
$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDAdc4dIUh1AbmMrMQtLH6nTNt6WZA9K5BzyNAEsDbbm8OzCYjGPFNexrxU2OyfHAUzLhs+ovXafX0RG5bvm44B04LH01maV8j32Vkag0DtNEiA96WjR9wpTeqfZy0Qwko9+TJOfK7lVT7+GCPm112pzU/t3i9oaptFdalGLYC+ib2+ViibkV0rZ8ds/zz/i0uzXDqvYl1HYfc7kA1CtinAimxV2FU/7WDTIj5HAfPnhyXPf+k1d3hPJEZ+T3qUmLnVpIXS2AHETPz29mu/I8EWUfc8/OVFJqS8RAyGghfnbFPrVEL3+jp/K6nwfX9nnpJWXvMtYenKwHI+mY8iuEYr ubuntu@ip-10-0-0-34
```

- 사용할 Master, Worker Node의 authorized_keys 파일에 공개키를 복사한다.
```
$ vi .ssh/authorized_keys
```

<br>

### <div id='2.3'> 2.3. Kubespray 다운로드
Kubespray 설치에 필요한 Source File을 Download 받아 Kubespray 설치 작업 경로로 위치시킨다.

- Kubespray Download URL : http://45.248.73.44/index.php/s/pJAYomHi2effiMR/download

- wget 명령을 통해 다음 경로에서 Kubespray 다운로드를 진행한다. 본 설치 가이드에서의 Kubespray 버전은 v2.14.1 이다.
```
$ wget --content-disposition http://45.248.73.44/index.php/s/pJAYomHi2effiMR/download

$ tar zxvf paasta-container-platform-kubespray.tar.gz
```

<br>

### <div id='2.4'> 2.4. Ubuntu, Python Package 설치
Kubespray 설치에 필요한 Ansible, Jinja 등 Python Package 설치를 진행한다.

- apt-get update를 진행한다.
```
$ sudo apt-get update
```

- python3-pip Package를 설치한다.
```
$ sudo apt-get install -y python3-pip
```

- Kubespray 설치경로 이동, pip를 이용하여 Kubespray 설치에 필요한 Python Package 설치를 진행한다.
```
$ cd paasta-container-platform-kubespray
$ sudo pip3 install -r requirements.txt
```

<br>

### <div id='2.5'> 2.5. Kubespray 파일 수정
Kubespray inventory 파일에는 배포할 Master, Worker Node의 구성을 정의한다.
본 설치 가이드에서는 1개의 Master Node와 3개의 Worker Node, 1개의 etcd 배포를 기준으로 가이드를 진행하며 기본 CNI는 calico로 설정되어있다.

- inventory sample 디렉토리를 복사한다.
```
$ cp -rfp inventory/sample inventory/mycluster
```

- 복사한 mycluster 디렉토리의 inventory.ini 파일을 설정한다.
```
$ vi inventory/mycluster/inventory.ini
```

```
# inventory.ini
# {MASTER_HOST_NAME}, {WORKER_HOST_NAME} : 실제 Master, Worker Node hostname
# {MASTER_NODE_IP}, {WORKER_NODE_IP} : Master, Worker Node Private IP

[all]
{MASTER_HOST_NAME} ansible_host={MASTER_NODE_IP}  # ip={MASTER_NODE_IP} etcd_member_name=etcd1
{WORKER_HOST_NAME1} ansible_host={WORKER_NODE_IP1}  # ip={WORKER_NODE_IP1}
{WORKER_HOST_NAME2} ansible_host={WORKER_NODE_IP2}  # ip={WORKER_NODE_IP2}
{WORKER_HOST_NAME3} ansible_host={WORKER_NODE_IP3}  # ip={WORKER_NODE_IP3}

[kube-master]
{MASTER_HOST_NAME}

[etcd]
{MASTER_HOST_NAME}

[kube-node]
{WORKER_HOST_NAME1}
{WORKER_HOST_NAME2}
{WORKER_HOST_NAME3}

[calico-rr]

[k8s-cluster:children]
kube-master
kube-node
calico-rr
```

- 설치할 Kubernetes Native Version을 변경한다.
```
$ vi inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml
```

```
# k8s-cluster.yml
# 23 Line kube_version: v1.18.9 -> kube_version: v1.18.6 변경
---
# Kubernetes configuration dirs and system namespace.
# Those are where all the additional config stuff goes
# the kubernetes normally puts in /srv/kubernetes.
# This puts them in a sane location and namespace.
# Editing those values will almost surely break something.
kube_config_dir: /etc/kubernetes
kube_script_dir: "{{ bin_dir }}/kubernetes-scripts"
kube_manifest_dir: "{{ kube_config_dir }}/manifests"

# This is where all the cert scripts and certs will be located
kube_cert_dir: "{{ kube_config_dir }}/ssl"

# This is where all of the bearer tokens will be stored
kube_token_dir: "{{ kube_config_dir }}/tokens"

# This is where to save basic auth file
kube_users_dir: "{{ kube_config_dir }}/users"

kube_api_anonymous_auth: true

## Change this to use another Kubernetes version, e.g. a current beta release
kube_version: v1.18.6
... ((생략)) ...
```

- inventory.py 파일을 수정하여 일부 설정을 변경한다.
```
$ vi contrib/inventory_builder/inventory.py
```

```
# inventory.py

# 본 설치 가이드에서는 1개의 Master Node로 설치를 진행하므로 Master Node의 갯수를 1개로 변경한다.
# 66 Line KUBE_MASTERS = int(os.environ.get("KUBE_MASTERS_MASTERS", 2)) -> KUBE_MASTERS = int(os.environ.get("KUBE_MASTERS_MASTERS", 1)) 변경

# False : 실제 hostname을 자동으로 변경한다 (ex: node1, node2 node3, node4), True : 실제 hostname을 유지한다.
# 73 Line USE_REAL_HOSTNAME = get_var_as_bool("USE_REAL_HOSTNAME", False) -> USE_REAL_HOSTNAME = get_var_as_bool("USE_REAL_HOSTNAME", True) 변경

# 본 설치 가이드에서는 총 4개의 Node를 사용하므로 변경하지 않을 시 inventory.ini 설정과는 무관하게 3개의 etcd가 설치된다.
# 102 Line etcd_hosts_count = 3 if len(self.hosts.keys()) >= 3 else 1 -> etcd_hosts_count = 3 if KUBE_MASTERS >= 3 else 1 변경
```

<br>

### <div id='2.6'> 2.6. Kubespray 설치
Ansible playbook을 이용하여 Kubespray 설치를 진행한다.

- 인벤토리 빌더로 Ansible 인벤토리 파일을 업데이트한다.
```
# {MASTER_NODE_IP}, {WORKER_NODE_IP} : Master, Worker Node Private IP
$ declare -a IPS=({MASTER_NODE_IP} {WORKER_NODE_IP1} {WORKER_NODE_IP2} {WORKER_NODE_IP3})

$ CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```

- Ansible playbook으로 Kubespray 배포를 진행한다. playbook은 root로 실행하도록 옵션을 지정한다. (--become-user=root)
```
$ ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml
```

- Kubespray 설치 완료 후 Cluster 사용을 위하여 다음 과정을 실행한다.
```
$ mkdir -p $HOME/.kube

$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

<br>

### <div id='2.7'> 2.7. Kubespray 설치 확인
Kubernetes Node 및 kube-system Namespace의 Pod를 확인하여 Kubespray 설치를 확인한다.

```
$ kubectl get nodes
NAME            STATUS   ROLES    AGE     VERSION
ip-10-0-0-140   Ready    <none>   2m      v1.18.6
ip-10-0-0-148   Ready    master   2m54s   v1.18.6
ip-10-0-0-175   Ready    <none>   2m      v1.18.6
ip-10-0-0-54    Ready    <none>   2m      v1.18.6

$ kubectl get pods -n kube-system
NAME                                          READY   STATUS    RESTARTS   AGE
calico-kube-controllers-86744fd67d-nl9zn      1/1     Running   0          101s
calico-node-gkmq8                             1/1     Running   1          2m2s
calico-node-pwdpp                             1/1     Running   1          2m2s
calico-node-qxbl5                             1/1     Running   1          2m2s
calico-node-sfbgc                             1/1     Running   1          2m2s
coredns-dff8fc7d-f2bhs                        1/1     Running   0          85s
coredns-dff8fc7d-zfn8m                        1/1     Running   0          90s
dns-autoscaler-66498f5c5f-vpw87               1/1     Running   0          87s
kube-apiserver-ip-10-0-0-148                  1/1     Running   0          3m14s
kube-controller-manager-ip-10-0-0-148         1/1     Running   0          3m14s
kube-proxy-27z6x                              1/1     Running   0          2m26s
kube-proxy-g9lwc                              1/1     Running   0          2m24s
kube-proxy-m67ts                              1/1     Running   0          2m24s
kube-proxy-vh7fr                              1/1     Running   0          2m24s
kube-scheduler-ip-10-0-0-148                  1/1     Running   1          3m14s
kubernetes-dashboard-667c4c65f8-htprk         1/1     Running   0          84s
kubernetes-metrics-scraper-54fbb4d595-dmmcr   1/1     Running   0          83s
nginx-proxy-ip-10-0-0-140                     1/1     Running   0          2m23s
nginx-proxy-ip-10-0-0-175                     1/1     Running   0          2m23s
nginx-proxy-ip-10-0-0-54                      1/1     Running   0          2m23s
nodelocaldns-pw8mq                            1/1     Running   0          85s
nodelocaldns-rt8sv                            1/1     Running   0          85s
nodelocaldns-sj4gh                            1/1     Running   0          85s
nodelocaldns-zhf2v                            1/1     Running   0          85s
```

<br>

## <div id='3'> 3. Kubespray 삭제
Ansible playbook을 이용하여 Kubespray 삭제를 진행한다.

```
$ ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root reset.yml
```

<br>

## <div id='4'> 4. 컨테이너 플랫폼 운영자 생성 및 Token 획득

### <div id='4.1'> 4.1. Cluster Role 운영자 생성 및 Token 획득
Kubespray 설치 이후에 Cluster Role을 가진 운영자의 Service Account를 생성한다. 해당 Service Account의 Token은 운영자 포털에서 Super Admin 계정 생성 시 이용된다.

- Service Account를 생성한다.
```
# {SERVICE_ACCOUNT} : Service Account 예시
$ kubectl create serviceaccount {SERVICE_ACCOUNT} -n kube-system
```

- Cluster Role을 생성한 Service Account에 바인딩한다.
```
$ kubectl create clusterrolebinding {SERVICE_ACCOUNT} --clusterrole=cluster-admin --serviceaccount=kube-system:{SERVICE_ACCOUNT}
```

- 생성한 Service Account의 Token을 획득한다.
```
# {SECRET_NAME} : Mountable secrets 값 확인
$ kubectl describe serviceaccount {SERVICE_ACCOUNT} -n kube-system

$ kubectl describe secret {SECRET_NAME} -n kube-system | grep -E '^token' | cut -f2 -d':' | tr -d " "
```

### <div id='4.2'> 4.2. Namespace 사용자 Token 획득
포털에서 Namespace 생성 및 사용자 등록 이후 Token값을 획득 시 이용된다.

- Namespace 사용자의 Token을 획득한다.
```
# {SECRET_NAME} : Mountable secrets 값 확인
# {NAMESPACE} : Namespace 명
$ kubectl describe serviceaccount {SERVICE_ACCOUNT} -n {NAMESPACE}

$ kubectl describe secret {SECRET_NAME} -n {NAMESPACE} | grep -E '^token' | cut -f2 -d':' | tr -d " "
```

<br>

## <div id='5'> 5. Resource 생성 시 주의사항
사용자가 직접 Resource를 생성 시 다음과 같은 prefix를 사용하지 않도록 주의한다.

|Resource 명|생성 시 제외해야 할 prefix|
|---|---|
|전체 Resource|kube*|
|Namespace|all|
||kubernetes-dashboard|
||paas-ta-container-platform-temp-namespace|
|Role|paas-ta-container-platform-init-role|
||paas-ta-container-platform-admin-role|
|ResourceQuota|paas-ta-container-platform-low-rq|
||paas-ta-container-platform-medium-rq|
||paas-ta-container-platform-high-rq|
|LimitRanges|paas-ta-container-platform-low-limit-range|
||paas-ta-container-platform-medium-limit-range|
||paas-ta-container-platform-high-limit-range|
|Pod|nodes|
||resources|

<br>
