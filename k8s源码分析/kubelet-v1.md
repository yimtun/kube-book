# apiserver



```
 --kubelet-client-certificate=/etc/kubernetes/cert/kubernetes.pem \\
 --kubelet-client-key=/etc/kubernetes/cert/kubernetes-key.pem \\
```







# kubelet

```
cat > kubelet.config.json.template <<EOF
{
  "kind": "KubeletConfiguration",
  "apiVersion": "kubelet.config.k8s.io/v1beta1",
  "authentication": {
    "x509": {
      "clientCAFile": "/etc/kubernetes/cert/ca.pem"
    },
    "webhook": {
      "enabled": true,
      "cacheTTL": "2m0s"
    },
    "anonymous": {
      "enabled": false
    }
  },
  "authorization": {
    "mode": "Webhook",
    "webhook": {
      "cacheAuthorizedTTL": "5m0s",
      "cacheUnauthorizedTTL": "30s"
    }
  },
  "address": "192.168.10.242",
  "port": 10250,
  "readOnlyPort": 0,
  "cgroupDriver": "cgroupfs",
  "hairpinMode": "promiscuous-bridge",
  "serializeImagePulls": false,
  "featureGates": {
    "RotateKubeletClientCertificate": true,
    "RotateKubeletServerCertificate": true
  },
  "clusterDomain": "${CLUSTER_DNS_DOMAIN}",
  "clusterDNS": ["${CLUSTER_DNS_SVC_IP}"]
}
EOF
```



```
cat > kubelet.service.template <<EOF
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=docker.service
Requires=docker.service

[Service]
WorkingDirectory=/var/lib/kubelet
ExecStart=/opt/k8s/bin/kubelet \\
  --cert-dir=/etc/kubernetes/cert \\
  --bootstrap-kubeconfig=/etc/kubernetes/kubelet-bootstrap.kubeconfig \\
  --kubeconfig=/etc/kubernetes/kubelet.kubeconfig \\
  --config=/etc/kubernetes/kubelet.config.json \\
  --hostname-override=192.168.10.242 \\
  --pod-infra-container-image=registry.access.redhat.com/rhel7/pod-infrastructure:latest \\
  --allow-privileged=true \\
  --alsologtostderr=true \\
  --logtostderr=false \\
  --log-dir=/var/log/kubernetes \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```





--cert-dir  是kubelet 自动生成的 自签名服务端证书的路径

这个证书是自签名的







kube-apiserver 调用kubelet api的时候

是以下面 这两个 作为凭证的吗

 --kubelet-client-certificate=/etc/kubernetes/cert/kubernetes.pem \\
 --kubelet-client-key=/etc/kubernetes/cert/kubernetes-key.pem \\



如果以上面的客户端证书作为凭证，那么kubelet 使用 /etc/kubernetes/cert/ca.pem 来验证

是可以的 因为api server 使用的kubelet客户端证书就是 ca.pem 签发的

上面的是  kubelet  作为服务端  验证 api server作为客户端 的 客户端证书







api server  在调用 kubelet api 时 如何验证 kubelet 的服务端证书呢

也就是  --cert-dir  下自动生成的 自签名服务端证书如何被客户端验证



api server 调用 kubelet api 的时候  如何验证 kubelet 的服务端证书







-----------------



这里时自己搜索的 链接 我没理解

> https://www.cnblogs.com/zhongpan/p/11964017.html





```
实际上是上述过程的特化，不指定tlsCertFile和tlsPrivateKeyFile时，kubelet会自动生成服务端证书保存在--cert-dir指定目录中，文件名为kubelet.crt和kubelet.key
```



```
这个证书是自签名的，所以apiserver不需要指定--kubelet-certificate-authority，其他配置是一样的
```





```
openssl x509 -in /etc/kubernetes/cert/kubelet-server.pem  -noout -subject -issuer
```

```
openssl x509 -in /var/lib/kubelet/pki/kubelet.crt   -noout -subject -issuer
subject= /CN=192.168.3.101@1593512598
issuer= /CN=192.168.3.101-ca@1593512598
```



```
openssl x509 -in /var/lib/kubelet/pki/kubelet-server-current.pem   -noout -subject -issuer
subject= /O=system:nodes/CN=system:node:192.168.3.101
issuer= /CN=kubernetes
```



