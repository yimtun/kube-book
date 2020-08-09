

# 说明

当192.168.3.104 做为node 加入集群的时候不再需要手动的 

```
kubectl  certificate approve  csr-xxx
```

将上述操作自动执行





# 自动批准证书



https://v1-18.docs.kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/#approval







# 192.168.3.101



```
rm -rf /opt/bootstrap-kubelet.conf
```





```
kubectl config set-cluster kubernetes \
      --certificate-authority=/etc/kubernetes/pki/ca.crt \
      --embed-certs=true \
      --server=https://192.168.3.101:6443 \
      --kubeconfig=/opt/bootstrap-kubelet.conf
      
      
kubectl config set-credentials kubelet-bootstrap \
      --token=398c7c.b18a35f794625922 \
      --kubeconfig=/opt/bootstrap-kubelet.conf
      


kubectl config set-context kubelet-bootstrap@kubernetes \
      --cluster=kubernetes \
      --user=kubelet-bootstrap \
      --kubeconfig=/opt/bootstrap-kubelet.conf
      
      
kubectl config use-context kubelet-bootstrap@kubernetes \
--kubeconfig=/opt/bootstrap-kubelet.conf

```



```
kubectl  -n kube-system  delete  secrets  bootstrap-token-398c7c
```



```
kubectl -n kube-system create secret generic bootstrap-token-398c7c \
--type 'bootstrap.kubernetes.io/token' \
--from-literal description="cluster bootstrap token" \
--from-literal token-id=398c7c \
--from-literal token-secret=b18a35f794625922 \
--from-literal usage-bootstrap-authentication=true \
--from-literal usage-bootstrap-signing=true
```





```
kubectl delete  clusterrolebinding  kubelet-bootstrap
```





```
kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --group=system:bootstrappers	
```





```
kubectl create clusterrolebinding auto-approve-csrs-for-group \
--clusterrole=system:certificates.k8s.io:certificatesigningrequests:nodeclient \
--group=system:bootstrappers
```



清除环境 重新添加node



```
kubectl drain 192.168.3.104 --ignore-daemonsets
kubectl delete node 192.168.3.104
```

```
kubectl get csr -w
```

```
kubectl get  node  -w
```



------------------------



# 192.168.3.104







###  sys init



```
sed -i 's/enforcing/disabled/g'  /etc/selinux/config
setenforce 0
sed -i 's/#UseDNS yes/UseDN	S no/g'   /etc/ssh/sshd_config
systemctl   restart sshd
grep DNS               /etc/ssh/sshd_config
grep SELINUX=disabled  /etc/selinux/config 
systemctl  disable firewalld  NetworkManager
systemctl  stop    firewalld    NetworkManager
mv /etc/yum.repos.d/CentOS-Base.repo      /etc/yum.repos.d/CentOS-Base.repo.backup
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
curl  -o /etc/yum.repos.d/epel.repo       http://mirrors.aliyun.com/repo/epel-7.repo
```



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



###   install docker


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



```
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.2
```



###  install kubelet

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
yumdownloader --destdir  /opt/rpms/    kubernetes-cni-0.7.5-0
```

```
yumdownloader --destdir  /opt/rpms/    kubelet-1.18.3
```



```
yum -y install  /opt/rpms/*
```







###  准备kubelte 配置文件



```
rm -rf /var/lib/kubelet/pki/*
rm -rf /etc/kubernetes/kubelet.conf
rm -rf /etc/cni/net.d/10-flannel.conflist
```



-------------



#### 拷贝ca证书 

```
mkdir  /etc/kubernetes/pki/
```

```
scp  192.168.3.101:/etc/kubernetes/pki/ca.crt /etc/kubernetes/pki/
```



---------------------





#### 10-kubeadm.conf



```
mkdir -p /usr/lib/systemd/system/kubelet.service.d/
```


```
cat > /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf  << EOF
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
EnvironmentFile=-/etc/sysconfig/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet \$KUBELET_KUBECONFIG_ARGS \$KUBELET_CONFIG_ARGS \$KUBELET_KUBEADM_ARGS \$KUBELET_EXTRA_ARGS
EOF
```



---------------------

#### kubeadm-flags.env

```
mkdir /var/lib/kubelet/
```

```
cat  > /var/lib/kubelet/kubeadm-flags.env << EOF
KUBELET_KUBEADM_ARGS="--cgroup-driver=systemd --hostname-override=192.168.3.104 --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2"
EOF
```

--------------------------


#### config.yaml

```
mkdir /var/lib/kubelet/
```



```
cat  > /var/lib/kubelet/config.yaml  << EOF
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 0s
    cacheUnauthorizedTTL: 0s
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
cpuManagerReconcilePeriod: 0s
evictionPressureTransitionPeriod: 0s
fileCheckFrequency: 0s
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 0s
imageMinimumGCAge: 0s
kind: KubeletConfiguration
nodeStatusReportFrequency: 0s
nodeStatusUpdateFrequency: 0s
rotateCertificates: true
runtimeRequestTimeout: 0s
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s
EOF
```



--------------


#### bootstrap-kubelet.conf

```
scp 192.168.3.101:/opt/bootstrap-kubelet.conf   /etc/kubernetes
```

----------------

```
cat  /etc/sysconfig/kubelet
```

```
KUBELET_EXTRA_ARGS=
```

------------------







# 启动kubelet

192.168.3.104

```
systemctl daemon-reload
```



```
systemctl enable kubelet
```



```
systemctl start kubelet
```



192.168.3.101



```
kubectl get csr -w
```



```
kubectl get node -w
```





192.168.3.104



```
rm -rf /etc/kubernetes/bootstrap-kubelet.conf
```

