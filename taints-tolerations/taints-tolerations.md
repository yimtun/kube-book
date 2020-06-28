> https://v1-17.docs.kubernetes.io/zh/docs/concepts/configuration/taint-and-toleration/



> https://v1-18.docs.kubernetes.io/zh/docs/concepts/configuration/taint-and-toleration/







#  taint and toleration

```
taint      [teɪnt]           污点
toleration [ˌtɑːləˈreɪʃn]     容忍

taint       代表污点    作用在  node  上；
toleration  代表容忍度   作用在  pod    上，是否能够容忍 node 上的 taint；
```



```
Taint 使节点node能够排斥一类特定的 pod。

Taint 和 toleration 相互配合，可以用来避免 pod 被分配到不合适的节点上。每个节点上都可以应用一个或多个 taint ，这表示对于那些不能容忍这些 taint 的 pod，是不会被该节点接受的。如果将 toleration 应用于 pod 上，则表示这些 pod 可以（但不要求）被调度到具有匹配 taint 的节点上。
```





1 场景  专机专用



2 或者隔离生产和测试环境





#  查看一个node上的所有taint

效率高 

```
kubectl  get  node 192.168.3.101  --output=jsonpath={.spec.taints}
```



比较易读 

```
yum -y install jq
```



```
kubectl  get  node 192.168.3.101 -o json | jq .spec.taints
```



# 删除一个node 上的所有taint



```
kubectl patch node 192.168.3.101 -p '{"spec":{"taints":[]}}'
```





# 为node 添加taint



为node 添加taint



```
kubectl  explain  nodes.spec.taints
```



####  语法

```
kubectl taint node [node] key=value:[effect]
```

value 可以省略   



effect 类型  

```
NoSchedule  PreferNoSchedule  NoExecute
```



#### key 和  effect 定位一个taint



#### 为node添加一个taint 

```
kubectl taint node 192.168.3.101 key-1=value-1:NoSchedule
```



```
kubectl  get  node 192.168.3.101  --output=jsonpath={.spec.taints}
```



```
kubectl taint node 192.168.3.101 key-1=value-2:NoSchedule
```

```
error: node 192.168.3.101 already has key-1 taint(s) with same effect(s) and --overwrite is false
```



省略value 的写法  key:effect

```
kubectl patch node 192.168.3.101 -p '{"spec":{"taints":[]}}'
```



```
kubectl taint node 192.168.3.101 key-1:PreferNoSchedule
```



```
kubectl taint node 192.168.3.101 key-1=value-1:PreferNoSchedule
```

```
error: node 192.168.3.101 already has key-1 taint(s) with same effect(s) and --overwrite is false
```









####  一次为一个node 添加多个taint

```
kubectl patch node 192.168.3.101 -p '{"spec":{"taints":[]}}'
```



```
kubectl taint node 192.168.3.101 key-1=value-1:NoSchedule  key-1=value-1:NoExecute key-2=value-2:NoSchedule
```



```
kubectl  get  node 192.168.3.101  --output=jsonpath={.spec.taints}
```



```
kubectl  get  node 192.168.3.101 -o json | jq .spec.taints
```



```
kubectl patch node 192.168.3.101 -p '{"spec":{"taints":[]}}'
```



####  为所有node 添加多个taint



```
kubectl taint node --all key-1=value-1:NoSchedule  key-1=value-1:NoExecute key-2=value-2:NoSchedule
```

```
kubectl  get  node 192.168.3.101 -o json | jq .spec.taints
kubectl patch node 192.168.3.101 -p '{"spec":{"taints":[]}}'
```





####  通过标签指定node 添加taint



```
kubectl get nodes --show-labels
```



```
kubectl label nodes 192.168.3.101  my-label=test
```



```
kubectl taint node -l my-label=test  key-1=value-1:NoSchedule  key-1=value-1:NoExecute key-2=value-2:NoSchedule
```



```
kubectl  get  node 192.168.3.101 -o json | jq .spec.taints
kubectl patch node 192.168.3.101 -p '{"spec":{"taints":[]}}'
kubectl label nodes 192.168.3.101  my-label-
```



```
kubectl get nodes --show-labels
```



####  删除node  的taint



```
kubectl taint node [node] key:[effect]-
```



#### 精确删除某个node 上的一个 taint

