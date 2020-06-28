



# flannel 以cni 插件的方式运行  是以 ds 方式运行的





## 权限相关



flannel 以cni 插件方式运行意味着flannel同 k8s 联系更加紧密，



```
kubectl  -n kube-system  get ds kube-flannel-ds-amd64  -o yaml
```



查看权限相关信息    flannel 需要调用 kube-api   在pod 内运行的程序 使用serviceAccount 作为调用api 的凭证，

这里直接查出 flannel  使用的  serviceAccount

-----------------

```
kubectl  -n kube-system  get ds kube-flannel-ds-amd64  -o json | jq .spec.template.spec.serviceAccount
```



```
"flannel"
```

---------------



根据serviceAccount  的名字 查找  哪个 clusterrolebinding 使用的这个 serviceAccount

```
kubectl get clusterrolebinding -o custom-columns='Name:metadata.name,Subject:subjects' | grep 'flannel'
```



```
flannel                                                [map[kind:ServiceAccount name:flannel namespace:kube-system]]
```

-----------



flannel 相关的 rolebinding  无



```
kubectl get rolebinding -o custom-columns='Name:metadata.name,Subject:subjects'  -n kube-system | grep 'flannel'
```



----------------

查看 名字为 flannel  的 clusterrolebinding  的信息 把它绑定的 clusterrole 也找出来


```
kubectl get clusterrolebinding  flannel -o custom-columns='Name:metadata.name,Role:roleRef,Subject:subjects'
```



```
Name      Role                                                                    Subject
flannel   map[apiGroup:rbac.authorization.k8s.io kind:ClusterRole name:flannel]   [map[kind:ServiceAccount name:flannel namespace:kube-system]]
```

------------



| clusterrolebinding | ClusterRole | subjects | subjects kind  |
| ------------------ | ----------- | -------- | -------------- |
| flannel            | flannel     | flannel  | ServiceAccount |
|                    |             |          |                |



名字为   flannel 的 clusterrolebinding  详细信息

```
kubectl  get clusterrolebinding flannel  -o yaml
```



-----------------

查看名字为 flannel 的 clusterrole 的 权限信息


```
kubectl  get clusterrole  flannel  -o yaml
```



```
rules:
- apiGroups:
  - extensions
  resourceNames:
  - psp.flannel.unprivileged
  resources:
  - podsecuritypolicies
  verbs:
  - use
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes/status
  verbs:
  - patch
```

------------



# 查看运行参数



```
ps -ef | grep flanneld | grep -v 'grep'
```



```
/opt/bin/flanneld --ip-masq --kube-subnet-mgr
```



#  查看flanneld 的配置文件

```
docker ps  --format "table {{.ID}}\t{{.Command}}"  --no-trunc | grep flannel
```



```
75ac96b686b2067302ef0fb7972ed378f9a04e098360f7453f894cb2df887789   "/opt/bin/flanneld --ip-masq --kube-subnet-mgr"
```



--------------



```
docker exec -it 75ac96b68 /bin/bash
```







```
cat /etc/kube-flannel/cni-conf.json 
{
  "name": "cbr0",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "flannel",
      "delegate": {
        "hairpinMode": true,
        "isDefaultGateway": true
      }
    },
    {
      "type": "portmap",
      "capabilities": {
        "portMappings": true
      }
    }
  ]
}
```



```
cat /etc/kube-flannel/net-conf.json 
{
  "Network": "10.244.0.0/16",
  "Backend": {
    "Type": "vxlan"
  }
}
```





 flanneld 启动时自动将网络配置信息写入subnet.env 

```
cat /run/flannel/subnet.env 
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.0.1/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true
```



