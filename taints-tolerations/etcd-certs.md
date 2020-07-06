

#  第一步创建etcd





## etcd 安装



```
virsh  undefine 192.168.3.102
virsh  destroy 192.168.3.102
rm -rf /kvm/image/192.168.3.102.qcow2
systemctl  restart libvirtd
```



##  host   

192.168.3.102





## sys init



```
sed -i 's/enforcing/disabled/g'  /etc/selinux/config
setenforce 0
sed -i 's/#UseDNS yes/UseDNS no/g'   /etc/ssh/sshd_config
systemctl   restart sshd
grep DNS               /etc/ssh/sshd_config
grep SELINUX=disabled  /etc/selinux/config 
systemctl  disable firewalld  NetworkManager
systemctl  stop    firewalld    NetworkManager
mv /etc/yum.repos.d/CentOS-Base.repo      /etc/yum.repos.d/CentOS-Base.repo.backup
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
curl  -o /etc/yum.repos.d/epel.repo       http://mirrors.aliyun.com/repo/epel-7.repo
```



##  get etcd



```
yum -y install docker
systemctl  start docker
```



```
docker pull registry.aliyuncs.com/google_containers/etcd:3.4.3-0
```



```
docker   run  --name etcd -it  registry.aliyuncs.com/google_containers/etcd:3.4.3-0  /bin/sh
```



新开一个终端 拷贝文件

```
docker cp etcd:/usr/local/bin/etcd  /usr/local/bin/
docker cp etcd:/usr/local/bin/etcdctl  /usr/local/bin/
```



```
etcd --version
```



```
docker stop etcd && docker rm etcd
```



```
systemctl  disable --now docker
```



```
yum remove docker  -y
```



## 1-node https etcd



证书准备工作



```
mkdir /etcd-certs/   -p
```



创建ca

```
cd /etcd-certs
openssl genrsa -out etcd-ca.key 2048
openssl req -x509 -new -nodes -key etcd-ca.key -subj "/CN=etcd-ca" -days 10000 -out  etcd-ca.crt
```





创建服务端证书

```
cd /etcd-certs
openssl genrsa -out etcd-server.key 2048
openssl req -new -key etcd-server.key -subj "/CN=etcd-server" -out etcd-server.csr
```







```
cat > etcd-server-csr.conf << EOF
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[ dn ]

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
IP.1 = 127.0.0.1
IP.2 = 192.168.3.102

[ v3_ext ]
authorityKeyIdentifier=keyid,issuer:always
basicConstraints=CA:FALSE
keyUsage=keyEncipherment,dataEncipherment
extendedKeyUsage=serverAuth,clientAuth
subjectAltName=@alt_names
EOF
```





```
openssl x509 -req -in etcd-server.csr  -CA etcd-ca.crt -CAkey etcd-ca.key \
-CAcreateserial -out etcd-server.crt -days 10000 \
-extensions v3_ext -extfile etcd-server-csr.conf
```





peer 证书



```
cd  /etcd-certs
openssl genrsa -out etcd-peer.key 2048
openssl req -new -key etcd-peer.key -subj "/CN=etcd-peer" -out etcd-peer.csr
```



```
cat > etcd-peer-csr.conf << EOF
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[ dn ]

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
IP.1 = 127.0.0.1
IP.2 = 192.168.3.102



[ v3_ext ]
authorityKeyIdentifier=keyid,issuer:always
basicConstraints=CA:FALSE
keyUsage=keyEncipherment,dataEncipherment
extendedKeyUsage=serverAuth,clientAuth
subjectAltName=@alt_names
EOF
```



```
openssl x509 -req -in etcd-peer.csr  \
-CA etcd-ca.crt -CAkey etcd-ca.key \
-CAcreateserial -out etcd-peer.crt -days 10000 \
-extensions v3_ext -extfile etcd-peer-csr.conf
```





客户端证书  给kube-apiserver  连接etcd服务端使用的客户端证书



```
cd  /etcd-certs
openssl genrsa -out  etcd-client.key 2048
openssl req -new -key etcd-client.key  -subj "/CN=etcd-client" -out  etcd-client.csr
```



