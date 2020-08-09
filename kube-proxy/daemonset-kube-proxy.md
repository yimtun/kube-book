

# 本节目标

二进制安装kube-proxy  以替换当前以ds 方式运行的kube-proxy







# 环境搭建



## 1 安装master



192.168.3.101



```
systemctl enable kubelet
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
bootstrapTokens:
  - token: usvieq.2zg86228emymgcd4
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





###  部署证书



上传  my-certs.zip 到虚拟机 /root/   下



```
yum -y install unzip
```



```
unzip  /root/my-certs.zip   -d /
```



```
mkdir -p /etc/kubernetes/pki/
```



```
cp /my-certs/{apiserver.crt,apiserver.key,apiserver-kubelet-client.crt,apiserver-kubelet-client.key}    /etc/kubernetes/pki/
cp /my-certs/{ca.crt,ca.key,front-proxy-ca.crt,front-proxy-ca.key}  /etc/kubernetes/pki/
cp /my-certs/{front-proxy-client.crt,front-proxy-client.key,sa.key,sa.pub} /etc/kubernetes/pki/
```



-------------

```
ls /etc/kubernetes/pki/
```



```
apiserver.crt  apiserver-kubelet-client.crt  ca.crt  front-proxy-ca.crt  front-proxy-client.crt  sa.key
apiserver.key  apiserver-kubelet-client.key  ca.key  front-proxy-ca.key  front-proxy-client.key  sa.pub
```



###  kubeadm 安装k8s



```
kubeadm config images  list --config  /opt/kube-1.18.3-config.yaml
```



```
kubeadm config images  pull  --config  /opt/kube-1.18.3-config.yaml
```



------------

```
kubeadm  init --config  /opt/kube-1.18.3-config.yaml
```



```
Your Kubernetes control-plane has initialized successfully!
```



----------





```
mkdir  /root/.kube
cp /etc/kubernetes/admin.conf  /root/.kube/config
```



----------------------





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
kubectl  get node
```





```
kubectl  get pod -A
```