添加taint

```
kubectl taint node 192.168.3.101 key-1=value-1:NoSchedule
```

删除taint

```
kubectl taint node 192.168.3.101 key-1=value-1:NoSchedule-
```



通过key 和 effect 即可定位到一个taint    所以删除时不必指定value

```
kubectl taint node 192.168.3.101 key-1=value-1:NoSchedule
kubectl taint node 192.168.3.101 key-1:NoSchedule-
kubectl  get  node 192.168.3.101  --output=jsonpath={.spec.taints}
```





####  删除指定拥有相同key 的所有taint



同一个key 最多也只能有三个tanit

```
kubectl taint node 192.168.3.101 key-1=value-1:NoSchedule key-1=value-1:PreferNoSchedule key-1=value-1:NoExecute
```



```
kubectl  get  node 192.168.3.101  --output=jsonpath={.spec.taints}
kubectl  get  node 192.168.3.101 -o json | jq .spec.taints
```



删除 key 名字为  key-1 的所有 taint

```
kubectl taint node 192.168.3.101 key-1-
```



```
kubectl  get  node 192.168.3.101  --output=jsonpath={.spec.taints}
```



# 为pod 添加toleration



#####  说明1



与其说为pod 添加toleration

不如说 为pod的toleration 匹配某个或某组node 上的taint

如果pod 中的toleration 没有匹配到任何一个node 的 taint ，那么这个toleration 

是没有意义的，不会对调度造成任何影响或干预



##### 说明2



如果pod 的toleration和 某个node 的taint 相匹配，仅仅表示该pod可以被调度到这个node上

实际的调度情况决定了 pod 具体是否在该node上运行或者在其他node上运行





pod 中的toleration 匹配了一个节点的所有taint,  那么这个pod 就可以被调度到这个节点上，但其实该

pod 也可以被调度到其他node上

toleration 不是将 pod 绑定到node  的方法， 这个是一种 node 排斥 pod 的一种方法



```
kubectl  explain  pod.spec.tolerations
```



# 查看所一个 pod  的所有toleration

```
kubectl  get pod myapp-pod  -o json | jq .spec.tolerations
```





####  查看pod 的默认tolerations



```
cat >  pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
EOF
```



```
kubectl  create  -f pod.yaml
```



```
kubectl  get pod myapp-pod  --output=jsonpath={.spec.tolerations}
```



----------------------------

```
kubectl  get pod myapp-pod  -o json | jq .spec.tolerations
```



```
[
  {
    "effect": "NoExecute",
    "key": "node.kubernetes.io/not-ready",
    "operator": "Exists",
    "tolerationSeconds": 300
  },
  {
    "effect": "NoExecute",
    "key": "node.kubernetes.io/unreachable",
    "operator": "Exists",
    "tolerationSeconds": 300
  }
]
```



```
kubectl  delete pod myapp-pod
```



# 一个 toleration 和一个 taint 匹配



一个pod 可以有多个toleration 一个node 可以有多个taint

只有pod 的toleration 可以匹配一个node 的所有taint

这个pod 才有调度到这个node 的可能，但是不一定会调度到这个node



toleration 有两种匹配taint的方式

operator 字段值为 Equal 

operator 字段值为  Exists 







示例    对于node 上的如下taint

```
kubectl patch node 192.168.3.101 -p '{"spec":{"taints":[]}}'
kubectl taint node 192.168.3.101 key-1=value-1:NoSchedule
```



------------------

```
kubectl  get  node 192.168.3.101 -o json | jq .spec.taints
```



```
[
  {
    "effect": "NoSchedule",
    "key": "key-1",
    "value": "value-1"
  }
]
```



-------------

第一种 

operator 字段值为 Equal      要匹配的taint 的 value 如果不为空     使用Equal 的情况下 需要填写value

 

```
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  tolerations:
  - key: "key-1"
    operator: "Equal"
    value: "value-1" 
    effect: "NoSchedule"
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
EOF
```



```
kubectl  delete pod myapp-pod
kubectl  create  -f pod.yaml
```



```
kubectl  get pod myapp-pod  -o json | jq .spec.tolerations
```



```
kubectl  get pod myapp-pod
```



第二种

operator 字段值为  Exists        value 必须为空   不写此字段即可