```
kubectl  get csr
NAME        AGE   SIGNERNAME                      REQUESTOR                   CONDITION
csr-w6ms5   64s   kubernetes.io/kubelet-serving   system:node:192.168.3.101   Pending
csr-wj4lm   30s   kubernetes.io/kubelet-serving   system:node:192.168.3.101   Pending
```



```
kubectl  certificate approve  csr-wj4lm
```



```
cat /var/lib/kubelet/config.yaml
```



```
RotateKubeletServerCertificate: true
serverTLSBootstrap: true
```



```
kubectl  logs -n kube-system  kube-apiserver-192.168.3.101 
Error from server: Get https://192.168.3.101:10250/containerLogs/kube-system/kube-apiserver-192.168.3.101/kube-apiserver: remote error: tls: internal error
```





```
为 shanhu5739＠ gmail. com。

郑东旭. Kubernetes源码剖析 (Kindle位置50). 电子工业出版社. Kindle 版本. 
```





```
 InitializeTLS checks for a configured TLSCertFile and TLSPrivateKeyFile: if unspecified a new self-signed
certificate and key file are generated. Returns a configured server.TLSOptions object.
```





```
https://github.com/kubernetes/kubernetes/blob/master/cmd/kubelet/app/server.go
```



```
// InitializeTLS checks for a configured TLSCertFile and TLSPrivateKeyFile: if unspecified a new self-signed
// certificate and key file are generated. Returns a configured server.TLSOptions object.
```



```
InitializeTLS
```

```
func InitializeTLS(kf *options.KubeletFlags, kc *kubeletconfiginternal.KubeletConfiguration) (*server.TLSOptions, error) {
	if !kc.ServerTLSBootstrap && kc.TLSCertFile == "" && kc.TLSPrivateKeyFile == "" {
		kc.TLSCertFile = path.Join(kf.CertDirectory, "kubelet.crt")
		kc.TLSPrivateKeyFile = path.Join(kf.CertDirectory, "kubelet.key")

		canReadCertAndKey, err := certutil.CanReadCertAndKey(kc.TLSCertFile, kc.TLSPrivateKeyFile)
		if err != nil {
			return nil, err
		}
		if !canReadCertAndKey {
			hostName, err := nodeutil.GetHostname(kf.HostnameOverride)
			if err != nil {
				return nil, err
			}
			cert, key, err := certutil.GenerateSelfSignedCertKey(hostName, nil, nil)
			if err != nil {
				return nil, fmt.Errorf("unable to generate self signed cert: %v", err)
			}

			if err := certutil.WriteCert(kc.TLSCertFile, cert); err != nil {
				return nil, err
			}

			if err := keyutil.WriteKey(kc.TLSPrivateKeyFile, key); err != nil {
				return nil, err
			}

			klog.V(4).Infof("Using self-signed cert (%s, %s)", kc.TLSCertFile, kc.TLSPrivateKeyFile)
		}
	}

	tlsCipherSuites, err := cliflag.TLSCipherSuites(kc.TLSCipherSuites)
	if err != nil {
		return nil, err
	}

	if len(tlsCipherSuites) > 0 {
		insecureCiphers := flag.InsecureTLSCiphers()
		for i := 0; i < len(tlsCipherSuites); i++ {
			for cipherName, cipherID := range insecureCiphers {
				if tlsCipherSuites[i] == cipherID {
					klog.Warningf("Use of insecure cipher '%s' detected.", cipherName)
				}
			}
		}
	}

	minTLSVersion, err := cliflag.TLSVersion(kc.TLSMinVersion)
	if err != nil {
		return nil, err
	}

	tlsOptions := &server.TLSOptions{
		Config: &tls.Config{
			MinVersion:   minTLSVersion,
			CipherSuites: tlsCipherSuites,
		},
		CertFile: kc.TLSCertFile,
		KeyFile:  kc.TLSPrivateKeyFile,
	}

	if len(kc.Authentication.X509.ClientCAFile) > 0 {
		clientCAs, err := certutil.NewPool(kc.Authentication.X509.ClientCAFile)
		if err != nil {
			return nil, fmt.Errorf("unable to load client CA file %s: %v", kc.Authentication.X509.ClientCAFile, err)
		}
		// Specify allowed CAs for client certificates
		tlsOptions.Config.ClientCAs = clientCAs
		// Populate PeerCertificates in requests, but don't reject connections without verified certificates
		tlsOptions.Config.ClientAuth = tls.RequestClientCert
	}

	return tlsOptions, nil
}
```







