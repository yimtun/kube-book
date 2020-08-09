[TOC]



# 添加node测试1


 将代表 bootstrap token 的用户 绑定到名字为 system:node-bootstrapper 的clusterrole





此环境下的user  就是   system:bootstrap:398c7c



192.168.3.101



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
kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --user=system:bootstrap:398c7c
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
kubectl  certificate approve  csr-xxx
```





192.168.3.104



停掉kubelet  删除证书 和   kubeconfig 文件

```
rm -rf /var/lib/kubelet/pki/*
rm -rf /etc/kubernetes/kubelet.conf
```



再次启动kubelet



```
/usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --cgroup-driver=systemd --hostname-override=192.168.3.104 --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2  --kubelet-cgroups=/systemd/system.slice
```





# 添加node测试2





 将代表 bootstrap token 的组  绑定到名字为 system:node-bootstrapper 的clusterrole



这里组名默认就是  system:bootstrappers



192.168.3.101



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


```
kubectl  certificate approve  csr-xxx
```

------------------------



192.168.3.104



停掉kubelet  删除证书 和   kubeconfig 文件

```
rm -rf /var/lib/kubelet/pki/*
rm -rf /etc/kubernetes/kubelet.conf
rm -rf /etc/cni/net.d/10-flannel.conflist
```

```
reboot
```



再次启动kubelet



```
/usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --cgroup-driver=systemd --hostname-override=192.168.3.104 --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2  --kubelet-cgroups=/systemd/system.slice
```

---------------------





# 添加node测试3



 将  bootstrap token 的扩展组   绑定到名字为 system:node-bootstrapper 的clusterrole



绑定到扩展组   system:bootstrappers:yandun-test





192.168.3.101

创建 type 为 bootstrap.kubernetes.io/token 的secret  并设置扩展组名为

system:bootstrappers:yandun-test





```
cat  >  bootstrap-token-398c7c.yaml << EOF
apiVersion: v1
kind: Secret
metadata:
  name: bootstrap-token-398c7c
  namespace: kube-system
type: bootstrap.kubernetes.io/token
stringData:
  token-id: '398c7c'
  token-secret: b18a35f794625922
  usage-bootstrap-authentication: "true"
  usage-bootstrap-signing: "true"
  auth-extra-groups: system:bootstrappers:yandun-test
EOF
```





```
kubectl  -n kube-system  delete secrets  bootstrap-token-398c7c
kubectl  create  -f bootstrap-token-398c7c.yaml
```





创建clusterrolebinding

```
kubectl delete  clusterrolebinding  kubelet-bootstrap
```



```
kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --group=system:bootstrappers:yandun-test
```





清除环境 重新添加node



192.168.3.101

```
kubectl drain 192.168.3.104 --ignore-daemonsets
kubectl delete node 192.168.3.104
```



```
kubectl get csr  -w
```



```
kubectl get node -w
```



```
kubectl  certificate approve  csr-xxxx  
```



192.168.3.104



停掉kubelet  删除证书 和   kubeconfig 文件

```
rm -rf /var/lib/kubelet/pki/*
rm -rf /etc/kubernetes/kubelet.conf
rm -rf /etc/cni/net.d/10-flannel.conflist
```



```
reboot
```



再次启动kubelet



```
/usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --cgroup-driver=systemd --hostname-override=192.168.3.104 --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2  --kubelet-cgroups=/systemd/system.slice
```









# 13查看 kubeadm 添加 node 设置的kubelet配置 文件



```
cat /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf 
```



```
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/sysconfig/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
```

----------------



```
cat /var/lib/kubelet/kubeadm-flags.env
```



```
KUBELET_KUBEADM_ARGS="--cgroup-driver=systemd --hostname-override=192.168.3.103 --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2"
```



-------------------



```
cat /var/lib/kubelet/config.yaml
```



```
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
- 10.1.0.10
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
```

----------------





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

---------------



#  14 systemd 

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

```
mkdir /var/lib/kubelet/
```

```
cat  > /var/lib/kubelet/kubeadm-flags.env << EOF
KUBELET_KUBEADM_ARGS="--cgroup-driver=systemd --hostname-override=192.168.3.104 --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2"
EOF
```





```
cat 	/etc/sysconfig/kubelet
```

```
KUBELET_EXTRA_ARGS=
```



```
systemctl daemon-reload
```



```
systemctl  stop  kubelet
```





```
systemctl  start   kubelet
```