```
tolerations:
  - key: "key-1"
    operator: "Exists"
    value: 
    effect: "NoSchedule"
```



```
tolerations:
  - key: "key-1"
    operator: "Exists"
    effect: "NoSchedule"
```



```
value must be empty when `operator` is 'Exists'
```



```
kubectl patch node 192.168.3.101 -p '{"spec":{"taints":[]}}'
kubectl delete pod --all -n default
kubectl taint node 192.168.3.101 key-1=value-1:NoSchedule
```



```
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  tolerations:
  - key: "key-1"
    operator: "Exists"
    effect: "NoSchedule"
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
EOF
```



```
kubectl  create  -f pod.yaml
```



```
kubectl  get pod myapp-pod  -o json | jq .spec.tolerations
kubectl  get pod
```



```
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  tolerations:
  - key: "key-1"
    operator: "Exists"
    value:
    effect: "NoSchedule"
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
EOF
```



```
kubectl delete pod --all -n default
kubectl  create  -f pod.yaml
```

```
kubectl  get pod myapp-pod  -o json | jq .spec.tolerations
kubectl  get pod
```



# 一个toleration匹配多个taint



#### 匹配key 名相同的所有 taint



```
kubectl patch node 192.168.3.101 -p '{"spec":{"taints":[]}}'
```



```
kubectl taint node 192.168.3.101 key-1=value-1:NoSchedule  key-1=value-1:NoExecute key-1=value-2:PreferNoSchedule
```

```
kubectl  get  node 192.168.3.101 -o json | jq .spec.taints
```





```
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  tolerations:
  - key: "key-1"
    operator: "Exists"
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
EOF
```



```
kubectl delete pod --all -n default
kubectl  create  -f pod.yaml
kubectl get pod 
```





#### 匹配所有taint



```
kubectl patch node 192.168.3.101 -p '{"spec":{"taints":[]}}'
```



```
kubectl taint node 192.168.3.101 key-1=value-1:NoSchedule  key-1=value-1:NoExecute key-2=value-2:NoSchedule
```

```
kubectl  get  node 192.168.3.101 -o json | jq .spec.taints
```



```
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  tolerations:
  - operator: "Exists"
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
EOF
```





```
kubectl delete pod --all -n default
kubectl  create  -f pod.yaml
kubectl get pod 
```



```
kubectl  get pod myapp-pod  -o json | jq .spec.tolerations
```







-----------------



# effect

effect 有三个值分别为

NoSchedule   PreferNoSchedule   NoExecute



```
Kubernetes 处理多个 taint 和 toleration 的过程就像一个过滤器：从一个节点的所有 taint 开始遍历，过滤掉那些 pod 中存在与之相匹配的 toleration 的 taint。余下未被过滤的 taint 的 effect 值决定了 pod 是否会被分配到该节点，特别是以下情况：
```





pod 

t1  t2  t3  t4   t5                      



node 

 t1 t2 t7 t8





```
如果未被过滤的 taint 中存在一个或一个以上 effect 值为 NoSchedule 的 taint，并且没有 effect值为NoExecute 的 taint,则 Kubernetes 不会将 pod 分配到该节点。 已经在此节点运行的pod不受影响  ex1 


如果未被过滤的 taint 中不存在 effect 值为 NoSchedule 和 NoExecute的 taint，但是存在 effect 值为 PreferNoSchedule 的 taint，则 Kubernetes 尽量避免将pod分配到该节点。
ex3



如果未被过滤的 taint 中存在一个或一个以上 effect 值为 NoExecute 的 taint，则 Kubernetes 不会将 pod 分配到该节点（如果 pod 还未在节点上运行），或者将 pod 从该节点驱逐（如果 pod 已经在节点上运行）。 ex2
```





### ex1

规划

```
pod  有如下toleration 

key 的名字分别为

t1 t2 t3 t4


node 有如下taint

key 名字分别为
t3 t4 t5 t6

其中
toleration  t3 t4 分别和 taint t3 t4 匹配
```





```
kubectl  delete pod --all
kubectl patch node 192.168.3.101 -p '{"spec":{"taints":[]}}'
```



在为node 添加taint 之前 先运行一个pod