```
cat > etcd-client-csr.conf  << EOF
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
[ dn ]
[ v3_ext ]
basicConstraints = CA:FALSE
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth
EOF
```





```
openssl x509 -req -in etcd-client.csr  \
-CA etcd-ca.crt -CAkey etcd-ca.key \
-CAcreateserial -out etcd-client.crt -days 10000 \
-extensions v3_ext -extfile etcd-client-csr.conf
```





启动etcd



```
mkdir  /var/lib/etcd
```



```
cat >   /usr/lib/systemd/system/etcd.service   << EOF
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
ExecStart=/usr/local/bin/etcd \\
--name=192.168.3.102 \\
--data-dir=/var/lib/etcd/  \\
--listen-client-urls=https://192.168.3.102:2379 \\
--advertise-client-urls=https://192.168.3.102:2379 \\
--cert-file=/etcd-certs/etcd-server.crt \\
--key-file=/etcd-certs/etcd-server.key  \\
--trusted-ca-file=/etcd-certs/etcd-ca.crt \\
--peer-cert-file=/etcd-certs/etcd-peer.crt \\
--peer-key-file=/etcd-certs/etcd-peer.key  \\
--peer-trusted-ca-file=/etcd-certs/etcd-ca.crt \\
--peer-client-cert-auth \\
--client-cert-auth 

Restart=on-failure
LimitNOFILE=65536

[Install]
EOF
```





---------------------------------------



查看写入配置



```
cat /usr/lib/systemd/system/etcd.service
```



```
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
ExecStart=/usr/local/bin/etcd \
--name=192.168.3.102 \
--data-dir=/var/lib/etcd/  \
--listen-client-urls=https://192.168.3.102:2379 \
--advertise-client-urls=https://192.168.3.102:2379 \
--cert-file=/etcd-certs/etcd-server.crt \
--key-file=/etcd-certs/etcd-server.key  \
--trusted-ca-file=/etcd-certs/etcd-ca.crt \
--peer-cert-file=/etcd-certs/etcd-peer.crt \
--peer-key-file=/etcd-certs/etcd-peer.key  \
--peer-trusted-ca-file=/etcd-certs/etcd-ca.crt \
--peer-client-cert-auth \
--client-cert-auth 

Restart=on-failure
LimitNOFILE=65536

[Install]
```





```
systemctl  enable --now etcd
systemctl  status  etcd
```



-----------------------------------





验证



```
ETCDCTL_API=3 etcdctl \
--endpoints=https://192.168.3.102:2379 \
--cacert=/etcd-certs/etcd-ca.crt \
--cert=/etcd-certs/etcd-client.crt \
--key=/etcd-certs/etcd-client.key endpoint health
```



```
https://192.168.3.102:2379 is healthy: successfully committed proposal: took = 33.203686ms
```



----------------------------









# 第二步 使用外置etcd 部署k8s



```
virsh  undefine 192.168.3.101
virsh  destroy 192.168.3.101
rm -rf /kvm/image/192.168.3.101.qcow2
systemctl  restart libvirtd
```



192.168.3.101



###  创建虚拟机



```
/root/create_vm.sh
```



```
/root/create_vm.sh 
前置环境检查
存储池 kvmimage 中存在vol 模板文件 CentOS7.6.1810.x86_64.qcow2 检查通过继续运行
请输入新建虚拟机名称: 192.168.3.101
Vol 192.168.3.101.qcow2 cloned from CentOS7.6.1810.x86_64.qcow2

请输入新建虚拟机磁盘大小单位G: 40
开始创建虚拟磁盘
Size of volume '192.168.3.101.qcow2' successfully changed to 40G

请输入新建虚拟机内存大小单位M   : 4096
请输入新建虚拟机cpu核心数 : 2
192.168.3.101 deinfe 成功 开始设置ip
请输入为虚拟机配置的ip:  192.168.3.101
请输入为虚拟机配置的掩码: 255.255.255.0
请输入为虚拟机配置的网关: 192.168.3.1
NAME=eth0
DEVICE=eth0
ONBOOT=yes
TYPE=Ethernet
BOOTPROTO=none
IPADDR=192.168.3.101
NETMASK=255.255.255.0
GATEWAY=192.168.3.1
DNS1=223.5.5.5
请确认您的网卡配置，按y键继续?[y/n]: y
开始配置192.168.3.101 的网卡，请稍后.......
正在启动虚拟机请稍后.....
虚拟机启动成功，登录赋值如下命令即可  ssh 192.168.3.101
```





