## Heketi

<b>Heketi란?</b>  
GlusterFS용 RESTful 볼륨 관리 프레임워크로서 GlusterFS 볼륨의 수명주기를 관리하는 RESTful 인터페이스를 제공한다.  

GlusterFS 스토리지가 미리 설정되어 있어야 한다.

<b>설치구성도</b>

![image](https://user-images.githubusercontent.com/67575226/108801379-e88ef200-75d8-11eb-9eac-58c4e403fdd6.png)


<b>Download Heketi bin</b>
```
## Install Heketi ##
$ wget https://github.com/heketi/heketi/releases/download/v9.0.0/heketi-v9.0.0.linux.amd64.tar.gz 
$ tar -zxvf heketi-v9.0.0.linux.amd64.tar.gz
```

<b>Copy bin</b>
```
$ chmod +x heketi/heketi
$ chmod +x heketi/heketi-cli
$ cp heketi/heketi /usr/local/bin
$ cp heketi/heketi-cli /usr/local/bin
```

<b>Check heketi is working</b>
```
$ heketi --version
$ heketi-cli --version
```

<b>Add a user/group for heketi</b>
```
$ groupadd --system heketi
$ useradd -s /sbin/nologin --system -g heketi heketi
```

<b>Create dir for heketi</b>   
jwt 섹션의 admin key와 user key를 설정해 준다. 여기서는 keypassword로 설정했다. 하단의 executor를 ssh 로 설정하고 sshexec섹션의 keyfile을 heketi ssh key 가 존재하는 경로로 변경한다.
```
$ mkdir -p /var/lib/heketi /etc/heketi /var/log/heketi
$ vi /etc/heketi/heketi.json
{ 
  "_port_comment": "Heketi Server Port Number", 
  "port": "8080", 
	"_enable_tls_comment": "Enable TLS in Heketi Server", 
	"enable_tls": false, 
	"_cert_file_comment": "Path to a valid certificate file", 
	"cert_file": "", 
	"_key_file_comment": "Path to a valid private key file", 
	"key_file": "", 
  "_use_auth": "Enable JWT authorization. Please enable for deployment", 
  "use_auth": false, 
  "_jwt": "Private keys for access", 
  "jwt": { 
    "_admin": "Admin has access to all APIs", 
    "admin": { 
      "key": "keypassword" 
    }, 
    "_user": "User only has access to /volumes endpoint", 
    "user": { 
      "key": "keypaasword" 
    } 
  }, 
  "_backup_db_to_kube_secret": "Backup the heketi database to a Kubernetes secret when running in Kubernetes. Default is off.", 
  "backup_db_to_kube_secret": false, 
  "_profiling": "Enable go/pprof profiling on the /debug/pprof endpoints.", 
  "profiling": false, 
  "_glusterfs_comment": "GlusterFS Configuration", 
  "glusterfs": { 
    "_executor_comment": [ 
      "Execute plugin. Possible choices: mock, ssh", 
      "mock: This setting is used for testing and development.", 
      "      It will not send commands to any node.", 
      "ssh:  This setting will notify Heketi to ssh to the nodes.", 
      "      It will need the values in sshexec to be configured.", 
      "kubernetes: Communicate with GlusterFS containers over", 
      "            Kubernetes exec api." 
    ], 
    "executor": "ssh", 
    "_sshexec_comment": "SSH username and private key file information", 
    "sshexec": { 
      "keyfile": "/etc/heketi/heketi_key", 
      "user": "root", 
      "port": "22", 
      "fstab": "/etc/fstab" 
    }, 
    "_db_comment": "Database file name", 
    "db": "/var/lib/heketi/heketi.db", 
     "_refresh_time_monitor_gluster_nodes": "Refresh time in seconds to monitor Gluster nodes", 
    "refresh_time_monitor_gluster_nodes": 120, 
    "_start_time_monitor_gluster_nodes": "Start time in seconds to monitor Gluster nodes when the heketi comes up", 
    "start_time_monitor_gluster_nodes": 10, 
    "_loglevel_comment": [ 
      "Set log level. Choices are:", 
      "  none, critical, error, warning, info, debug", 
      "Default is warning" 
    ], 
    "loglevel" : "debug", 
    "_auto_create_block_hosting_volume": "Creates Block Hosting volumes automatically if not found or exsisting volume exhausted", 
    "auto_create_block_hosting_volume": true, 
    "_block_hosting_volume_size": "New block hosting volume will be created in size mentioned, This is considered only if auto-create is enabled.", 
    "block_hosting_volume_size": 500, 
    "_block_hosting_volume_options": "New block hosting volume will be created with the following set of options. Removing the group gluster-block option is NOT recommended. Additional options can be added next to it separated by a comma.", 
    "block_hosting_volume_options": "group gluster-block", 
    "_pre_request_volume_options": "Volume options that will be applied for all volumes created. Can be overridden by volume options in volume create request.", 
    "pre_request_volume_options": "", 
    "_post_request_volume_options": "Volume options that will be applied for all volumes created. To be used to override volume options in volume create request.", 
    "post_request_volume_options": "" 
  } 
}
```

<b>Load all Kernel modules that wiill be required by Heketi.</b>
```
$ sudo modprobe dm_snapshot
$ sudo modprobe dm_mirror
$ sudo modprobe dm_thin_pool
```

<b>Create ssh key for the API to connect to the other hosts</b>
```
## Heketi Server
$ ssh-keygen -f /etc/heketi/heketi_key -t rsa -N ''
$ chown heketi:heketi /etc/heketi/heketi_key*
```

<b>Send key to all hosts</b>
```
## Not allowed ##
$ ssh-copy-id -i /etc/heketi/heketi_key.pub root@swarm-1
$ ssh-copy-id -i /etc/heketi/heketi_key.pub root@swarm-2
$ ssh-copy-id -i /etc/heketi/heketi_key.pub root@swarm-3
```

```
## Heketi Server
$ cat /etc/heketi/heketi_key.pub  
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC6yN+AWN/nQ/jkUyKCL/dP2cDZ0zuA5ATtrfxEzXeWqVD38eBIdrz0Q0BlUYLTQjeJQQqA16eXwPAtvbFdID4OGdUYsjJy0+2LtVRUoimLSqZKXcBaAQd9dpNqT+wOX/XoNI1KglWM6txZ2oh5D4vsgrorGgXM89bkrv6mIlacFFk9MVl64BzGIutHVzURSZot3Qdc0UTLb3MCLeAQrAU+c6BuVi9i6wISEzoL5ATjQfzcqK1UkV6z8BvyulPZopZ1IP6S6z7ZLm+d3jZifr9Q4YOWA7xm5eNxLUoVmsMXkthTD3+usFCigSbmcuPV9HP3/PYde98ABZofcYa+fr6h root@paasta-cp-suslmk-hkt

## glusterfs Node
$ sudo su
$ vi $HOME/.ssh/authorized_keys
    ~~ Heketi Server에서 생성한 공개키를 넣어준다.
```

<b>Allow heketi user perms on folders</b>
```
$ chown heketi:heketi /var/lib/heketi /var/log/heketi /etc/heketi
```

<b>Create a systemd file</b>
```
$ vi /etc/systemd/system/heketi.service

[Unit] 
Description=Heketi Server 
[Service] 
Type=simple 
WorkingDirectory=/var/lib/heketi 
EnvironmentFile=-/etc/heketi/heketi.env 
User=heketi 
ExecStart=/usr/local/bin/heketi --config=/etc/heketi/heketi.json 
Restart=on-failure 
StandardOutput=syslog 
StandardError=syslog 
[Install] 
WantedBy=multi-user.target
```

<b>Reload systemd and enable new heketi service</b>
```
$ systemctl daemon-reload 
$ systemctl enable --now heketi
$ systemctl start heketi

## curl을 이용하여 web 서비스 확인
$ curl http://localhost:8080/hello
Hello from Heketi
```

<b>Create topology</b>
```
$ vi /etc/heketi/topology.json

{ 
  "clusters": [ 
    { 
      "nodes": [ 
                    { 
          "node": { 
            "hostnames": { 
              "manage": [ 
                "swarm-1" 
              ], 
              "storage": [ 
                "172.16.10.245" 
              ] 
            }, 
            "zone": 1 
          }, 
          "devices": [ 
            "/dev/vdb" 
          ] 
        },            { 
          "node": { 
            "hostnames": { 
              "manage": [ 
                "swarm-2" 
              ], 
              "storage": [ 
                "172.16.10.9" 
              ] 
            }, 
            "zone": 1 
          }, 
          "devices": [ 
            "/dev/vdb" 
          ] 
        },            { 
          "node": { 
            "hostnames": { 
              "manage": [ 
                "swarm-3" 
              ], 
              "storage": [ 
                "172.16.10.23" 
              ] 
            }, 
            "zone": 1 
          }, 
          "devices": [ 
            "/dev/vdb" 
          ] 
        } 
      ] 
    } 
  ] 
}
```

<b>Load topology</b>
```
$ heketi-cli topology load --user admin --secret keypassword --json=/etc/heketi/topology.json -s http://localhost:8080
```

<b>Result</b>
```
root@ip-10-0-0-172:/etc/heketi# heketi-cli topology load --user admin --secret keypassword --json=/etc/heketi/topology.json 
	Found node swarm-1 on cluster 6e9b3e662fbcbee33de05d86a24ffaa4 
		Adding device /xvdb ... Unable to add device: Setup of device /xvdb failed (already initialized or contains data?):   Device /xvdb not found. 
	Creating node swarm-2 ... ID: dffa95dc868144f5d6dd8957cd063847 
		Adding device /xvdb ... Unable to add device: Setup of device /xvdb failed (already initialized or contains data?): ssh: handshake failed: ssh: unable to authenticate, attempted methods [none publickey], no supported methods remain 
	Creating node swarm-3 ... ID: fcf0081066404542d859a70423f5b40e 
		Adding device /xvdb ... Unable to add device: Setup of device /xvdb failed (already initialized or contains data?): ssh: handshake failed: ssh: unable to authenticate, attempted methods [none publickey], no supported methods remain 
root@ip-10-0-0-172:/etc/heketi# vi topology.json  
root@ip-10-0-0-172:/etc/heketi# heketi-cli topology load --user admin --secret keypassword --json=/etc/heketi/topology.json 
	Found node swarm-1 on cluster 6e9b3e662fbcbee33de05d86a24ffaa4 
		Adding device /dev/xvdb ... OK 
	Found node swarm-2 on cluster 6e9b3e662fbcbee33de05d86a24ffaa4 
		Adding device /xvdb ... Unable to add device: Setup of device /xvdb failed (already initialized or contains data?): ssh: handshake failed: ssh: unable to authenticate, attempted methods [none publickey], no supported methods remain 
	Found node swarm-3 on cluster 6e9b3e662fbcbee33de05d86a24ffaa4 
		Adding device /xvdb ... Unable to add device: Setup of device /xvdb failed (already initialized or contains data?): ssh: handshake failed: ssh: unable to authenticate, attempted methods [none publickey], no supported methods remain 
root@ip-10-0-0-172:/etc/heketi# vi topology.json  
root@ip-10-0-0-172:/etc/heketi# heketi-cli topology load --user admin --secret keypassword --json=/etc/heketi/topology.json 
	Found node swarm-1 on cluster 6e9b3e662fbcbee33de05d86a24ffaa4 
		Found device /dev/xvdb 
	Found node swarm-2 on cluster 6e9b3e662fbcbee33de05d86a24ffaa4 
		Adding device /dev/xvdb ... OK 
	Found node swarm-3 on cluster 6e9b3e662fbcbee33de05d86a24ffaa4 
		Adding device /dev/xvdb ... OK
```

<b>Check connection to other devices work</b>
```
$ heketi-cli cluster list
```


<b>Secret</b>
```
apiVersion: V1 
kind: Secret 
metadata: 
  name: heketi-secret 
  namespace: default 
type: "kubernetes.io/glusterfs" 
data: 
  # echo -n "password" | base64 
  # key: PASSWORD_BASE64_ENCODED 
  key: "a2V5cGFzc3dvcmQ="
```

<b>StorageClass</b>
```
apiVersion: storage.k8s.io/v1 
kind: StorageClass 
metadata: 
  name: "gluster-heketi" 
provisioner: kubernetes.io/glusterfs 
parameters: 
  resturl: "http://172.16.10.11:8080" 
  restuser: "admin" 
  # restuserkey: "keypassword" 
  secretName: "heketi-secret" 
  secretNamespace: "default" 
  gidMin: "40000" 
  gidMax: "50000"
```
* gidMin~gidMax 값은 Storage Class 내에서 각 볼륨에 대해 유일한 값으로 순차적으로 정해지는 값이며, 볼륨이 삭제되면 gid Pool에 반납되도록 되어 있는 supplemental Group ID 값이다(PV가 attach된 Pod 내의 root 계정으로 전달되어 PV에 write 가능하게 된다. id 명령으로 확인 가능 - 참고), 별도 지정하지 않으면 2000~2147483647 사이의 값이 임의로 할당되는데, Heketi 3.x 이후 버전에서는 이 범위를 Storage Class 정의 시 지정하게 되어 있다 - 참고.

```
## heketi-server
## 생성되는 heketi-storage.json 파일을 Kubernetes Master서버로 
$ heketi-cli setup-openshift-heketi-storage
Saving heketi-storage.json
```

```
## Kubernetes Master ##
ubuntu@paasta-cp-ktw-kubespray-005:~$ kubectl delete -f heketi-storage.json  
secret "heketi-storage-secret" deleted 
endpoints "heketi-storage-endpoints" deleted 
service "heketi-storage-endpoints" deleted 
job.batch "heketi-storage-copy-job" deleted 
ubuntu@paasta-cp-ktw-kubespray-005:~$ kubectl get ep,job,pod,svc 
NAME                                                               ENDPOINTS                                      AGE 
endpoints/glusterfs-dynamic-21fd8f50-fd34-4a02-af74-2e311b3a8a3e   172.16.10.17:1,172.16.10.46:1,172.16.10.96:1   3h7m 
endpoints/kubernetes                                               172.16.10.241:6443                             24h 
NAME               READY   STATUS              RESTARTS   AGE 
pod/nginx-pv-pod   0/1     ContainerCreating   0          3h 
NAME                                                             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE 
service/glusterfs-dynamic-21fd8f50-fd34-4a02-af74-2e311b3a8a3e   ClusterIP   10.233.35.102   <none>        1/TCP     3h7m 
service/kubernetes                                               ClusterIP   10.233.0.1      <none>        443/TCP   24h 
ubuntu@paasta-cp-ktw-kubespray-005:~$ kubectl create -f heketi-storage.json  
secret/heketi-storage-secret created 
endpoints/heketi-storage-endpoints created 
service/heketi-storage-endpoints created 
job.batch/heketi-storage-copy-job created
```

<b>PVC 생성</b>
```
apiVersion: v1 
kind: PersistentVolumeClaim 
metadata: 
  name: pvc-1 
spec: 
  storageClassName: gluster-heketi 
  accessModes: 
  - ReadWriteMany 
  resources: 
    requests: 
      storage: 2Gi
```

```
ubuntu@paasta-cp-ktw-kubespray-005:~$ kubectl get pvc 
NAME              STATUS        VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS     AGE 
pvc-1             Bound         pvc-cac6bd0e-466c-40c1-a4f3-1829233046f0   2Gi        RWX            gluster-heketi   112m
ubuntu@paasta-cp-ktw-kubespray-005:~$ kubectl get pv 
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                     STORAGECLASS     REASON   AGE 
pvc-cac6bd0e-466c-40c1-a4f3-1829233046f0   2Gi        RWX            Delete           Bound    default/pvc-1             gluster-heketi            112m
```

<b>Deployment 생성</b>
```
apiVersion: apps/v1 
kind: Deployment 
metadata: 
  name: gfs-client 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: ubuntu 
  template: 
    metadata: 
      labels: 
        app: ubuntu 
    spec: 
      containers: 
      - name: ubuntu 
        image: ubuntu 
        volumeMounts: 
        - name: gfs 
          mountPath: /mnt 
        command: ["/usr/bin/tail","-f","/dev/null"] 
      volumes: 
      - name: gfs 
        persistentVolumeClaim: 
          claimName: pvc-1
```

<b>deployment, pod 확인</b>
```
ubuntu@paasta-cp-ktw-kubespray-005:~$ kubectl get deploy,pod 
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE 
deployment.apps/gfs-client   1/1     1            1           56m 
NAME                                READY   STATUS              RESTARTS   AGE 
pod/gfs-client-7c9f989964-w5v78     1/1     Running             0          56m

ubuntu@paasta-cp-ktw-kubespray-005:~$ kubectl describe pod gfs-client-7c9f989964-w5v78  
Name:         gfs-client-7c9f989964-w5v78 
Namespace:    default 
Priority:     0 
Node:         paasta-cp-ktw-kubespray-007/172.16.10.198 
Start Time:   Thu, 18 Feb 2021 09:32:58 +0000 
Labels:       app=ubuntu 
              pod-template-hash=7c9f989964 
Annotations:  cni.projectcalico.org/podIP: 10.233.97.5/32 
              cni.projectcalico.org/podIPs: 10.233.97.5/32 
Status:       Running 
IP:           10.233.97.5 
IPs: 
  IP:           10.233.97.5 
Controlled By:  ReplicaSet/gfs-client-7c9f989964 
Containers: 
  ubuntu: 
    Container ID:  docker://febda6399c08e87b05b9acef5983b3825685c8b6a05fb4ebe5322f45b46e4ee6 
    Image:         ubuntu 
    Image ID:      docker-pullable://ubuntu@sha256:703218c0465075f4425e58fac086e09e1de5c340b12976ab9eb8ad26615c3715 
    Port:          <none> 
    Host Port:     <none> 
    Command: 
      /usr/bin/tail 
      -f 
      /dev/null 
    State:          Running 
      Started:      Thu, 18 Feb 2021 09:33:01 +0000 
    Ready:          True 
    Restart Count:  0 
    Environment:    <none> 
    Mounts: 
      /mnt from gfs (rw) 
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-rkkjt (ro) 
Conditions: 
  Type              Status 
  Initialized       True  
  Ready             True  
  ContainersReady   True  
  PodScheduled      True  
Volumes: 
  gfs: 
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace) 
    ClaimName:  pvc-1 
    ReadOnly:   false 
  default-token-rkkjt: 
    Type:        Secret (a volume populated by a Secret) 
    SecretName:  default-token-rkkjt 
    Optional:    false 
QoS Class:       BestEffort 
Node-Selectors:  <none> 
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s 
                 node.kubernetes.io/unreachable:NoExecute for 300s 
Events: 
  Type    Reason     Age        From                                  Message 
  ----    ------     ----       ----                                  ------- 
  Normal  Scheduled  <unknown>  default-scheduler                     Successfully assigned default/gfs-client-7c9f989964-w5v78 to paasta-cp-ktw-kubespray-007 
  Normal  Pulling    56m        kubelet, paasta-cp-ktw-kubespray-007  Pulling image "ubuntu" 
  Normal  Pulled     56m        kubelet, paasta-cp-ktw-kubespray-007  Successfully pulled image "ubuntu" 
  Normal  Created    56m        kubelet, paasta-cp-ktw-kubespray-007  Created container ubuntu 
  Normal  Started    56m        kubelet, paasta-cp-ktw-kubespray-007  Started container ubuntu
```

<b>pod 내에서 파일 생성 후 volume에서 파일 확인</b>
```
ubuntu@paasta-cp-ktw-kubespray-005:~$ kubectl exec -it gfs-client-7c9f989964-w5v78 bash 
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl kubectl exec [POD] -- [COMMAND] instead. 
groups: cannot find name for group ID 40001 
root@gfs-client-7c9f989964-w5v78:/# cd /mnt 
root@gfs-client-7c9f989964-w5v78:/mnt# echo 'eskeifjsf' > aa.txt 
root@gfs-client-7c9f989964-w5v78:/mnt# ll 
total 5 
drwxrwsr-x 3 root 40001   38 Feb 18 10:39 ./ 
drwxr-xr-x 1 root root  4096 Feb 18 09:33 ../ 
-rw-r--r-- 1 root 40001   10 Feb 18 10:39 aa.txt


root@paasta-cp-ktw-kubespray-007:/home/ubuntu# mount -l | grep gluster 
172.16.10.23:vol_98312f8ad7522ad4a95de3a48c4fb18d on /var/lib/kubelet/pods/e02e8cac-720d-421d-986b-0e72b080e809/volumes/kubernetes.io~glusterfs/pvc-cac6bd0e-466c-40c1-a4f3-1829233046f0 type fuse.glusterfs (rw,relatime,user_id=0,group_id=0,default_permissions,allow_other,max_read=131072)



root@paasta-cp-swarm-1:~# gluster volume info vol_98312f8ad7522ad4a95de3a48c4fb18d 
Volume Name: vol_98312f8ad7522ad4a95de3a48c4fb18d 
Type: Replicate 
Volume ID: 004caaac-4758-472e-aac3-ca9fb6086721 
Status: Started 
Snapshot Count: 0 
Number of Bricks: 1 x 3 = 3 
Transport-type: tcp 
Bricks: 
Brick1: 172.16.10.9:/var/lib/heketi/mounts/vg_78c797f3587e7e687a8bb922d9fd6ec7/brick_1e491a3cd38f7f4724351e47e6b65cca/brick 
Brick2: 172.16.10.23:/var/lib/heketi/mounts/vg_ee5a7fd72b56ae50f811b4c14ee6fd44/brick_5281777ac8e42a148a823c6d8fcd472b/brick 
Brick3: 172.16.10.245:/var/lib/heketi/mounts/vg_78bd9744edbfcb45e06b724ed0b32f74/brick_734b5faa288eed6b863cd12dc62b2b52/brick 
Options Reconfigured: 
user.heketi.id: 98312f8ad7522ad4a95de3a48c4fb18d 
transport.address-family: inet 
nfs.disable: on 
performance.client-io-threads: off

root@paasta-cp-swarm-1:~# ls -l /var/lib/heketi/mounts/vg_78bd9744edbfcb45e06b724ed0b32f74/brick_734b5faa288eed6b863cd12dc62b2b52/brick/ 
total 4 
-rw-r--r-- 2 root 40001 10 Feb 18 10:39 aa.txt
```

```
## Volume 정리
$ dmsetup remove_all
$ wipefs -a -f /dev/vdb
$ fdisk -l
$ lsblk
```




<b>Node정보 업데이트 해야 할 경우 </b>

```
root@paasta-cp-suslmk-hkt:/etc/heketi# heketi-cli topology info 
Cluster Id: a51969941dfb603e1cb97461a4943c09 
    File:  true 
    Block: true 
    Volumes: 
    Nodes: 
        Node Id: 6973c409b7e82ae49f9a302262564ad9 
        State: online 
        Cluster Id: a51969941dfb603e1cb97461a4943c09 
        Zone: 1 
        Management Hostnames: swarm-3 
        Storage Hostnames: 172.16.10.250 
        Devices: 
        Node Id: 92d804890bb64596a5b462e2ad780163 
        State: online 
        Cluster Id: a51969941dfb603e1cb97461a4943c09 
        Zone: 1 
        Management Hostnames: swarm-1 
        Storage Hostnames: 172.16.10.245 
        Devices: 
                Id:5cfe6f6f71913b5184df094837e5e23b   Name:/dev/vdb            State:online    Size (GiB):29      Used (GiB):0       Free (GiB):29       
                        Bricks: 
        Node Id: fdf81ce3efe6b6db832acc1470937b3c 
        State: online 
        Cluster Id: a51969941dfb603e1cb97461a4943c09 
        Zone: 1 
        Management Hostnames: swarm-2 
        Storage Hostnames: 172.16.10.76 
        Devices: 
                Id:758d4520bd4c5efe4b399162a55639f7   Name:/dev/vdb            State:online    Size (GiB):29      Used (GiB):0       Free (GiB):29       
                        Bricks: 
root@paasta-cp-suslmk-hkt:/etc/heketi# heketi-cli device disable 5cfe6f6f71913b5184df094837e5e23b 
Device 5cfe6f6f71913b5184df094837e5e23b is now offline 
root@paasta-cp-suslmk-hkt:/etc/heketi# heketi-cli device remove 5cfe6f6f71913b5184df094837e5e23b 
Device 5cfe6f6f71913b5184df094837e5e23b is now removed 
root@paasta-cp-suslmk-hkt:/etc/heketi# heketi-cli device delete 5cfe6f6f71913b5184df094837e5e23b 
Device 5cfe6f6f71913b5184df094837e5e23b deleted 
root@paasta-cp-suslmk-hkt:/etc/heketi# heketi-cli device disable 758d4520bd4c5efe4b399162a55639f7 
Device 758d4520bd4c5efe4b399162a55639f7 is now offline 
root@paasta-cp-suslmk-hkt:/etc/heketi# heketi-cli device remove 758d4520bd4c5efe4b399162a55639f7 
Device 758d4520bd4c5efe4b399162a55639f7 is now removed 
root@paasta-cp-suslmk-hkt:/etc/heketi# heketi-cli device delete 758d4520bd4c5efe4b399162a55639f7 
Device 758d4520bd4c5efe4b399162a55639f7 deleted 
root@paasta-cp-suslmk-hkt:/etc/heketi#
root@paasta-cp-suslmk-hkt:/etc/heketi# heketi-cli node delete 92d804890bb64596a5b462e2ad780163 
Node 92d804890bb64596a5b462e2ad780163 deleted 
root@paasta-cp-suslmk-hkt:/etc/heketi# heketi-cli node delete fdf81ce3efe6b6db832acc1470937b3c 
Node fdf81ce3efe6b6db832acc1470937b3c deleted 
root@paasta-cp-suslmk-hkt:/etc/heketi# hekete-cli cluster delete a51969941dfb603e1cb97461a4943c09 
hekete-cli: command not found 
root@paasta-cp-suslmk-hkt:/etc/heketi# heketi-cli cluster delete a51969941dfb603e1cb97461a4943c09 
Cluster a51969941dfb603e1cb97461a4943c09 deleted
root@paasta-cp-suslmk-hkt:/etc/heketi# heketi-cli topology info 
root@paasta-cp-suslmk-hkt:/etc/heketi#
root@paasta-cp-suslmk-hkt:/etc/heketi# heketi-cli topology load --user admin --secret keypassword --json=/etc/heketi/topology.json -s http://localhost:8080 
Creating cluster ... ID: 04fec28a8853fa9c0e03b07c82219186 
        Allowing file volumes on cluster. 
        Allowing block volumes on cluster. 
        Creating node swarm-1 ... ID: f4384c7f4eac1b4ec8fa69963381a5e8 
                Adding device /dev/vdb ... OK 
        Creating node swarm-2 ... ID: 0b69961e08ab618f62ee0bae7f9bd7f0 
                Adding device /dev/vdb ... OK 
        Creating node swarm-3 ... ID: 864895f25363ee5da0d22bcae0aea4a1 
                Adding device /dev/vdb ... OK 
root@paasta-cp-suslmk-hkt:/etc/heketi#

root@paasta-cp-suslmk-hkt:/etc/heketi# heketi-cli topology info 
Cluster Id: 04fec28a8853fa9c0e03b07c82219186 
    File:  true 
    Block: true 
    Volumes: 
    Nodes: 
        Node Id: 0b69961e08ab618f62ee0bae7f9bd7f0 
        State: online 
        Cluster Id: 04fec28a8853fa9c0e03b07c82219186 
        Zone: 1 
        Management Hostnames: swarm-2 
        Storage Hostnames: 172.16.10.46 
        Devices: 
                Id:69ad566e7ce567976a678c4d68f0d993   Name:/dev/vdb            State:online    Size (GiB):29      Used (GiB):0       Free (GiB):29       
                        Bricks: 
        Node Id: 864895f25363ee5da0d22bcae0aea4a1 
        State: online 
        Cluster Id: 04fec28a8853fa9c0e03b07c82219186 
        Zone: 1 
        Management Hostnames: swarm-3 
        Storage Hostnames: 172.16.10.17 
        Devices: 
                Id:ecf6ac38f1c1ce417e9993325bb7a9c0   Name:/dev/vdb            State:online    Size (GiB):29      Used (GiB):0       Free (GiB):29       
                        Bricks: 
        Node Id: f4384c7f4eac1b4ec8fa69963381a5e8 
        State: online 
        Cluster Id: 04fec28a8853fa9c0e03b07c82219186 
        Zone: 1 
        Management Hostnames: swarm-1 
        Storage Hostnames: 172.16.10.96 
        Devices: 
                Id:18aa9a08c300f0173ec423d7530f7325   Name:/dev/vdb            State:online    Size (GiB):29      Used (GiB):0       Free (GiB):29       
                        Bricks: 
root@paasta-cp-suslmk-hkt:/etc/heketi#
```


<b>volume 생성 테스트</b>
```
root@paasta-cp-suslmk-hkt:/etc/heketi# heketi-cli volume create --size=1 
Error: WARNING: This metadata update is NOT backed up. 
  /usr/sbin/thin_check: execvp failed: No such file or directory 
  WARNING: Integrity check of metadata for pool vg_18aa9a08c300f0173ec423d7530f7325/tp_caddf5e40447ef57467d03232cc1f846 failed. 
  /usr/sbin/thin_check: execvp failed: No such file or directory 
  Check of pool vg_18aa9a08c300f0173ec423d7530f7325/tp_caddf5e40447ef57467d03232cc1f846 failed (status:2). Manual repair required! 
  Failed to activate thin pool vg_18aa9a08c300f0173ec423d7530f7325/tp_caddf5e40447ef57467d03232cc1f846. 
Removal of pool metadata spare logical volume vg_18aa9a08c300f0173ec423d7530f7325/lvol0_pmspare disables automatic recovery attempts after damage to a thin or cache pool. Proceed? [y/n]: [n] 
  Logical volume vg_18aa9a08c300f0173ec423d7530f7325/lvol0_pmspare not removed. 

## gluster 노드 에 thin-provisioning-tools을 설치해야 한다.
root@paasta-cp-suslmk-hkt:/etc/heketi# apt-get -y install thin-provisioning-tools 
Reading package lists... Done 
Building dependency tree        
Reading state information... Done 
The following additional packages will be installed: 
  libaio1 
The following NEW packages will be installed: 
  libaio1 thin-provisioning-tools 
0 upgraded, 2 newly installed, 0 to remove and 1 not upgraded. 
Need to get 366 kB of archives. 
After this operation, 1382 kB of additional disk space will be used. 
Get:1 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libaio1 amd64 0.3.110-5ubuntu0.1 [6476 B] 
Get:2 http://nova.clouds.archive.ubuntu.com/ubuntu bionic/universe amd64 thin-provisioning-tools amd64 0.7.4-2ubuntu3 [359 kB] 
Fetched 366 kB in 2s (184 kB/s)                    
Selecting previously unselected package libaio1:amd64. 
(Reading database ... 60169 files and directories currently installed.) 
Preparing to unpack .../libaio1_0.3.110-5ubuntu0.1_amd64.deb ... 
Unpacking libaio1:amd64 (0.3.110-5ubuntu0.1) ... 
Selecting previously unselected package thin-provisioning-tools. 
Preparing to unpack .../thin-provisioning-tools_0.7.4-2ubuntu3_amd64.deb ... 
Unpacking thin-provisioning-tools (0.7.4-2ubuntu3) ... 
Setting up libaio1:amd64 (0.3.110-5ubuntu0.1) ... 
Setting up thin-provisioning-tools (0.7.4-2ubuntu3) ... 
Processing triggers for libc-bin (2.27-3ubuntu1.4) ... 
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
root@paasta-cp-suslmk-hkt:/etc/heketi# heketi-cli volume create --size=1 
Name: vol_752109c09f439047849be74246d41432 
Size: 1 
Volume Id: 752109c09f439047849be74246d41432 
Cluster Id: 04fec28a8853fa9c0e03b07c82219186 
Mount: 172.16.10.46:vol_752109c09f439047849be74246d41432 
Mount Options: backup-volfile-servers=172.16.10.17,172.16.10.96 
Block: false 
Free Size: 0 
Reserved Size: 0 
Block Hosting Restriction: (none) 
Block Volumes: [] 
Durability Type: replicate 
Distribute Count: 1 
Replica Count: 3 
root@paasta-cp-suslmk-hkt:/etc/heketi#
```


https://bryan.wiki/283  
https://naleejang.tistory.com/238