```
https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/config/types.go
```



```
ServerTLSBootstrap
```



```
serverTLSBootstrap enables server certificate bootstrap. Instead of self
signing a serving certificate, the Kubelet will request a certificate from
the certificates.k8s.io API. This requires an approver to approve the
certificate signing requests. The RotateKubeletServerCertificate feature
must be enabled.
	ServerTLSBootstrap bool
```

```
serverTLSBootstrap启用服务器证书引导。而不是自己

签署服务证书时，Kubelet将从

certificates.k8s.io API。这需要审批者批准

证书签名请求。RotateKubeletServerCertificate功能

必须启用。
```



```
RotateKubeletServerCertificate": true
```

```
rotateCertificates: true
```





kubelet 参数



```
- --feature-gates=RotateKubeletServerCertificate=true #开启server证书签发
```





```
// Run runs the specified KubeletServer with the given Dependencies. This should never exit.
// The kubeDeps argument may be nil - if so, it is initialized from the settings on KubeletServer.
// Otherwise, the caller is assumed to have set up the Dependencies object and a default one will
// not be generated.
func Run(s *options.KubeletServer, kubeDeps *kubelet.Dependencies, featureGate featuregate.FeatureGate, stopCh <-chan struct{}) error {
	// To help debugging, immediately log version
	klog.Infof("Version: %+v", version.Get())
	if err := initForOS(s.KubeletFlags.WindowsService); err != nil {
		return fmt.Errorf("failed OS init: %v", err)
	}
	if err := run(s, kubeDeps, featureGate, stopCh); err != nil {
		return fmt.Errorf("failed to run Kubelet: %v", err)
	}
	return nil
}
```





```
go get -d k8s.io/kubernetes
```

```
go get -d github.com/kubernetes/kubernetes
```



```
wget https://github.com/kubernetes/kubernetes/archive/release-1.18.zip
```

```
wget  https://github.com/kubernetes/kubernetes/archive/master.zip
```





```
cd /gopath/src/k8s.io/kubernetes/cmd/kubelet
go doc app
```



```
go doc k8s.io/kubernetes/cmd/kubelet/app
```



```

```



```
go doc k8s.io/kubernetes/cmd/kubelet/app.Run
package app // import "k8s.io/kubernetes/cmd/kubelet/app"

func Run(s *options.KubeletServer, kubeDeps *kubelet.Dependencies, featureGate featuregate.FeatureGate, stopCh <-chan struct{}) error
    Run runs the specified KubeletServer with the given Dependencies. This
    should never exit. The kubeDeps argument may be nil - if so, it is
    initialized from the settings on KubeletServer. Otherwise, the caller is
    assumed to have set up the Dependencies object and a default one will not be
    generated.
```





```
cat /gopath/src/k8s.io/kubernetes/cmd/kubelet/kubelet.go
```



```
package main

import (
	"math/rand"
	"os"
	"time"

	"k8s.io/component-base/logs"
	_ "k8s.io/component-base/metrics/prometheus/restclient"
	_ "k8s.io/component-base/metrics/prometheus/version" // for version metric registration
	"k8s.io/kubernetes/cmd/kubelet/app"
)

func main() {
	rand.Seed(time.Now().UnixNano())

	command := app.NewKubeletCommand()
	logs.InitLogs()
	defer logs.FlushLogs()

	if err := command.Execute(); err != nil {
		os.Exit(1)
	}
}
```





```
go doc k8s.io/kubernetes/cmd/kubelet/app.NewKubeletCommand
package app // import "k8s.io/kubernetes/cmd/kubelet/app"

func NewKubeletCommand() *cobra.Command
    NewKubeletCommand creates a *cobra.Command object with default parameters

```



```
go doc k8s.io/kubernetes/cmd/kubelet/app/options
```





```
kubectl  logs -n kube-system  kube-apiserver-192.168.3.101 
```