---------------------------



###  系统初始化

关闭selinux

```
sed -i 's/enforcing/disabled/g'  /etc/selinux/config
setenforce 0
sed -i 's/#UseDNS yes/UseDNS no/g'   /etc/ssh/sshd_config
systemctl   restart sshd
grep DNS               /etc/ssh/sshd_config
grep SELINUX=disabled  /etc/selinux/config 
systemctl  disable firewalld  NetworkManager
systemctl  stop    firewalld    NetworkManager
mv /etc/yum.repos.d/CentOS-Base.repo      /etc/yum.repos.d/CentOS-Base.repo.backup
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
curl  -o /etc/yum.repos.d/epel.repo       http://mirrors.aliyun.com/repo/epel-7.repo
```



时间相关

```
timedatectl set-timezone Asia/Shanghai
yum -y install ntpdate
ntpdate ntp1.aliyun.com
timedatectl set-local-rtc 0
systemctl restart rsyslog
systemctl restart crond
yum -y install ntp
```



```
cat >   /etc/ntp.conf  << EOF
driftfile /var/lib/ntp/drift
restrict default nomodify notrap nopeer noquery
restrict 127.0.0.1 
restrict ::1
server ntp1.aliyun.com iburst  iburst
logfile /var/log/ntp.log
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
disable monitor
EOF
```



```
systemctl  enable  --now ntpd
ntpq -p
```


###  配置必要的内核参数

```
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```



```
modprobe br_netfilter
sysctl -p /etc/sysctl.d/k8s.conf
```





###  docker 安装与配置

```
yum -y install yum-utils
```



```
yum-config-manager --add-repo \
https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```



```
mkdir  /etc/docker
```



```
cat > /etc/docker/daemon.json  << EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": ["https://2xdz2l32.mirror.aliyuncs.com"]
}
EOF
```





```
yum -y install docker-ce-18.06.1.ce-3.el7
systemctl  enable --now docker
```





###  k8s 安装基础组件




```
cat > /etc/yum.repos.d/kubernetes.repo <<  EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
EOF
```





```
mkdir /opt/rpms/
```



```
yumdownloader --destdir  /opt/rpms/   kubernetes-cni-0.7.5-0
```



```
yumdownloader --destdir  /opt/rpms/    kubelet-1.18.3
```



```
yumdownloader --destdir  /opt/rpms/    cri-tools-1.13.0
```



```
yumdownloader --destdir  /opt/rpms/    kubectl-1.18.3
```



```
yumdownloader --destdir  /opt/rpms/    kubeadm-1.18.3
```



```
yum -y install  /opt/rpms/*
```



```
systemctl enable kubelet.service
```





```
kubectl  version
```

```
kubeadm  version
```

```
kubelet  --version
```



```
crictl   --version
```



###  kubeclt 命令补全



```
yum   -y install bash-completion
```



```
source /usr/share/bash-completion/bash_completion
```



```
echo 'source <(kubectl completion bash)'     >>  /root/.bashrc
```



```
source  /root/.bashrc
```



测试  复制下面内容到终端按 tab 测试是否自动补全

```
kubectl  desc
```









### 配置集群参数文件



```
cat > /opt/kube-1.18.3-config.yaml << EOF
apiVersion: kubeadm.k8s.io/v1beta2
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 0.0.0.0
  bindPort: 6443
nodeRegistration:
  name: 192.168.3.101
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.18.3
imageRepository: registry.aliyuncs.com/google_containers
controlPlaneEndpoint: 192.168.3.101:6443
controllerManager: {}
scheduler: {}
etcd:
    external:
        endpoints:
        - https://192.168.3.102:2379
        caFile: /etc/etcd/certs/etcd-ca.crt
        certFile: /etc/etcd/certs/etcd-client.crt
        keyFile: /etc/etcd/certs/etcd-client.key
networking:
  serviceSubnet: 10.96.0.0/12
  podSubnet: 10.244.0.0/16
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs
EOF
```