```
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: pre-pod
  labels:
    app: pre
spec:
  tolerations:
  - key: "t1"
    operator: "Equal"
    value: "t1-v" 
    effect: "NoSchedule"
  - key: "t2"
    operator: "Equal"
    value: "t2-v" 
    effect: "NoSchedule"
  - key: "t3"
    operator: "Equal"
    value: "t3-v" 
    effect: "NoSchedule"
  - key: "t4"
    operator: "Exists"
    effect: "NoExecute"
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
EOF
```





```
kubectl  create  -f pod.yaml
kubectl  get pod
```



```
kubectl taint node 192.168.3.101 t3=t3-v:NoSchedule t4:NoExecute t5=t5-v:NoSchedule t6:PreferNoSchedule
```



为node 添加taint 后再次创建一个pod



```
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: post-pod
  labels:
    app: post
spec:
  tolerations:
  - key: "t1"
    operator: "Equal"
    value: "t1-v" 
    effect: "NoSchedule"
  - key: "t2"
    operator: "Equal"
    value: "t2-v" 
    effect: "NoSchedule"
  - key: "t3"
    operator: "Equal"
    value: "t3-v" 
    effect: "NoSchedule"
  - key: "t4"
    operator: "Exists"
    effect: "NoExecute"
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
EOF
```



```
kubectl  create  -f pod.yaml
```



---------------

```
kubectl  get pod  post-pod  
```



```
NAME       READY   STATUS    RESTARTS   AGE
post-pod   0/1     Pending   0          16s
```



此示例同时也说明 

NoSchedule 优先级高于PreferNoSchedule

----------------



### ex2



```
kubectl delete pod --all
kubectl patch node 192.168.3.101 -p '{"spec":{"taints":[]}}'
```



在为node 添加taint 之前 先运行一个pod



```
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: pre-pod
  labels:
    app: pre
spec:
  tolerations:
  - key: "t1"
    operator: "Equal"
    value: "t1-v" 
    effect: "NoSchedule"
  - key: "t2"
    operator: "Equal"
    value: "t2-v" 
    effect: "NoSchedule"
  - key: "t3"
    operator: "Equal"
    value: "t3-v" 
    effect: "NoSchedule"
  - key: "t4"
    operator: "Exists"
    effect: "NoExecute"
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
EOF
```





```
kubectl  create  -f pod.yaml
```



```
kubectl  get pod  -w
```



```
kubectl taint node 192.168.3.101 t3=t3-v:NoSchedule t4:NoExecute t5=t5-v:NoSchedule t6:NoExecute
```



t5  t6 都是未被 匹配的taint 这里也可以说明  NoExecute 的优先级比  NoSchedule 的高



```
此场景下  已经存在的pod 会被删除  如果使用其他类型的workload  就是被驱逐 
```



```
kubectl  get pod pre-pod  -w
```







添加taint 之后运行一个pod



```
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: post-pod
  labels:
    app: post
spec:
  tolerations:
  - key: "t1"
    operator: "Equal"
    value: "t1-v" 
    effect: "NoSchedule"
  - key: "t2"
    operator: "Equal"
    value: "t2-v" 
    effect: "NoSchedule"
  - key: "t3"
    operator: "Equal"
    value: "t3-v" 
    effect: "NoSchedule"
  - key: "t4"
    operator: "Exists"
    effect: "NoExecute"
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
EOF
```



```
kubectl  create  -f pod.yaml
kuebctl get pod post-pod -w
```





```
kubectl  get pod post-pod  -w
```







### ex3

```
kubectl delete pod --all
kubectl patch node 192.168.3.101 -p '{"spec":{"taints":[]}}'
```



```
在为node 添加taint 之前 先运行一个pod
```



```
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: pre-pod
  labels:
    app: pre
spec:
  tolerations:
  - key: "t1"
    operator: "Equal"
    value: "t1-v" 
    effect: "NoSchedule"
  - key: "t2"
    operator: "Equal"
    value: "t2-v" 
    effect: "NoSchedule"
  - key: "t3"
    operator: "Equal"
    value: "t3-v" 
    effect: "NoSchedule"
  - key: "t4"
    operator: "Exists"
    effect: "NoExecute"
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
EOF
```



```
kubectl  create  -f pod.yaml
```



```
kubectl  get pod pre-pod
```



```
kubectl taint node 192.168.3.101 t3=t3-v:NoSchedule t4:NoExecute t5=t5-v:PreferNoSchedule
```





