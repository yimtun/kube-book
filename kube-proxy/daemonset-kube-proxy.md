

# 查看当前配置信息

192.168.3.101



###  daemonset 的方式运行kube-proxy







查看pod  kube-proxy  使用的sa

```
kubectl  -n kube-system  get ds kube-proxy -o json | jq .spec.template.spec.serviceAccount
```



```
"kube-proxy"
```



查看sa  kube-proxy   管理的  clusterrolebinding

```
kubectl get clusterrolebinding -o custom-columns='Name:metadata.name,xx:subjects'  -n kube-system | grep 'kube-proxy'
```



```
kubeadm:node-proxier                                   [map[kind:ServiceAccount name:kube-proxy namespace:kube-system]]
system:node-proxier                                    [map[apiGroup:rbac.authorization.k8s.io kind:User name:system:kube-proxy]]
```



| clusterrolebinding   | clusterrole         | subjects kind  | subjects name              |      |
| -------------------- | ------------------- | -------------- | -------------------------- | ---- |
| kubeadm:node-proxier | system:node-proxier | ServiceAccount | kube-proxy   ns:kube-proxy |      |
| system:node-proxier  | system:node-proxier | User           | system:kube-proxy          |      |
|                      |                     |                |                            |      |





查看sa  kube-proxy   关联的  rolebinding

```
kubectl get rolebinding -o custom-columns='Name:metadata.name,xx:subjects'  -n kube-system | grep 'kube-proxy'
```



```
kube-proxy                                          [map[apiGroup:rbac.authorization.k8s.io kind:Group name:system:bootstrappers:kubeadm:default-node-token]]
```





| rolebinding | rolebinding ns | role       | subjects kind | subjects name                                   |
| ----------- | -------------- | ---------- | ------------- | ----------------------------------------------- |
| kube-proxy  | kube-system    | kube-proxy | Group         | system:bootstrappers:kubeadm:default-node-token |
|             |                |            |               |                                                 |
|             |                |            |               |                                                 |





kube-proxy 正常运行需要的 权限是  名字为  system:node-proxier  的 clusterrole













```
docker ps  --format "table {{.ID}}\t{{.Command}}"  --no-trunc | grep kube-proxy
```



```
9be91024d915af1631d5fa831fd94c46286d930b8548ba2ab45db014aa983355   "/usr/local/bin/kube-proxy --config=/var/lib/kube-proxy/config.conf --hostname-override=192.168.3.101"
```





-------------------



```
docker exec 9be91024d cat /var/lib/kube-proxy/config.conf 
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



```
docker exec 9be91024d  conntrack --version
```



```
conntrack v1.4.5 (conntrack-tools)
```





#  配置kube-proxy



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

```
docker run -it --name kube-proxy registry.aliyuncs.com/google_containers/kube-proxy:v1.18.3  /bin/sh
```





```
docker cp kube-proxy:/usr/sbin/conntrack    /opt/kube-proxy
docker cp kube-proxy:/usr/local/bin/kube-proxy  /opt/kube-proxy
```

```
docker stop kube-proxy
docker rm kube-proxy
```



```
conntrack --version
conntrack v1.4.4 (conntrack-tools)
```

```
rm -rf /usr/sbin/conntrack
```



```
cp  /opt/kube-proxy/conntrack  /usr/sbin/
```



```
conntrack --version
conntrack v1.4.5 (conntrack-tools)
```



```
cp /opt/kube-proxy/kube-proxy  /usr/local/bin/
```



kubeconfig



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



# 启动 kube-proxy



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
cat > nginx-dp.yml <<EOF
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - name: http
    port: 80
    targetPort: 80
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
EOF
```



```
kubectl run nginx \
      --image=nginx:latest \
      --port=80
```





> https://kubernetes.io/zh/docs/reference/kubectl/cheatsheet/

```
kubectl create deployment nginx --image=nginx
```