###  设置etcd 证书



拷贝etcd客户端证书 

```
mkdir -p /etc/etcd/certs/
```



```
scp  192.168.3.102:/etcd-certs/{etcd-ca.crt,etcd-client.crt,etcd-client.key}    /etc/etcd/certs/
```



拷贝etcd客户端工具 etcdctl  方便调试

```
scp  192.168.3.102:/usr/local/bin/etcdctl    /usr/local/bin/
```



在 api server 上使用命令行客户端 工具 etcdctl  进行连接测试

```
ETCDCTL_API=3 etcdctl \
--endpoints=https://192.168.3.102:2379 \
--cacert=/etc/etcd/certs/etcd-ca.crt \
--cert=/etc/etcd/certs/etcd-client.crt \
--key=/etc/etcd/certs/etcd-client.key endpoint health
```



```
https://192.168.3.102:2379 is healthy: successfully committed proposal: took = 41.89946ms
```





查看etcd 数据



```
ETCDCTL_API=3 etcdctl \
--endpoints=https://192.168.3.102:2379 \
--cacert=/etc/etcd/certs/etcd-ca.crt \
--cert=/etc/etcd/certs/etcd-client.crt \
--key=/etc/etcd/certs/etcd-client.key  \
get / --prefix --keys-only
```





删除所有数据   重新装的时候  etcd  不必重新安装  清除数据即可

安装前如果查看当前etcd 有数据使用下面命令删除

```
ETCDCTL_API=3 etcdctl \
--endpoints=https://192.168.3.102:2379 \
--cacert=/etc/etcd/certs/etcd-ca.crt \
--cert=/etc/etcd/certs/etcd-client.crt \
--key=/etc/etcd/certs/etcd-client.key  \
del / --prefix
```





### 创建k8s 需要的证书



#####  ca

```
mkdir /my-certs
cd /my-certs
openssl genrsa -out ca.key 2048
openssl req -x509 -new -nodes -key ca.key -subj "/CN=kubernetes" -days 10000 -out ca.crt
```

##### apiserver.crt

```
cd /my-certs
openssl genrsa -out apiserver.key 2048
openssl req -new -key apiserver.key -subj "/CN=kube-apiserver" -out  apiserver.csr
```

```
cat > apiserver-csr.conf << EOF
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[ dn ]


[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
IP.1 = 10.96.0.1
IP.2 = 192.168.3.101


[ v3_ext ]
authorityKeyIdentifier=keyid,issuer:always
basicConstraints=CA:FALSE
keyUsage=keyEncipherment,dataEncipherment
extendedKeyUsage=serverAuth
subjectAltName=@alt_names
EOF
```



```
openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key \
-CAcreateserial -out apiserver.crt -days 10000 \
-extensions v3_ext -extfile apiserver-csr.conf
```



##### apiserver-kubelet-client.crt

```
cd /my-certs
openssl genrsa -out apiserver-kubelet-client.key 2048
openssl req -new -key apiserver-kubelet-client.key  -subj "/O=system:masters/CN=kube-apiserver-kubelet-client" -out apiserver-kubelet-client.csr
```



```
cat > client.ext << EOF
extendedKeyUsage=clientAuth
EOF
```

```
openssl x509 -req -in apiserver-kubelet-client.csr  -CA ca.crt -CAkey ca.key -CAcreateserial -extfile client.ext -out  apiserver-kubelet-client.crt -days 5000
```



##### front-proxy-ca.crt

```
cd /my-certs
openssl genrsa -out front-proxy-ca.key 2048	
openssl req -x509 -new -nodes -key front-proxy-ca.key -subj "/CN=front-proxy-ca" -days 10000 -out front-proxy-ca.crt
```



##### 	front-proxy-client.crt