```
Error from server: Get https://192.168.3.101:10250/containerLogs/kube-system/kube-apiserver-192.168.3.101/kube-apiserver: remote error: tls: internal error
[root@localhost kubelet]# kubectl  logs -n kube-system  kube-apiserver-192.168.3.101  -v=8
I0630 19:01:24.868159   17757 loader.go:375] Config loaded from file:  /root/.kube/config
I0630 19:01:24.873706   17757 round_trippers.go:420] GET https://192.168.3.101:6443/api/v1/namespaces/kube-system/pods/kube-apiserver-192.168.3.101
I0630 19:01:24.873721   17757 round_trippers.go:427] Request Headers:
I0630 19:01:24.873726   17757 round_trippers.go:431]     Accept: application/json, */*
I0630 19:01:24.873732   17757 round_trippers.go:431]     User-Agent: kubectl/v1.18.3 (linux/amd64) kubernetes/2e7996e
I0630 19:01:24.884092   17757 round_trippers.go:446] Response Status: 200 OK in 10 milliseconds
I0630 19:01:24.884108   17757 round_trippers.go:449] Response Headers:
I0630 19:01:24.884112   17757 round_trippers.go:452]     Cache-Control: no-cache, private
I0630 19:01:24.884116   17757 round_trippers.go:452]     Content-Type: application/json
I0630 19:01:24.884119   17757 round_trippers.go:452]     Date: Tue, 30 Jun 2020 11:01:24 GMT
I0630 19:01:24.884186   17757 request.go:1068] Response Body: {"kind":"Pod","apiVersion":"v1","metadata":{"name":"kube-apiserver-192.168.3.101","namespace":"kube-system","selfLink":"/api/v1/namespaces/kube-system/pods/kube-apiserver-192.168.3.101","uid":"9058f969-ab7c-45d3-8780-5542afa6ca05","resourceVersion":"433","creationTimestamp":"2020-06-29T07:48:16Z","labels":{"component":"kube-apiserver","tier":"control-plane"},"annotations":{"kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint":"192.168.3.101:6443","kubernetes.io/config.hash":"1bf0514148610c4ba50b4e80c6d26f69","kubernetes.io/config.mirror":"1bf0514148610c4ba50b4e80c6d26f69","kubernetes.io/config.seen":"2020-06-29T15:48:09.623616307+08:00","kubernetes.io/config.source":"file"},"ownerReferences":[{"apiVersion":"v1","kind":"Node","name":"192.168.3.101","uid":"cd6cff8c-8b28-45ea-869c-aec4836b8949","controller":true}],"managedFields":[{"manager":"kubelet","operation":"Update","apiVersion":"v1","time":"2020-06-29T07:48:35Z","fieldsType":"FieldsV1","fieldsV1":{"f:metadata":{"f:annotations":{".":{},"f:kubea [truncated 6420 chars]
I0630 19:01:24.888463   17757 round_trippers.go:420] GET https://192.168.3.101:6443/api/v1/namespaces/kube-system/pods/kube-apiserver-192.168.3.101/log
I0630 19:01:24.888475   17757 round_trippers.go:427] Request Headers:
I0630 19:01:24.888480   17757 round_trippers.go:431]     Accept: application/json, */*
I0630 19:01:24.888484   17757 round_trippers.go:431]     User-Agent: kubectl/v1.18.3 (linux/amd64) kubernetes/2e7996e
I0630 19:01:24.892083   17757 round_trippers.go:446] Response Status: 500 Internal Server Error in 3 milliseconds
I0630 19:01:24.892098   17757 round_trippers.go:449] Response Headers:
I0630 19:01:24.892103   17757 round_trippers.go:452]     Cache-Control: no-cache, private
I0630 19:01:24.892106   17757 round_trippers.go:452]     Content-Type: application/json
I0630 19:01:24.892110   17757 round_trippers.go:452]     Content-Length: 229
I0630 19:01:24.892113   17757 round_trippers.go:452]     Date: Tue, 30 Jun 2020 11:01:24 GMT
I0630 19:01:24.892130   17757 request.go:1068] Response Body: {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"Get https://192.168.3.101:10250/containerLogs/kube-system/kube-apiserver-192.168.3.101/kube-apiserver: remote error: tls: internal error","code":500}
I0630 19:01:24.892451   17757 helpers.go:216] server response object: [{
  "metadata": {},
  "status": "Failure",
  "message": "Get https://192.168.3.101:10250/containerLogs/kube-system/kube-apiserver-192.168.3.101/kube-apiserver: remote error: tls: internal error",
  "code": 500
}]
F0630 19:01:24.892471   17757 helpers.go:115] Error from server: Get https://192.168.3.101:10250/containerLogs/kube-system/kube-apiserver-192.168.3.101/kube-apiserver: remote error: tls: internal error
```



