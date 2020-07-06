

# 查看信息



### secret name:     bootstrap-token-usvieq

```
kubectl  -n kube-system  get secrets  bootstrap-token-usvieq  -o yaml
```

```
apiVersion: v1
data:
  auth-extra-groups: c3lzdGVtOmJvb3RzdHJhcHBlcnM6a3ViZWFkbTpkZWZhdWx0LW5vZGUtdG9rZW4=
  expiration: MjAyMC0wNi0yNlQxNDo1NzozMyswODowMA==
  token-id: dXN2aWVx
  token-secret: MnpnODYyMjhlbXltZ2NkNA==
  usage-bootstrap-authentication: dHJ1ZQ==
  usage-bootstrap-signing: dHJ1ZQ==
kind: Secret
metadata:
  creationTimestamp: "2020-06-25T06:57:33Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:auth-extra-groups: {}
        f:expiration: {}
        f:token-id: {}
        f:token-secret: {}
        f:usage-bootstrap-authentication: {}
        f:usage-bootstrap-signing: {}
      f:type: {}
    manager: kubeadm
    operation: Update
    time: "2020-06-25T06:57:33Z"
  name: bootstrap-token-usvieq
  namespace: kube-system
  resourceVersion: "901038"
  selfLink: /api/v1/namespaces/kube-system/secrets/bootstrap-token-usvieq
  uid: 01cb5c4e-4e68-4bfd-bca2-eefe04c54f1a
type: bootstrap.kubernetes.io/token
```



```
type: bootstrap.kubernetes.io/token
```



单独查看data 字段下的信息

```
data:
  auth-extra-groups: c3lzdGVtOmJvb3RzdHJhcHBlcnM6a3ViZWFkbTpkZWZhdWx0LW5vZGUtdG9rZW4=
  expiration: MjAyMC0wNi0yNlQxNDo1NzozMyswODowMA==
  token-id: dXN2aWVx
  token-secret: MnpnODYyMjhlbXltZ2NkNA==
  usage-bootstrap-authentication: dHJ1ZQ==
  usage-bootstrap-signing: dHJ1ZQ==
```



###  auth-extra-groups

```
echo 'c3lzdGVtOmJvb3RzdHJhcHBlcnM6a3ViZWFkbTpkZWZhdWx0LW5vZGUtdG9rZW4=' | base64 -d
system:bootstrappers:kubeadm:default-node-token
```

### expiration

```
echo 'MjAyMC0wNi0yNlQxNDo1NzozMyswODowMA==' | base64 -d
2020-07-04T10:03:11+08:00
```



### token-id

```
echo 'dXN2aWVx' | base64 -d
usvieq
```



### token-secret

```
echo 'MnpnODYyMjhlbXltZ2NkNA==' | base64 -d
2zg86228emymgcd4
```



### usage-bootstrap-authentication

```
echo 'dHJ1ZQ==' | base64 -d
true
```



### usage-bootstrap-signing

```
echo 'dHJ1ZQ==' | base64 -d
true
```







# tls





> https://v1-18.docs.kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/#authorize-kubelet-to-create-csr









# 从 token 开始分析

token   usvieq.2zg86228emymgcd4      存放于 secret 下  secret 的名字是 bootstrap-token-usvieq 

secret 的 所属的namespace 是  kube-system



```
token usvieq.2zg86228emymgcd4 属于组system:bootstrappers:kubeadm:default-node-token
```





RBAC 基于角色的访问控制     



下面就以这个组名 system:bootstrappers:kubeadm:default-node-token  为切入点，观察这个group 被赋予的哪些权限





过滤出 同组 system:bootstrappers:kubeadm:default-node-token 相关的 clusterrolebinding



-------------------

```
kubectl get clusterrolebinding -o custom-columns='Name:metadata.name,Subjects:subjects'  | grep 'system:bootstrappers:kubeadm:default-node-token'
```



```
kubeadm:get-nodes                                      [map[apiGroup:rbac.authorization.k8s.io kind:Group name:system:bootstrappers:kubeadm:default-node-token]]
kubeadm:kubelet-bootstrap                              [map[apiGroup:rbac.authorization.k8s.io kind:Group name:system:bootstrappers:kubeadm:default-node-token]]
kubeadm:node-autoapprove-bootstrap                     [map[apiGroup:rbac.authorization.k8s.io kind:Group name:system:bootstrappers:kubeadm:default-node-token]]
```