```
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: post-pod
  labels:
    app: post
spec:
  tolerations:
  - key: "t1"
    operator: "Equal"
    value: "t1-v" 
    effect: "NoSchedule"
  - key: "t2"
    operator: "Equal"
    value: "t2-v" 
    effect: "NoSchedule"
  - key: "t3"
    operator: "Equal"
    value: "t3-v" 
    effect: "NoSchedule"
  - key: "t4"
    operator: "Exists"
    effect: "NoExecute"
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
EOF
```



```
kubectl  create   -f pod.yaml
```

```
kubectl get pod post-pod
```





### ex4 



测试下 PreferNoSchedule  和 NoExecute 的优先级

```
kubectl delete pod --all
kubectl patch node 192.168.3.101 -p '{"spec":{"taints":[]}}'
```



```
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: pre-pod
  labels:
    app: pre
spec:
  tolerations:
  - key: "t1"
    operator: "Equal"
    value: "t1-v" 
    effect: "NoSchedule"
  - key: "t2"
    operator: "Equal"
    value: "t2-v" 
    effect: "NoSchedule"
  - key: "t3"
    operator: "Equal"
    value: "t3-v" 
    effect: "NoSchedule"
  - key: "t4"
    operator: "Exists"
    effect: "NoExecute"
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
EOF
```



```
kubectl  create   -f pod.yaml
```



```
kubectl get pod pre-pod -w
```





```
kubectl taint node 192.168.3.101 t3=t3-v:NoSchedule t4:NoExecute t5=t5-v:PreferNoSchedule t6:NoExecute
```





# 特殊情况

 

```
kubectl  explain  pod.spec.tolerations.tolerationSeconds
```





effect 值为 `NoExecute` 的taint，它会影响已经在节点上运行的 pod

 

```
如果 pod 不能忍受effect 值为 NoExecute 的 taint，那么 pod 将马上被驱逐  ex2
```



```
如果 pod 能够忍受effect 值为 NoExecute 的 taint，但是在 toleration 定义中没有指定 tolerationSeconds，则 pod 还会一直在这个节点上运行。   ex1
```













```
如果 pod 能够忍受effect 值为 NoExecute 的 taint，而且指定了 tolerationSeconds，则 pod 还能在这个节点上继续运行这个指定的时间长度。 ex5
```





### ex5

```
kubectl delete pod --all
kubectl patch node 192.168.3.101 -p '{"spec":{"taints":[]}}'
```



在为node 添加taint 之前 先创建一个pod

```
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: pre-pod
  labels:
    app: pre
spec:
  tolerations:
  - key: "t1"
    operator: "Equal"
    value: "t1-v" 
    effect: "NoSchedule"
  - key: "t2"
    operator: "Equal"
    value: "t2-v" 
    effect: "NoSchedule"
  - key: "t3"
    operator: "Equal"
    value: "t3-v" 
    effect: "NoSchedule"
  - key: "t4"
    operator: "Exists"
    effect: "NoExecute"
    tolerationSeconds: 10
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
EOF
```



```
kubectl  create  -f pod.yaml
```

```
kubectl  get pod pre-pod  -w
```



```
kubectl taint node 192.168.3.101 t3=t3-v:NoSchedule t4:NoExecute
```



在添加taint 后 30秒 会被删除







```
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: post-pod
  labels:
    app: post
spec:
  tolerations:
  - key: "t1"
    operator: "Equal"
    value: "t1-v" 
    effect: "NoSchedule"
  - key: "t2"
    operator: "Equal"
    value: "t2-v" 
    effect: "NoSchedule"
  - key: "t3"
    operator: "Equal"
    value: "t3-v" 
    effect: "NoSchedule"
  - key: "t4"
    operator: "Exists"
    effect: "NoExecute"
    tolerationSeconds: 30
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
EOF
```



```
kubectl  create  -f pod.yaml
```



```
kubectl  get pod post-pod  -w
```



# 查看coredns 的 toleration





```
kubectl  get pod -n kube-system  coredns-7ff77c879f-44d6p  -o json | jq .spec.tolerations
```



