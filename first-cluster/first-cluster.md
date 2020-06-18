k8s 版本 1.18.3

操作系统  CentOS7.6



单节点 

ip



192.168.3.101



# 创建虚拟机



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
yum list  kubeadm  --showduplicates | sort -r
```






```
yum  -y install   kubelet-1.18.3
```

```
yum  -y install   kubectl-1.18.3
```

```
yum -y install    kubeadm-1.18.3
```



```
systemctl enable kubelet.service
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







### 配置集群参数





```
kubeadm config images list --v=5 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.18.3 
```



```
registry.aliyuncs.com/google_containers/kube-apiserver:v1.18.3
registry.aliyuncs.com/google_containers/kube-controller-manager:v1.18.3
registry.aliyuncs.com/google_containers/kube-scheduler:v1.18.3
registry.aliyuncs.com/google_containers/kube-proxy:v1.18.3
registry.aliyuncs.com/google_containers/pause:3.2
registry.aliyuncs.com/google_containers/etcd:3.4.3-0
registry.aliyuncs.com/google_containers/coredns:1.6.7
```



```
kubeadm config images pull --v=5 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.18.3 
```



###  命令行安装





```
kubeadm init \
--apiserver-advertise-address=192.168.3.101 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.18.3 \
--service-cidr=10.1.0.0/16 \
--pod-network-cidr=10.244.0.0/16 \
--node-name=192.168.3.101 
```





```
mkdir  /root/.kube
cp /etc/kubernetes/admin.conf  /root/.kube/config
```







```
kubectl  get node
```



```
NAME            STATUS     ROLES    AGE   VERSION
172.16.99.100   NotReady   master   94s   v1.12.3
```







```
kubectl  get pod -A
```



```
NAME                                    READY   STATUS    RESTARTS   AGE
coredns-6c66ffc55b-5ff2z                0/1     Pending   0          94s
coredns-6c66ffc55b-h6x6n                0/1     Pending   0          94s
etcd-172.16.99.100                      1/1     Running   0          52s
kube-apiserver-172.16.99.100            1/1     Running   0          65s
kube-controller-manager-172.16.99.100   1/1     Running   0          40s
kube-proxy-hhj2b                        1/1     Running   0          94s
kube-scheduler-172.16.99.100            1/1     Running   0          51s
```







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
172.16.99.100   Ready    master   8m45s   v1.12.3
```



```
kubectl  get pod -A
```



```
NAME                                    READY   STATUS    RESTARTS   AGE
coredns-6c66ffc55b-5ff2z                1/1     Running   0          8m54s
coredns-6c66ffc55b-h6x6n                1/1     Running   0          8m54s
etcd-172.16.99.100                      1/1     Running   0          8m12s
kube-apiserver-172.16.99.100            1/1     Running   0          8m25s
kube-controller-manager-172.16.99.100   1/1     Running   0          8m
kube-flannel-ds-amd64-8qsks             1/1     Running   0          43s
kube-proxy-hhj2b                        1/1     Running   0          8m54s
kube-scheduler-172.16.99.100            1/1     Running   0          8m11s
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
NAME                                    READY   STATUS    RESTARTS   AGE
coredns-6c66ffc55b-5ff2z                1/1     Running   0          12m
coredns-6c66ffc55b-h6x6n                1/1     Running   0          12m
etcd-172.16.99.100                      1/1     Running   0          11m
kube-apiserver-172.16.99.100            1/1     Running   0          11m
kube-controller-manager-172.16.99.100   1/1     Running   0          11m
kube-flannel-ds-amd64-8qsks             1/1     Running   0          3m50s
kube-proxy-hhj2b                        1/1     Running   0          12m
kube-scheduler-172.16.99.100            1/1     Running   0          11m
metrics-server-5df586cb87-mvxqk         0/1     Pending   0          22s
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
172.16.99.100   854m         10%    1509Mi          9%    
```





```
kubectl  top  pod -n kube-system
```



```
NAME                                       CPU(cores)   MEMORY(bytes)   
calico-kube-controllers-789f6df884-ntpgx   9m           12Mi            
calico-node-p4qnd                          101m         44Mi            
coredns-7ff77c879f-9cng7                   8m           13Mi            
coredns-7ff77c879f-d6sf2                   11m          12Mi            
etcd-172.16.99.100                         44m          48Mi            
kube-apiserver-172.16.99.100               120m         404Mi           
kube-controller-manager-172.16.99.100      42m          46Mi            
kube-proxy-nv7x7                           1m           14Mi            
kube-scheduler-172.16.99.100               14m          18Mi            
metrics-server-8c4f4d7c4-tn8p2             3m           13Mi          
```







> https://blog.csdn.net/l1028386804/article/details/105904557/

###  undo



```
kubeadm  reset -f
```



```
rm -rf /root/.kube/
rm -rf /etc/kubernetes/
rm -rf /var/lib/kubelet
rm -rf /var/lib/etcd/
```