```
rotateCertificates: true
rotateKubeletServerCertificate: true
serverTLSBootstrap: true
```



```
--insecure-skip-tls-verify-backend=false
```



```
kubectl  logs  myapp-pod  --v=8   --insecure-skip-tls-verify-backend=false
```



---------------

apiserver 

```
E0630 11:24:50.564331       1 status.go:71] apiserver received an error that is not an metav1.Status: &url.Error{Op:"Get", URL:"https://192.168.3.101:10250/containerLogs/default/myapp-pod/myapp-container", Err:(*net.OpError)(0xc0050b6ff0)}
```



kubelet



```
no serving certificate available for the kubelet
```

```
kubectl  get csr
NAME        AGE    SIGNERNAME                      REQUESTOR                   CONDITION
csr-2f6sz   13m    kubernetes.io/kubelet-serving   system:node:192.168.3.101   Approved,Issued
csr-775fd   5m6s   kubernetes.io/kubelet-serving   system:node:192.168.3.101   Pending
csr-frdlt   14m    kubernetes.io/kubelet-serving   system:node:192.168.3.101   Pending
csr-j892q   40m    kubernetes.io/kubelet-serving   system:node:192.168.3.101   Approved,Issued
csr-rtzv9   13m    kubernetes.io/kubelet-serving   system:node:192.168.3.101   Pending
```

```

```





# 自签证书验证



```
curl  https://192.168.3.101:10250/containerLogs/default/myapp-pod/myapp-container -k
Unauthorized[root@localhost ~]# 
```



```
curl  https://192.168.3.101:10250/containerLogs/default/myapp-pod/myapp-container --cert /etc/kubernetes/pki/apiserver-kubelet-client.crt   --key   /etc/kubernetes/pki/apiserver-kubelet-client.key  -k
```



```
curl  https://192.168.3.101:10250/healthz --cert /etc/kubernetes/pki/apiserver-kubelet-client.crt   --key   /etc/kubernetes/pki/apiserver-kubelet-client.key  -k
ok
```



------------------



```
curl  https://192.168.3.101:6443/api/v1/nodes/192.168.3.101 -k  --cert /etc/kubernetes/pki/apiserver-kubelet-client.crt   --key   /etc/kubernetes/pki/apiserver-kubelet-client.key
```

```
https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/#certificate-rotation
```





```
kubelet.crt 该文件在 kubelet 完成 TLS bootstrapping 后并且没有配置 --feature-gates=RotateKubeletServerCertificate=true 时才会生成；这种情况下该文件为一个独立于 apiserver CA 的自签 CA 证书，有效期为 1 年；被用作 kubelet 10250 api 端口

作者：ywhu
链接：https://www.jianshu.com/p/bb973ab1029b
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```



```
https://www.jianshu.com/p/bb973ab1029b
```





# 默认情况 自签证书





```
curl  https://192.168.3.101:10250/containerLogs/default/myapp-pod/myapp-container --cert /etc/kubernetes/pki/apiserver-kubelet-client.crt   --key   /etc/kubernetes/pki/apiserver-kubelet-client.key  --cacert /var/lib/kubelet/pki/kubelet.crt
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
```





# openssl



```
openssl s_client -connect ip-172-18-7-123.ec2.internal:10250 -cert /etc/origin/master/admin.crt -key /etc/origin/master/admin.key  -debug 
```





```
kubectl  logs myapp-pod 
Error from server: Get https://192.168.3.101:10250/containerLogs/default/myapp-pod/myapp-container: remote error: tls: internal error
```



# kubelet



```
/usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --cgroup-driver=systemd --hostname-override=192.168.3.101 --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2 --feature-gates=RotateKubeletServerCertificate=true --rotate-server-certificates=true
```





> https://www.huweihuang.com/tags/