------------------



| clusterrolebinding                 | clusterrole                                                  | group                                           |      |
| ---------------------------------- | ------------------------------------------------------------ | ----------------------------------------------- | ---- |
| kubeadm:get-nodes                  | kubeadm:get-nodes                                            | system:bootstrappers:kubeadm:default-node-token |      |
| kubeadm:kubelet-bootstrap          | system:node-bootstrapper                                     | system:bootstrappers:kubeadm:default-node-token |      |
| kubeadm:node-autoapprove-bootstrap | system:certificates.k8s.io:certificatesigningrequests:nodeclient | system:bootstrappers:kubeadm:default-node-token |      |



------------

```
yum -y install jq
```



```
kubectl    get clusterrole kubeadm:get-nodes -o json | jq .rules
```



```
[
  {
    "apiGroups": [
      ""
    ],
    "resources": [
      "nodes"
    ],
    "verbs": [
      "get"
    ]
  }
]
```





---------

```
kubectl    get clusterrole system:node-bootstrapper -o json | jq .rules
```



```
[
  {
    "apiGroups": [
      "certificates.k8s.io"
    ],
    "resources": [
      "certificatesigningrequests"
    ],
    "verbs": [
      "create",
      "get",
      "list",
      "watch"
    ]
  }
]
```

------------



```
kubectl   get clusterrole system:certificates.k8s.io:certificatesigningrequests:nodeclient -o json | jq .rules
```



```
[
  {
    "apiGroups": [
      "certificates.k8s.io"
    ],
    "resources": [
      "certificatesigningrequests/nodeclient"
    ],
    "verbs": [
      "create"
    ]
  }
]
```



---------------





过滤出 同组 system:bootstrappers:kubeadm:default-node-token 相关的 rolebinding



```
kubectl get rolebinding -o custom-columns='Name:metadata.name,Subjects:subjects'  -A | grep 'system:bootstrappers:kubeadm:default-node-token'	
```



```
kube-proxy                                          [map[apiGroup:rbac.authorization.k8s.io kind:Group name:system:bootstrappers:kubeadm:default-node-token]]
kubeadm:kubelet-config-1.18                         [map[apiGroup:rbac.authorization.k8s.io kind:Group name:system:nodes] map[apiGroup:rbac.authorization.k8s.io kind:Group name:system:bootstrappers:kubeadm:default-node-token]]
kubeadm:nodes-kubeadm-config                        [map[apiGroup:rbac.authorization.k8s.io kind:Group name:system:bootstrappers:kubeadm:default-node-token] map[apiGroup:rbac.authorization.k8s.io kind:Group name:system:nodes]]
```



| rolebinding                  | role                         | group                                                        |
| ---------------------------- | ---------------------------- | ------------------------------------------------------------ |
| kube-proxy                   | kube-proxy                   | system:bootstrappers:kubeadm:default-node-token              |
| kubeadm:kubelet-config-1.18  | kubeadm:kubelet-config-1.18  | system:bootstrappers:kubeadm:default-node-token\|system:nodes |
| kubeadm:nodes-kubeadm-config | kubeadm:nodes-kubeadm-config | system:bootstrappers:kubeadm:default-node-token\|system:nodes |



-------

```
kubectl  get  -n kube-system  role kube-proxy  -o yaml
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



```
kubectl  get -n kube-system  configmaps  kube-proxy   -o yaml
```



------------

```
kubectl  get  -n kube-system  role kubeadm:kubelet-config-1.18   -o yaml
```



```
rules:
- apiGroups:
  - ""
  resourceNames:
  - kubelet-config-1.18
  resources:
  - configmaps
  verbs:
  - get
```

---------------



```
kubectl  get  -n kube-system  role kubeadm:nodes-kubeadm-config   -o yaml
```



```
rules:
- apiGroups:
  - ""
  resourceNames:
  - kubeadm-config
  resources:
  - configmaps
  verbs:
  - get
```









