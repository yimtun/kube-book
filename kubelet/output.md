

```
[bootstrap-token] Using token: usvieq.2zg86228emymgcd4
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
```



# 



# bootstrap-token output1

```
Using token: usvieq.2zg86228emymgcd4
```



如果没有额外指定kubeadm 安装时会自动生成这个token 



# bootstrap-token output2

```
Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
```



# bootstrap-token  output3



```
configured RBAC rules to allow Node Bootstrap tokens to get nodes
```

配置 Bootstrap tokens  具有 get  nodes 的权限

# bootstrap-token  output4

```
configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
```



配置 Bootstrap tokens   具有 发送csr 的权限    以便获取证书



# bootstrap-token  output5

```
configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
```



配置RBAC规则以允许csr approver控制器从节点引导令牌自动批准CSR





# bootstrap-token  output6

```
configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
```



已配置RBAC规则以允许对群集中的所有节点客户端证书进行证书轮换



# bootstrap-token  output7



```
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
```



创建一个configmap     名字是 cluster-info   所在的namespace 名字是 kube-public





```
kubectl  -n kube-public  get configmaps  cluster-info -o yaml
```



```
apiVersion: v1
data:
  jws-kubeconfig-usvieq: eyJhbGciOiJIUzI1NiIsImtpZCI6InVzdmllcSJ9..jQjvFwp_Cjn2dbP0dLUkFD9WihQIvGqDJDvFwCL-QPg
  kubeconfig: |
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01EWXlPREExTkRZeE1Gb1hEVE13TURZeU5qQTFORFl4TUZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTG5jClVIZmFiQzljcFk4MXNKdmwvTUlnU1ZWNHRMUVNoVjI0U280MWNuVy9HMWFWTUtYNk5kR0wwWDl2YTBsS1I4SFMKV25hcVo4K3BnN3hmYVgvVVBMbzU4dlptVDhvZC9DZ0pQaG9iczA5SFc0TklNNmlGOFVkMkVUbkNiUDJkTkNMRApZd29FYUgwTEdhVHE4ejh1ZzE5Y1pXUHI1UGtQdkZxV3JneGQwUWxyRzhyd2R2SkpLVWlFL3hqOS81VkNNTG9xCmVndVo5YjlXWmZEMktMTlhFamFEb25pbzRhMmdxaVBFdEJpaFVhOFdZWU5URHptWVFtRVI3VjRORkx3bW5KV3YKbjNRUGs2aXBOOEI1VmFlbCtaZ1B0bmQwc3VmajBKSG1rVk1Ra2pVdnpvZjRKUEIvVUFXUTRrLzNIaWhLRE9VVQpSeXYvREU5M0h5ZCtuVW9iTG1FQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFJdXlzU3NmM1E1OW9SZ0NyakQvQmV5N0V1KzYKZ3kxZ0FQNzI3OGxBRUV3K0pTclhrenN6MUk4QXBwbmJxb0tKbjlPNmw1UGxLRlA1NWFkS3JZT2Q0TWJKcWtXcQpIT1h3YUVOWVdESldHTjhONVJydXJMMXA5ckQ2K0NPa3ZMSWNMb21vTW0ya2JpRldJMFJ0MVdMaitiK2ZWK2d0ClpqT1orVFE1TmFmeUVRM3BTVUROWFNKYlQ0Y2lxSXR2Z05aNzVvTmYwWmozNDBVeE9jSVhtYityODc2ZDlJMFAKNHNQTkRDVGJSeUY1YXd1VWhXRmtBOXhCREdFS1hOTHVkV29sREowOVJWTU8xQ1A5eStheDJiQnk5TTlNWWtNTQpQa05NanFhYzlwcVB0dVVJUlJkdkZ4QlZZVVZJQnVxV2JTdnNwcWw0L2F0OXN1c1NxV2J0QU5nVGlFST0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
        server: https://192.168.3.101:6443
      name: ""
    contexts: null
    current-context: ""
    kind: Config
    preferences: {}
    users: null
kind: ConfigMap
metadata:
  creationTimestamp: "2020-06-28T05:46:34Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:kubeconfig: {}
    manager: kubeadm
    operation: Update
    time: "2020-06-28T05:46:34Z"
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        f:jws-kubeconfig-usvieq: {}
    manager: kube-controller-manager
    operation: Update
    time: "2020-06-28T05:46:51Z"
  name: cluster-info
  namespace: kube-public
  resourceVersion: "1396711"
  selfLink: /api/v1/namespaces/kube-public/configmaps/cluster-info
  uid: 46990d5f-ef2a-463d-84b7-9e04327ae092
```







----------

```
kubeadm join 192.168.3.101:6443 --token usvieq.2zg86228emymgcd4 \
    --discovery-token-ca-cert-hash sha256:f15e6eefb83447b09aed5e0d34b061ba2240426aec1649366153add3ca46a787 
```



稍后 作为node 节点加入集群的命令  sha256的值 取自己的环境输出所得 

稍后不直接使用这个命令 而是添加一个参数 --node-name 