````
cd /my-certs
openssl genrsa -out front-proxy-client.key 2048
openssl req -new -key front-proxy-client.key  -subj "/CN=front-proxy-client" -out front-proxy-client.csr
````

```
cat > client.ext << EOF
extendedKeyUsage=clientAuth
EOF
```

```
openssl x509 -req -in front-proxy-client.csr  -CA front-proxy-ca.crt -CAkey front-proxy-ca.key  -CAcreateserial -extfile client.ext -out front-proxy-client.crt -days 5000
```

##### service  acount

```
cd /my-certs
openssl  genrsa -out sa.key 2046
openssl  rsa -in sa.key  -pubout -out sa.pub
```

#####  部署证书

```
mkdir -p /etc/kubernetes/pki/
```

```
cp /my-certs/{apiserver.crt,apiserver.key,apiserver-kubelet-client.crt,apiserver-kubelet-client.key}    /etc/kubernetes/pki/
```

```
cp /my-certs/{ca.crt,ca.key,front-proxy-ca.crt,front-proxy-ca.key}  /etc/kubernetes/pki/
```

```
cp /my-certs/{front-proxy-client.crt,front-proxy-client.key,sa.key,sa.pub} /etc/kubernetes/pki/
```



###  kubeadm 安装k8s



```
kubeadm config images  list --config  /opt/kube-1.18.3-config.yaml
```



```
kubeadm config images  pull  --config  /opt/kube-1.18.3-config.yaml
```





```
kubeadm  init --config  /opt/kube-1.18.3-config.yaml
```



查看etcd 是否被写入数据

```
ETCDCTL_API=3 etcdctl \
--endpoints=https://192.168.3.102:2379 \
--cacert=/etc/etcd/certs/etcd-ca.crt \
--cert=/etc/etcd/certs/etcd-client.crt \
--key=/etc/etcd/certs/etcd-client.key  \
get / --prefix --keys-only
```

```
mkdir  /root/.kube
cp /etc/kubernetes/admin.conf  /root/.kube/config
```





----------

```
kubectl  get node
```

```
NAME            STATUS     ROLES    AGE   VERSION
192.168.3.101   NotReady   master   18s   v1.18.3
```

--------------





```
kubectl  get pod -A
```



```
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
kube-system   coredns-7ff77c879f-ggms2                0/1     Pending   0          19s
kube-system   coredns-7ff77c879f-rpkzr                0/1     Pending   0          19s
kube-system   kube-apiserver-192.168.3.101            1/1     Running   0          29s
kube-system   kube-controller-manager-192.168.3.101   1/1     Running   0          29s
kube-system   kube-proxy-wfxrl                        1/1     Running   1          19s
kube-system   kube-scheduler-192.168.3.101            1/1     Running   0          29s
```

--------------





网络两个选项选择一个安装即可

### 网络选项1 

### 安装  calico



```
docker pull  calico/cni:v3.14.0
```

```
docker pull  calico/node:v3.14.0
```

```
docker pull calico/pod2daemon-flexvol:v3.14.0
```

```
docker pull calico/kube-controllers:v3.14.0
```



```
yum -y install git
git clone https://github.com/yimtun/kube-config.git  /opt/kube-config
```

```
kubectl  apply  -f /opt/kube-config/calico/calico-v3.14.0/calico.yaml
```



或者直接使用官网 yaml

```
kubectl apply -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml
```



```
kubectl  delete  -f /opt/kube-config/calico/calico-v3.14.0/calico.yaml
```





### 网络选项2 

### 安装flannel



```
docker pull quay.io/coreos/flannel:v0.12.0-amd64
```



> https://raw.githubusercontent.com/coreos/flannel/v0.12.0/Documentation/kube-flannel.yml



```
yum -y install git
git clone https://github.com/yimtun/kube-config.git  /opt/kube-config
```



```
kubectl  apply  -f /opt/kube-config/flannel/flannel-v0.12.0/flannel-v0.12.0-amd64.yaml
```



```
kubectl  get node
```



```
NAME            STATUS   ROLES    AGE     VERSION
192.168.3.101   Ready    master   4m31s   v1.18.3
```