```
[
  {
    "key": "CriticalAddonsOnly",
    "operator": "Exists"
  },
  {
    "effect": "NoSchedule",
    "key": "node-role.kubernetes.io/master"
  },
  {
    "effect": "NoExecute",
    "key": "node.kubernetes.io/not-ready",
    "operator": "Exists",
    "tolerationSeconds": 300
  },
  {
    "effect": "NoExecute",
    "key": "node.kubernetes.io/unreachable",
    "operator": "Exists",
    "tolerationSeconds": 300
  }
]
```



```
kubectl  get  node 192.168.3.101 -o json | jq .spec.taints
```



```
[
  {
    "effect": "NoSchedule",
    "key": "node-role.kubernetes.io/master"
  }
]
```



```
kubectl taint node   192.168.3.101   node-role.kubernetes.io/master:NoSchedule
```



```
kubectl taint node   192.168.3.101   node-role.kubernetes.io/master:NoSchedule-
```

















# 匹配只有key 和effect 的taint





```
kubectl patch node 192.168.3.101 -p '{"spec":{"taints":[]}}'
kubectl taint node   192.168.3.101   node-role.kubernetes.io/master:NoSchedule
```





```
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
EOF
```



```
kubectl   create -f pod.yaml
```





------------------

```
kubectl  get pod myapp-pod 
```



```
NAME        READY   STATUS    RESTARTS   AGE
myapp-pod   0/1     Pending   0          26s
```

--------------------





### 1

```
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  labels:
    app: app
spec:
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
EOF
```





```
kubectl  create  -f pod.yaml
kubectl  get pod  test-pod
```



### 2

```
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  labels:
    app: app
spec:
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
    operator: Equal
EOF
```



```
kubectl delete pod test-pod
kubectl  create  -f pod.yaml
kubectl  get pod  test-pod
```

### 3



```
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  labels:
    app: app
spec:
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
    operator: Equal
    value: ""
EOF
```



```
kubectl delete pod test-pod
kubectl  create  -f pod.yaml
kubectl  get pod  test-pod
```



### 4



```
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  labels:
    app: app
spec:
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
    operator: Exists
    value: ""
EOF
```



```
kubectl delete pod test-pod
kubectl  create  -f pod.yaml
kubectl  get pod  test-pod
```



### 6



```
cat > pod.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  labels:
    app: app
spec:
  containers:
  - name: myapp-container
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
    operator: Exists
EOF
```



```
kubectl delete pod test-pod
kubectl  create  -f pod.yaml
kubectl  get pod  test-pod
```




# metrics-server



```
kubectl taint node   192.168.3.101   node-role.kubernetes.io/master:NoSchedule-
kubectl taint node   192.168.3.101   node-role.kubernetes.io/master:NoSchedule
```



```
rm -rf /opt/metrics-yaml/
```



```
yum -y install git
git clone https://github.com/yimtun/metrics-yaml.git   /opt/metrics-yaml
```



```
kubectl  delete -f /opt/metrics-yaml/1.8+/
```

```
kubectl  create -f /opt/metrics-yaml/1.8+/
```



```
kubectl  -n kube-system  get pod -l k8s-app=metrics-server  -w
```





```
vim /opt/metrics-yaml/1.8+/metrics-server-deployment.yaml 
```



```
cat /opt/metrics-yaml/1.8+/metrics-server-deployment.yaml 
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: metrics-server
  namespace: kube-system
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metrics-server
  namespace: kube-system
  labels:
    k8s-app: metrics-server
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  template:
    metadata:
      name: metrics-server
      labels:
        k8s-app: metrics-server
    spec:
      serviceAccountName: metrics-server
      volumes:
      # mount in tmp so we can safely use from-scratch images and/or read-only containers
      - name: tmp-dir
        emptyDir: {}
      containers:
      - name: metrics-server
        image: k8s.gcr.io/metrics-server-amd64:v0.3.6
        command:
        - /metrics-server
        - --kubelet-insecure-tls
        - --v=10
        imagePullPolicy: Never
        volumeMounts:
        - name: tmp-dir
          mountPath: /tmp
      tolerations:
      - effect: NoSchedule
        key: node-role.kubernetes.io/master
```





```
kubectl  apply  -f /opt/metrics-yaml/1.8+/metrics-server-deployment.yaml
```





# kubeadm 默认为master添加taint的目的



```
kubectl taint node   192.168.3.101   node-role.kubernetes.io/master:NoSchedule
```



