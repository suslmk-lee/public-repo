## GlusterFS

<b>GlusterFS 서버 설치 및 Peer 연결</b>
```
$ vi /etc/hosts
10.0.0.151 master 
10.0.0.252 swarm-1 
10.0.0.158 swarm-2 
10.0.0.177 swarm-3

## 설정여부 확인
$ ping swarm-1

## install Gluster FS Server
$ sudo apt install software-properties-common -y 
$ wget -O- https://download.gluster.org/pub/gluster/glusterfs/6/rsa.pub | apt-key add - 
$ sudo add-apt-repository ppa:gluster/glusterfs-6 
$ sudo apt install glusterfs-server -y 
$ sudo systemctl start glusterd 
$ sudo systemctl enable glusterd

## Configure Gluster FS Servers
## peer로 연결 설정/ 각서버
$ sudo gluster peer probe swarm-2
$ sudo gluster peer probe swarm-3

## peer 연결 상태 확인
$ sudo gluster peer status 
Number of Peers: 2 
Hostname: swarm-2 
Uuid: f574b1ef-c525-4d7d-9cdd-fe677ca90202 
State: Peer in Cluster (Connected) 
Hostname: swarm-3 
Uuid: 570fb78a-ba68-4a2f-a762-d88ce0bdd420 
State: Peer in Cluster (Connected)

$ sudo gluster pool list 
UUID					Hostname 	State 
f574b1ef-c525-4d7d-9cdd-fe677ca90202	swarm-2  	Connected  
570fb78a-ba68-4a2f-a762-d88ce0bdd420	swarm-3  	Connected  
31b3a96d-c869-4ab6-800a-f5b291043742	localhost	Connected
```

<b>볼륨생성 및 시작</b>  
Heketi를 이용할 경우 이후 작업 필요없음.
```
## 각각 볼륨디렉토리 생성
$ sudo mkdir -p /gfsvolume/swarm

## 볼륨생성
$ sudo gluster volume create swarm_vol transport tcp \ 
    swarm-1:/gfsvolume/swarm swarm-2:/gfsvolume/swarm swarm-3:/gfsvolume/swarm force 

## 볼륨 시작 
$ sudo gluster volume start swarm_vol
volume start: vol: success

## 생성된 볼륨 확인
$ sudo gluster volume info swarm_vol
Volume Name: swarm_vol 
Type: Distribute 
Volume ID: ef6db4db-4fc2-4ece-876e-8e5235bc6359 
Status: Started 
Snapshot Count: 0 
Number of Bricks: 3 
Transport-type: tcp 
Bricks: 
Brick1: swarm-1:/gfsvolume/swarm 
Brick2: swarm-2:/gfsvolume/swarm 
Brick3: swarm-3:/gfsvolume/swarm 
Options Reconfigured: 
transport.address-family: inet 
nfs.disable: on
```

<b>GlusterFS 마운트</b>  
생성된 볼륨을 swarm-1과 swarm-2서버에 각각 마운트하여 파일이 동기화 되는지 확인해 보도록 한다.   
먼저 swarm-1서버에서 아래의 명령어를 실행한다. 

```
swarm-1$ sudo mkdir -p /mnt/gluster
swarm-1$ sudo mount -t glusterfs swarm-1:/swarm_vol /mnt/gluster
```
위의 명령어를 실행하면 Gluster swarm-1서버의 볼륨 swarm_vol이 /mnt/gluster 디렉터리에 마운트 되게 된다.    
그리고 다음 명령어로 테스트 파일을 생성한다.

```
swarm-1$ sudo touch /mnt/gluster/hello.txt
```
다음은 swarm-2 서버에서 Gluster을 마운트하고, 파일이 동기화 되었는지 확인해 보도록 한다.    
swarm-2서버에서 다음의 명령어로 볼륨을 마운트 한다.

```
swarm-2$ sudo mkdir -p /mnt/gluster
swarm-2$ sudo mount -t glusterfs swarm-2:/swarm_vol /mnt/gluster
```
그리고 아래의 명령어로 파일이 동기화 되었는지 확인 한다.

```
swarm-2$ ls -alh /mnt/gluster/hello.txt
-rw-r--r-- 1 root root 0 Oct 26 15:53 /mnt/gluster/hello.txt
```

swarm-3 서버도 gluser 마운트한다. 
※ mount -t  :: 파일시스템 옵션 여기서는 glusterfs로 지정한다.
```
swarm-3$ sudo mkdir -p /mnt/gluster
swarm-3$ sudo mount -t glusterfs swarm-3:/swarm_vol /mnt/gluster
swarm-3$ ll /mnt/gluster
total 8 
drwxr-xr-x 3 root root 4096 Feb 15 04:37 ./ 
drwxr-xr-x 3 root root 4096 Feb 15 04:58 ../ 
-rw-r--r-- 1 root root    0 Feb 15 04:37 hello.txt
```

- Client 작업
  GlusterFS서버에서 파일시스템으로 마운트 할 수 있을 뿐만 아니라, 외부 서버에서 마운트 할 수 있다.   
  아래의 명령어로 glusterfs-client 패키지를 설치한다.
  Kubernetes에서도 외부 heketi를 통해 glusterfs를 이용하려면 모든 노드에 glusterfs-client를 설치하여야 한다.
```
client$ sudo apt-get install software-properties-common 
client$ sudo add-apt-repository ppa:gluster/glusterfs-6 
client$ sudo apt-get update 
client$ sudo apt install glusterfs-client

## 볼륨 마운트
client$ mkdir -p /mnt/gluster 
client$ sudo mount -t glusterfs master:/swarm_vol /mnt/gluster

client$ df -h /mnt/gluster/ 
Filesystem         Size  Used Avail Use% Mounted on 
master:/swarm_vol  1.3T   92G  1.1T   8% /mnt/swarm
```


```
## 설치된 패키지 확인
$ sudo apt list | grep glusterfs
WARNING: apt does not have a stable CLI interface. Use with caution in scripts. 
glusterfs-client/bionic-updates,now 3.13.2-1ubuntu1 amd64 [installed] 
glusterfs-common/bionic-updates,now 3.13.2-1ubuntu1 amd64 [installed,automatic] 
glusterfs-dbg/bionic 3.12.15-ubuntu1~bionic1 amd64 
glusterfs-server/bionic-updates 3.13.2-1ubuntu1 amd64 
libvirt-daemon-driver-storage-gluster/bionic-updates,bionic-security 4.0.0-1ubuntu8.17 amd64 
nfs-ganesha-gluster/bionic 2.6.0-2 amd64 
uwsgi-plugin-glusterfs/bionic-updates,bionic-security 2.0.15-10.2ubuntu2.1 amd64

## glusterfs-client 삭제
$ sudo apt purge glusterfs-client

```