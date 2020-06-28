# 准备一个虚拟机



###  创建虚拟机 



192.168.3.103  作为一个node 节点 稍好将被加入到集群中



```
virsh  undefine 192.168.3.103
virsh  destroy  192.168.3.103
rm -rf /kvm/image/192.168.3.103.qcow2
systemctl restart libvirtd
```



---------------

```
/root/create_vm.sh
```



```
前置环境检查
存储池 kvmimage 中存在vol 模板文件 CentOS7.6.1810.x86_64.qcow2 检查通过继续运行
请输入新建虚拟机名称: 192.168.3.103
Vol 192.168.3.103.qcow2 cloned from CentOS7.6.1810.x86_64.qcow2

请输入新建虚拟机磁盘大小单位G: 40
开始创建虚拟磁盘
Size of volume '192.168.3.103.qcow2' successfully changed to 40G

请输入新建虚拟机内存大小单位M   : 4096
请输入新建虚拟机cpu核心数 : 2
192.168.3.101 deinfe 成功 开始设置ip
请输入为虚拟机配置的ip:  192.168.3.103
请输入为虚拟机配置的掩码: 255.255.255.0
请输入为虚拟机配置的网关: 192.168.3.1
NAME=eth0
DEVICE=eth0
ONBOOT=yes
TYPE=Ethernet
BOOTPROTO=none
IPADDR=192.168.3.103
NETMASK=255.255.255.0
GATEWAY=192.168.3.1
DNS1=223.5.5.5
请确认您的网卡配置，按y键继续?[y/n]: y
开始配置192.168.3.103 的网卡，请稍后.......
正在启动虚拟机请稍后.....
虚拟机启动成功，登录赋值如下命令即可  ssh 192.168.3.103
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







# 192.168.3.103 作为node 加入到集群

```
运行kubeadm init 输出的 加入集群为work 节点的命令
```



```
kubeadm join 192.168.3.101:6443 --node-name 192.168.3.103--token usvieq.2zg86228emymgcd4
--discovery-token-ca-cert-hash
sha256:f15e6eefb83447b09aed5e0d34b061ba2240426aec1649366153add3ca46a787 
```



# token过期 重新创建 并打印加入集群为node 节点的命令





192.168.3.101 





```
kubeadm token create --print-join-command
```



```
kubeadm join 192.168.3.101:6443 --node-name 192.168.3.103 --token gwypur.4axtvuagk01xw20w   --discovery-token-ca-cert-hash sha256:c5b32961b3001c3bf921ac95887c01fc8e50d4bc44db6a109ed532f37a959ed1
```







补充

--discovery-token-ca-cert-hash 的值 可以使用下面方式计算得出

```
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt|openssl rsa -pubin -outform der 2> /dev/null|openssl dgst -sha256 -hex | sed  's/^.* //'
```



```
c5b32961b3001c3bf921ac95887c01fc8e50d4bc44db6a109ed532f37a959ed1
```



kubeadm token create  命令相当于创建了如下 secret



```
kubectl  get -n kube-system secrets  bootstrap-token-gwypur
```

还会 将相关的信息写入 cluster-info ConfigMap

extra





# undo

192.168.3.101

```
kubectl  delete node 192.168.3.103
```



192.168.3.103

```
kubeadm  reset -f
```