```
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
kube-system   coredns-7ff77c879f-66ts8                0/1     Pending   0          54s
kube-system   coredns-7ff77c879f-dpv97                0/1     Pending   0          54s
kube-system   kube-apiserver-192.168.3.101            1/1     Running   0          63s
kube-system   kube-controller-manager-192.168.3.101   1/1     Running   0          63s
kube-system   kube-proxy-qn66v                        1/1     Running   1          54s
kube-system   kube-scheduler-192.168.3.101            1/1     Running   0          63s
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



------------------

```
kubectl  get node
```



```
NAME            STATUS   ROLES    AGE     VERSION
192.168.3.101   Ready    master   4m31s   v1.18.3
```

-----------------





```
kubectl  get pod -A
```



```
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
kube-system   coredns-7ff77c879f-66ts8                1/1     Running   0          2m19s
kube-system   coredns-7ff77c879f-dpv97                1/1     Running   0          2m19s
kube-system   kube-apiserver-192.168.3.101            1/1     Running   0          2m28s
kube-system   kube-controller-manager-192.168.3.101   1/1     Running   0          2m28s
kube-system   kube-flannel-ds-amd64-6g628             1/1     Running   0          27s
kube-system   kube-proxy-qn66v                        1/1     Running   1          2m19s
kube-system   kube-scheduler-192.168.3.101            1/1     Running   0          2m28s
```



--------------



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
kube-system   coredns-7ff77c879f-66ts8                1/1     Running   0          3m23s
kube-system   coredns-7ff77c879f-dpv97                1/1     Running   0          3m23s
kube-system   kube-apiserver-192.168.3.101            1/1     Running   0          3m32s
kube-system   kube-controller-manager-192.168.3.101   1/1     Running   0          3m32s
kube-system   kube-flannel-ds-amd64-6g628             1/1     Running   0          91s
kube-system   kube-proxy-qn66v                        1/1     Running   1          3m23s
kube-system   kube-scheduler-192.168.3.101            1/1     Running   0          3m32s
kube-system   metrics-server-8c4f4d7c4-6bclh          0/1     Pending   0          4s
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

最好重新部署虚拟机  网络资源 kubeadm reset 清理的不干净



```
kubeadm  reset -f
rm -rf /etc/kubernetes
rm -rf /var/lib/kubelet
rm -rf /root/.kube
rm -rf /etc/cni/net.d/*
```



```
ETCDCTL_API=3 etcdctl \
--endpoints=https://192.168.3.102:2379 \
--cacert=/etc/etcd/certs/etcd-ca.crt \
--cert=/etc/etcd/certs/etcd-client.crt \
--key=/etc/etcd/certs/etcd-client.key  \
del / --prefix
```



```
reboot
```



-----------------------





##  2 手动添加节点前的准备



### 192.168.3.101




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
kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --group=system:bootstrappers	
```



```
kubectl create clusterrolebinding auto-approve-csrs-for-group \
--clusterrole=system:certificates.k8s.io:certificatesigningrequests:nodeclient \
--group=system:bootstrappers
```



```
kubectl create clusterrolebinding auto-approve-renewals-for-nodes \
--clusterrole=system:certificates.k8s.io:certificatesigningrequests:selfnodeclient \
--group=system:nodes
```



-------------------------



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



## 3 手动添加一个node

192.168.3.104



```
docker pull registry.aliyuncs.com/google_containers/pause:3.2
```



```
docker pull registry.aliyuncs.com/google_containers/kube-proxy:v1.18.3
```



```
docker pull quay.io/coreos/flannel:v0.12.0-amd64
```



### ca.crt



```
mkdir  /etc/kubernetes/pki/
```

```
scp  192.168.3.101:/etc/kubernetes/pki/ca.crt /etc/kubernetes/pki/
```



### 10-kubeadm.conf



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





### kubeadm-flags.env





```
mkdir /var/lib/kubelet/
```



```
cat  > /var/lib/kubelet/kubeadm-flags.env << EOF
KUBELET_KUBEADM_ARGS="--cgroup-driver=systemd --hostname-override=192.168.3.104 --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2"
EOF
```





###  config.yaml



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



###  bootstrap-kubelet.conf

```
scp 192.168.3.101:/opt/bootstrap-kubelet.conf   /etc/kubernetes
```





```
cat  > /etc/sysconfig/kubelet << EOF
KUBELET_EXTRA_ARGS=
EOF
```



-----------------



```
cat /usr/lib/systemd/system/kubelet.service
```



```
[Unit]
Description=kubelet: The Kubernetes Node Agent
Documentation=https://kubernetes.io/docs/

[Service]
ExecStart=/usr/bin/kubelet
Restart=always
StartLimitInterval=0
RestartSec=10

[Install]
WantedBy=multi-user.target
```



### 	启动kubelet



```
systemctl enable kubelet
systemctl start kubelet
```



## 4 在master 节点上查看



### 192.168.3.101



```
kubectl get node 
```



# 查看当前kube-proxy配置信息

192.168.3.101



kubeadm 安装的k8s 集群中  kube-proxy 组件是以daemonset 运行的

其本质还是运行在pod 中，kube-proxy 也需要调用api server

在pod 里面的运行的kube-proxy 使用 serviceAccount 作为凭证来调用api server

serviceAccount 是区分命名空间的



查找名字为kube-proxy  的daemonset  使用的 serviceaccount的名字



```
yum -y install jq
```



```
kubectl  -n kube-system  get ds kube-proxy -o json | jq .spec.template.spec.serviceAccount
```



查找到 名字为kube-proxy  的daemonset  使用的 serviceaccount的名字是 kube-proxy



查看 名字为  kube-proxy  的  serviceaccount

```
kubectl  get -n kube-system  sa kube-proxy -o yaml
```



```
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: "2020-07-08T13:33:01Z"
  name: kube-proxy
  namespace: kube-system
  resourceVersion: "1723600"
  selfLink: /api/v1/namespaces/kube-system/serviceaccounts/kube-proxy
  uid: 41dcb09b-4470-4804-9277-1f3446da5141
secrets:
- name: kube-proxy-token-t64cc
```

在命名空间kube-system 下 名字为 kube-proxy 的ServiceAccount 里有一个 secret

secret 的名字是 kube-proxy-token-t64cc



```
kubectl  -n kube-system  get secrets kube-proxy-token-t64cc  -o yaml
```



这个secret 的类型是 kubernetes.io/service-account-token

```
type: kubernetes.io/service-account-token
```







### 查看哪些 clusterrolebinding 中存在名字中包含 kube-proxy  的subject



```
kubectl get clusterrolebinding -o custom-columns='Name:metadata.name,RoleRef:roleRef.name,Subjects:subjects'  | grep 'kube-proxy'
```





```
kubeadm:node-proxier                                   system:node-proxier                                                    [map[kind:ServiceAccount name:kube-proxy namespace:kube-system]]
system:node-proxier                                    system:node-proxier                                                    [map[apiGroup:rbac.authorization.k8s.io kind:User name:system:kube-proxy]]
```

--------------





| clusterrolebinding   | clusterrole         | subjects kind  | subjects name     | ns          |
| -------------------- | ------------------- | -------------- | ----------------- | ----------- |
| kubeadm:node-proxier | system:node-proxier | ServiceAccount | kube-proxy        | kube-system |
| system:node-proxier  | system:node-proxier | User           | system:kube-proxy |             |



这两个 clusterrolebinding 里的clusterrole 是同一个 

--------------







### 查看哪些 rolebinding 中存在名字中包含 kube-proxy  的subject



```
kubectl get rolebinding -A -o custom-columns='Name:metadata.name,RoleRef:roleRef.name,Subjects:subjects'  | grep 'kube-proxy'
```



```
kube-proxy                                          kube-proxy                                       [map[apiGroup:rbac.authorization.k8s.io kind:Group name:system:bootstrappers:kubeadm:default-node-token]]
```





| rolebinding | namespace   | role       | subjects kind | subjects name                                   |
| ----------- | ----------- | ---------- | ------------- | ----------------------------------------------- |
| kube-proxy  | kube-system | kube-proxy | Group         | system:bootstrappers:kubeadm:default-node-token |



-----------------

```
kubectl get role -n kube-system  kube-proxy  -o yaml
```





```
rules:
- apiGroups:
  - ""
  resourceNames:
  - kube-proxy
  resources:
  - configmaps
  verbs:
  - get
```





这里时间是匹配的并不是 subject 里的内容  而是匹配到的  rolebinding 的名字和  role的名字











### kube-proxy 正常运行需要的 权限是  名字为  system:node-proxier  的 clusterrole







# 查看kube-proxy 当前运行参数





192.168.3.101



```
docker ps  --format "table {{.ID}}\t{{.Command}}"  --no-trunc | grep kube-proxy
```



```
9be91024d915af1631d5fa831fd94c46286d930b8548ba2ab45db014aa983355   "/usr/local/bin/kube-proxy --config=/var/lib/kube-proxy/config.conf --hostname-override=192.168.3.101"
```





-------------------



### 	config.conf

查看kube-proxy 的配置文件   exec 后为上一条命令输出的前面的id  



```
docker exec fc1c607  cat /var/lib/kube-proxy/config.conf
```



```
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
clientConnection:
  acceptContentTypes: ""
  burst: 0
  contentType: ""
  kubeconfig: /var/lib/kube-proxy/kubeconfig.conf
  qps: 0
clusterCIDR: 10.244.0.0/16
configSyncPeriod: 0s
conntrack:
  maxPerCore: null
  min: null
  tcpCloseWaitTimeout: null
  tcpEstablishedTimeout: null
detectLocalMode: ""
enableProfiling: false
healthzBindAddress: ""
hostnameOverride: ""
iptables:
  masqueradeAll: false
  masqueradeBit: null
  minSyncPeriod: 0s
  syncPeriod: 0s
ipvs:
  excludeCIDRs: null
  minSyncPeriod: 0s
  scheduler: ""
  strictARP: false
  syncPeriod: 0s
  tcpFinTimeout: 0s
  tcpTimeout: 0s
  udpTimeout: 0s
kind: KubeProxyConfiguration
metricsBindAddress: ""
mode: ipvs
nodePortAddresses: null
oomScoreAdj: null
portRange: ""
showHiddenMetricsForVersion: ""
udpIdleTimeout: 0s
winkernel:
  enableDSR: false
  networkName: ""
  sourceVip: ""
```



--------



### kubeconfig.conf



```
docker exec fc1c607  cat /var/lib/kube-proxy/kubeconfig.conf
```



```
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    server: https://192.168.3.101:6443
  name: default
contexts:
- context:
    cluster: default
    namespace: default
    user: default
  name: default
current-context: default
users:
- name: default
  user:
    tokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
```



--------------



###  token





```
docker exec fc1c607  cat /var/run/secrets/kubernetes.io/serviceaccount/token
```



```
eyJhbGciOiJSUzI1NiIsImtpZCI6Ikk1VzNwT3prNGx5YVd2bnVqNUpRMlJrRWNoenZHZWpCWHF3QmpVNmpGSkUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlLXByb3h5LXRva2VuLXQ2NGNjIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6Imt1YmUtcHJveHkiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI0MWRjYjA5Yi00NDcwLTQ4MDQtOTI3Ny0xZjM0NDZkYTUxNDEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06a3ViZS1wcm94eSJ9.J0agLxhkuhIG3jSGofmwLfuR2kVM2tA-jtfhDbwZZrDLyneomyGP0UrAV747RGOdBpY756yy2KZIra-PhmwNj9mLJo9eVoKaok6gH288YvgNi-ul0nUCkS0nwHNJnx__QeDlgHCtUmdiy0pd1cIM3J_NsUop2onMyo3JqkKAYoEBw_zfzXTZ-gk5ogILDkH-tT4LKv-YNsVrZK9CllXjtDg8GDTBzQ17O8WcA4XmcIgqtS8OlhhyJxSZpjnFM280JZBsTWd6X3wYgmFxH0LCEpc7EvRnnIWUiAfAv29J955ik82pXcJMfJ_d7llLEWFE1TaYAeNThUF52tI3XvSiCg
```



这里存放的值就是 之前追踪到的那个   kube-proxy-token-t64cc  的secret  中存放的token

```
kubectl  -n kube-system  get secrets kube-proxy-token-t64cc  -o yaml
```



```
  namespace: a3ViZS1zeXN0ZW0=
  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklrazFWek53VDNwck5HeDVZVmQyYm5WcU5VcFJNbEpyUldOb2VuWkhaV3BDV0hGM1FtcFZObXBHU2tVaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUpyZFdKbExYTjVjM1JsYlNJc0ltdDFZbVZ5Ym1WMFpYTXVhVzh2YzJWeWRtbGpaV0ZqWTI5MWJuUXZjMlZqY21WMExtNWhiV1VpT2lKcmRXSmxMWEJ5YjNoNUxYUnZhMlZ1TFhRMk5HTmpJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5elpYSjJhV05sTFdGalkyOTFiblF1Ym1GdFpTSTZJbXQxWW1VdGNISnZlSGtpTENKcmRXSmxjbTVsZEdWekxtbHZMM05sY25acFkyVmhZMk52ZFc1MEwzTmxjblpwWTJVdFlXTmpiM1Z1ZEM1MWFXUWlPaUkwTVdSallqQTVZaTAwTkRjd0xUUTRNRFF0T1RJM055MHhaak0wTkRaa1lUVXhOREVpTENKemRXSWlPaUp6ZVhOMFpXMDZjMlZ5ZG1salpXRmpZMjkxYm5RNmEzVmlaUzF6ZVhOMFpXMDZhM1ZpWlMxd2NtOTRlU0o5LkowYWdMeGhrdWhJRzNqU0dvZm13TGZ1UjJrVk0ydEEtanRmaERid1packRMeW5lb215R1AwVXJBVjc0N1JHT2RCcFk3NTZ5eTJLWklyYS1QaG13Tmo5bUxKbzllVm9LYW9rNmdIMjg4WXZnTmktdWwwblVDa1MwbndITkpueF9fUWVEbGdIQ3RVbWRpeTBwZDFjSU0zSl9Oc1VvcDJvbk15bzNKcWtLQVlvRUJ3X3pmelhUWi1nazVvZ0lMRGtILXRUNExLdi1ZTnNWclpLOUNsbFhqdERnOEdEVEJ6UTE3TzhXY0E0WG1jSWdxdFM4T2xoaHlKeFNacGpuRk0yODBKWkJzVFdkNlgzd1lnbUZ4SDBMQ0VwYzdFdlJubklXVWlBZkF2MjlKOTU1aWs4MnBYY0pNZkpfZDdsbExFV0ZFMVRhWUFlTlRoVUY1MnRJM1h2U2lDZw==
```



--------------

```
echo 'ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklrazFWek53VDNwck5HeDVZVmQyYm5WcU5VcFJNbEpyUldOb2VuWkhaV3BDV0hGM1FtcFZObXBHU2tVaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUpyZFdKbExYTjVjM1JsYlNJc0ltdDFZbVZ5Ym1WMFpYTXVhVzh2YzJWeWRtbGpaV0ZqWTI5MWJuUXZjMlZqY21WMExtNWhiV1VpT2lKcmRXSmxMWEJ5YjNoNUxYUnZhMlZ1TFhRMk5HTmpJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5elpYSjJhV05sTFdGalkyOTFiblF1Ym1GdFpTSTZJbXQxWW1VdGNISnZlSGtpTENKcmRXSmxjbTVsZEdWekxtbHZMM05sY25acFkyVmhZMk52ZFc1MEwzTmxjblpwWTJVdFlXTmpiM1Z1ZEM1MWFXUWlPaUkwTVdSallqQTVZaTAwTkRjd0xUUTRNRFF0T1RJM055MHhaak0wTkRaa1lUVXhOREVpTENKemRXSWlPaUp6ZVhOMFpXMDZjMlZ5ZG1salpXRmpZMjkxYm5RNmEzVmlaUzF6ZVhOMFpXMDZhM1ZpWlMxd2NtOTRlU0o5LkowYWdMeGhrdWhJRzNqU0dvZm13TGZ1UjJrVk0ydEEtanRmaERid1packRMeW5lb215R1AwVXJBVjc0N1JHT2RCcFk3NTZ5eTJLWklyYS1QaG13Tmo5bUxKbzllVm9LYW9rNmdIMjg4WXZnTmktdWwwblVDa1MwbndITkpueF9fUWVEbGdIQ3RVbWRpeTBwZDFjSU0zSl9Oc1VvcDJvbk15bzNKcWtLQVlvRUJ3X3pmelhUWi1nazVvZ0lMRGtILXRUNExLdi1ZTnNWclpLOUNsbFhqdERnOEdEVEJ6UTE3TzhXY0E0WG1jSWdxdFM4T2xoaHlKeFNacGpuRk0yODBKWkJzVFdkNlgzd1lnbUZ4SDBMQ0VwYzdFdlJubklXVWlBZkF2MjlKOTU1aWs4MnBYY0pNZkpfZDdsbExFV0ZFMVRhWUFlTlRoVUY1MnRJM1h2U2lDZw==' | base64 -d
```



```
eyJhbGciOiJSUzI1NiIsImtpZCI6Ikk1VzNwT3prNGx5YVd2bnVqNUpRMlJrRWNoenZHZWpCWHF3QmpVNmpGSkUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlLXByb3h5LXRva2VuLXQ2NGNjIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6Imt1YmUtcHJveHkiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI0MWRjYjA5Yi00NDcwLTQ4MDQtOTI3Ny0xZjM0NDZkYTUxNDEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06a3ViZS1wcm94eSJ9.J0agLxhkuhIG3jSGofmwLfuR2kVM2tA-jtfhDbwZZrDLyneomyGP0UrAV747RGOdBpY756yy2KZIra-PhmwNj9mLJo9eVoKaok6gH288YvgNi-ul0nUCkS0nwHNJnx__QeDlgHCtUmdiy0pd1cIM3J_NsUop2onMyo3JqkKAYoEBw_zfzXTZ-gk5ogILDkH-tT4LKv-YNsVrZK9CllXjtDg8GDTBzQ17O8WcA4XmcIgqtS8OlhhyJxSZpjnFM280JZBsTWd6X3wYgmFxH0LCEpc7EvRnnIWUiAfAv29J955ik82pXcJMfJ_d7llLEWFE1TaYAeNThUF52tI3XvSiCg
```



-----------------------



# 获取可执行文件



192.168.3.101



当前系统的版本是 1.4.4 

```
conntrack --version
```



```
conntrack v1.4.4 (conntrack-tools)
```



kube-proxy 容器内的版本  是1.4.5 

```
docker exec fc1c607  conntrack --version
```



```
conntrack v1.4.5 (conntrack-tools)
```

----------------



```
docker run -it --name kube-proxy registry.aliyuncs.com/google_containers/kube-proxy:v1.18.3  /bin/sh
```



```
mkdir  /opt/kube-proxy
```



```
docker cp kube-proxy:/usr/sbin/conntrack    /opt/kube-proxy
docker cp kube-proxy:/usr/local/bin/kube-proxy  /opt/kube-proxy
```



```
rm -rf /usr/sbin/conntrack
```



```
cp  /opt/kube-proxy/conntrack  /usr/sbin/
```



```
cp /opt/kube-proxy/kube-proxy  /usr/local/bin/
```



```
docker stop kube-proxy
docker rm kube-proxy
```







#  配置kube-proxy  config.conf



```
mkdir /var/lib/kube-proxy/
```



```
cat >  /var/lib/kube-proxy/config.conf  << EOF
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
clientConnection:
  acceptContentTypes: ""
  burst: 0
  contentType: ""
  kubeconfig: /var/lib/kube-proxy/kubeconfig.conf
  qps: 0
clusterCIDR: 10.244.0.0/16
configSyncPeriod: 0s
conntrack:
  maxPerCore: null
  min: null
  tcpCloseWaitTimeout: null
  tcpEstablishedTimeout: null
detectLocalMode: ""
enableProfiling: false
healthzBindAddress: ""
hostnameOverride: ""
iptables:
  masqueradeAll: false
  masqueradeBit: null
  minSyncPeriod: 0s
  syncPeriod: 0s
ipvs:
  excludeCIDRs: null
  minSyncPeriod: 0s
  scheduler: ""
  strictARP: false
  syncPeriod: 0s
  tcpFinTimeout: 0s
  tcpTimeout: 0s
  udpTimeout: 0s
kind: KubeProxyConfiguration
metricsBindAddress: ""
mode: ipvs
nodePortAddresses: null
oomScoreAdj: null
portRange: ""
showHiddenMetricsForVersion: ""
udpIdleTimeout: 0s
winkernel:
  enableDSR: false
  networkName: ""
  sourceVip: ""
EOF
```



-------------------------







# 创建 kube-proxy 使用的 kubeconfig



###  创建客户端证书  

CN 为  system:kube-proxy



```
mkdir /cert/k8s -p
cd /cert/k8s
```



```
openssl genrsa -out kube-proxy.key 2048
openssl req -new -key kube-proxy.key  -subj "/CN=system:kube-proxy" -out kube-proxy.csr
```



```
cat > kube-proxy-csr.conf  << EOF
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
openssl x509 -req -in kube-proxy.csr  \
-CA  /etc/kubernetes/pki/ca.crt  -CAkey /etc/kubernetes/pki/ca.key  \
-CAcreateserial -out kube-proxy.crt -days 10000 \
-extensions v3_ext -extfile kube-proxy-csr.conf
```

-------------



###  kubeconfig.conf



```
kubectl config set-cluster default \
      --certificate-authority=/etc/kubernetes/pki/ca.crt \
      --embed-certs=true \
      --server=https://192.168.3.101:6443 \
      --kubeconfig=/var/lib/kube-proxy/kubeconfig.conf
```



```
kubectl config set-credentials  default  \
  --client-certificate=/cert/k8s/kube-proxy.crt \
  --client-key=/cert/k8s/kube-proxy.key \
  --embed-certs=true \
  --kubeconfig=/var/lib/kube-proxy/kubeconfig.conf
```



```
kubectl config set-context  default  \
  --cluster=default \
  --user=default \
  --kubeconfig=/var/lib/kube-proxy/kubeconfig.conf
```



```
kubectl config use-context default   --kubeconfig=/var/lib/kube-proxy/kubeconfig.conf
```



# 启动测试 kube-proxy



```
kubectl  -n kube-system  delete ds kube-proxy
```



```
yum -y install screen
```



```
screen  -S kube-proxy
```



```
/usr/local/bin/kube-proxy --config=/var/lib/kube-proxy/config.conf --hostname-override=192.168.3.101
```



```
ctrl +  a  +d 
```







# config another node

192.168.3.103

```
scp 192.168.3.101:/usr/local/bin/kube-proxy  /usr/local/bin/
```

```
scp 192.168.3.101:/usr/sbin/conntrack  /usr/sbin/
```



```
mkdir /var/lib/kube-proxy/
```



```
scp 192.168.3.101:/var/lib/kube-proxy/config.conf  /var/lib/kube-proxy/
```

```
scp 192.168.3.101:/var/lib/kube-proxy/kubeconfig.conf  /var/lib/kube-proxy/
```



```
yum -y install screen
```



```
screen  -S kube-proxy
```



```
/usr/local/bin/kube-proxy --config=/var/lib/kube-proxy/config.conf --hostname-override=192.168.3.103
```



```
ctrl +  a  +d 
```



# systemd



example 

192.168.3.103



```
cat  > /usr/lib/systemd/system/kube-proxy.service  << EOF
[Unit]
Description=Kubernetes Kube-Proxy Server
After=network.target
Documentation=https://kubernetes.io/docs/

[Service]
ExecStart=/usr/local/bin/kube-proxy --config=/var/lib/kube-proxy/config.conf --hostname-override=192.168.3.103

Restart=always
StartLimitInterval=0
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF
```



```
systemctl  start  kube-proxy
systemctl  enable   kube-proxy
```



# test



```
ethtool -K  flannel.1 rx off tx off tso off gso off
```



```
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
EOF
```



```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
 name: nginx-service
spec:
 type: NodePort
 clusterIP: 10.96.0.100
 ports:
 - port: 8080
   targetPort: 80
   nodePort: 30001
 selector:
  app: nginx
EOF
```





```
time curl -m 10 -s -o /dev/null 10.96.0.100:8080
```



```
time curl -m 10 -s -o /dev/null 192.168.3.103:30001
```

```
time curl -m 10 -s -o /dev/null 192.168.3.101:30001
```