```
kubectl  get pod -A
```



```
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
kube-system   coredns-7ff77c879f-ggms2                0/1     Running   0          90s
kube-system   coredns-7ff77c879f-rpkzr                0/1     Running   0          90s
kube-system   kube-apiserver-192.168.3.101            1/1     Running   0          100s
kube-system   kube-controller-manager-192.168.3.101   1/1     Running   0          100s
kube-system   kube-flannel-ds-amd64-m9qzj             1/1     Running   0          18s
kube-system   kube-proxy-wfxrl                        1/1     Running   1          90s
kube-system   kube-scheduler-192.168.3.101            1/1     Running   0          100s
```









### install metrics-server



```
yum -y install git
git clone https://github.com/yimtun/metrics-yaml.git   /opt/metrics-yaml
```





```
docker pull yimtune/metrics-server-amd64:v0.3.6
docker tag  yimtune/metrics-server-amd64:v0.3.6  k8s.gcr.io/metrics-server-amd64:v0.3.6
```



```
kubectl  create -f /opt/metrics-yaml/1.8+/
```



```
kubectl  get pod -A
```



```
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
kube-system   coredns-7ff77c879f-ggms2                1/1     Running   0          2m24s
kube-system   coredns-7ff77c879f-rpkzr                1/1     Running   0          2m24s
kube-system   kube-apiserver-192.168.3.101            1/1     Running   0          2m34s
kube-system   kube-controller-manager-192.168.3.101   1/1     Running   0          2m34s
kube-system   kube-flannel-ds-amd64-m9qzj             1/1     Running   0          72s
kube-system   kube-proxy-wfxrl                        1/1     Running   1          2m24s
kube-system   kube-scheduler-192.168.3.101            1/1     Running   0          2m34s
kube-system   metrics-server-8c4f4d7c4-qv7z5          0/1     Pending   0          4s
```





```
kubectl  get event -n kube-system
```

```
8s          Warning   FailedScheduling    Pod          0/1 nodes are available: 1 node(s) had taints that the pod didn't tolerate.
34s         Normal    SuccessfulCreate    ReplicaSet   Created pod: metrics-server-5df586cb87-mvxqk
34s         Normal    ScalingReplicaSet   Deployment   Scaled up replica set metrics-server-5df586cb87 to 1
```





```
kubectl taint node   192.168.3.101   node-role.kubernetes.io/master:NoSchedule-
```





```
kubectl  get event -n kube-system
```



```
2s          Normal    Scheduled           Pod          Successfully assigned kube-system/metrics-server-5df586cb87-mvxqk to 172.16.99.100
0s          Normal    Pulled              Pod          Container image "k8s.gcr.io/metrics-server-amd64:v0.3.6" already present on machine
0s          Normal    Created             Pod          Created container
```





```
kubectl  top node
```



```
NAME            CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
192.168.3.101   133m         6%     1481Mi          40%    
```





```
kubectl  top  pod -n kube-system
```



```
NAME                                    CPU(cores)   MEMORY(bytes)   
coredns-7ff77c879f-8hvrm                2m           9Mi             
coredns-7ff77c879f-bfd68                2m           10Mi            
kube-apiserver-192.168.3.101            23m          369Mi           
kube-controller-manager-192.168.3.101   7m           38Mi            
kube-flannel-ds-amd64-5lkzz             2m           8Mi             
kube-proxy-7nzdc                        1m           12Mi            
kube-scheduler-192.168.3.101            3m           15Mi            
metrics-server-8c4f4d7c4-zrvs2          1m           11Mi      
```





----------------------

###  undo





```
kubeadm  reset -f
rm -rf /etc/kubernetes
rm -rf /var/lib/kubelet
rm -rf /root/.kube
```



```
ETCDCTL_API=3 etcdctl \
--endpoints=https://192.168.3.102:2379 \
--cacert=/etc/etcd/certs/etcd-ca.crt \
--cert=/etc/etcd/certs/etcd-client.crt \
--key=/etc/etcd/certs/etcd-client.key  \
del / --prefix
```





