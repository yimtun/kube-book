

# 关闭 GO111MODULE



```
export GO111MODULE=off
```



# 源码查看





### PodLogOptions



```
go doc k8s.io/kubernetes/pkg/apis/core.PodLogOptions
```



```
package core // import "k8s.io/kubernetes/pkg/apis/core"

type PodLogOptions struct {
	metav1.TypeMeta

	// Container for which to return logs
	Container string
	// If true, follow the logs for the pod
	Follow bool
	// If true, return previous terminated container logs
	Previous bool
	// A relative time in seconds before the current time from which to show logs. If this value
	// precedes the time a pod was started, only logs since the pod start will be returned.
	// If this value is in the future, no logs will be returned.
	// Only one of sinceSeconds or sinceTime may be specified.
	SinceSeconds *int64
	// An RFC3339 timestamp from which to show logs. If this value
	// precedes the time a pod was started, only logs since the pod start will be returned.
	// If this value is in the future, no logs will be returned.
	// Only one of sinceSeconds or sinceTime may be specified.
	SinceTime *metav1.Time
	// If true, add an RFC 3339 timestamp with 9 digits of fractional seconds at the beginning of every line
	// of log output.
	Timestamps bool
	// If set, the number of lines from the end of the logs to show. If not specified,
	// logs are shown from the creation of the container or sinceSeconds or sinceTime
	TailLines *int64
	// If set, the number of bytes to read from the server before terminating the
	// log output. This may not display a complete final line of logging, and may return
	// slightly more or slightly less than the specified limit.
	LimitBytes *int64

	// insecureSkipTLSVerifyBackend indicates that the apiserver should not confirm the validity of the
	// serving certificate of the backend it is connecting to.  This will make the HTTPS connection between the apiserver
	// and the backend insecure. This means the apiserver cannot verify the log data it is receiving came from the real
	// kubelet.  If the kubelet is configured to verify the apiserver's TLS credentials, it does not mean the
	// connection to the real kubelet is vulnerable to a man in the middle attack (e.g. an attacker could not intercept
	// the actual log data coming from the real kubelet).
	// +optional
	InsecureSkipTLSVerifyBackend bool
}
    PodLogOptions is the query options for a Pod's logs REST call

func (in *PodLogOptions) DeepCopy() *PodLogOptions
func (in *PodLogOptions) DeepCopyInto(out *PodLogOptions)
func (in *PodLogOptions) DeepCopyObject() runtime.Object
```





```
insecureSkipTLSVerifyBackend indicates that the apiserver should not confirm the validity of the serving certificate of the backend it is connecting to.  This will make the HTTPS connection between the apiserver and the backend insecure. This means the apiserver cannot verify the log data it is receiving came from the real kubelet.  If the kubelet is configured to verify the apiserver's TLS credentials, it does not mean the
connection to the real kubelet is vulnerable to a man in the middle attack (e.g. an attacker could not intercept the actual log data coming from the real kubelet). +optional
```



```
InsecureSkiptsVerifyBackend表示apiserver不应确认其所连接的后端的服务证书的有效性。这将使apiserver和后端之间的HTTPS连接不安全。这意味着apiserver无法验证它接收到的日志数据来自真实的kubelet。如果kubelet配置为验证apiserver的TLS凭据，则并不意味着

与真实kubelet的连接容易受到中间人攻击（例如，攻击者无法拦截来自真实kubelet的实际日志数据）。+可选
```



--------------------



### LogREST.Get



```
go doc k8s.io/kubernetes/pkg/registry/core/pod/rest.LogREST.Get
```



```
package rest // import "k8s.io/kubernetes/pkg/registry/core/pod/rest"

func (r *LogREST) Get(ctx context.Context, name string, opts runtime.Object) (runtime.Object, error)
    Get retrieves a runtime.Object that will stream the contents of the pod log
```



```
Get检索运行时.对象它将对pod日志的内容进行流式处理
```



```
// Get retrieves a runtime.Object that will stream the contents of the pod log
func (r *LogREST) Get(ctx context.Context, name string, opts runtime.Object) (runtime.Object, error) {
        logOpts, ok := opts.(*api.PodLogOptions)
        if !ok {
                return nil, fmt.Errorf("invalid options object: %#v", opts)
        }
        if !utilfeature.DefaultFeatureGate.Enabled(features.AllowInsecureBackendProxy) {
                logOpts.InsecureSkipTLSVerifyBackend = false
        }

        if errs := validation.ValidatePodLogOptions(logOpts); len(errs) > 0 {
                return nil, errors.NewInvalid(api.Kind("PodLogOptions"), name, errs)
        }
        location, transport, err := pod.LogLocation(ctx, r.Store, r.KubeletConn, name, logOpts)
        if err != nil {
                return nil, err
        }
        return &genericrest.LocationStreamer{
                Location:        location,
                Transport:       transport,
                ContentType:     "text/plain",
                Flush:           logOpts.Follow,
                ResponseChecker: genericrest.NewGenericHttpResponseChecker(api.Resource("pods/log"), name),
                RedirectChecker: genericrest.PreventRedirects,
        }, nil
}
```



------------------


### LogLocation

```
go doc k8s.io/kubernetes/pkg/registry/core/pod.LogLocation
```



```
package pod // import "k8s.io/kubernetes/pkg/registry/core/pod"

func LogLocation(
	ctx context.Context, getter ResourceGetter,
	connInfo client.ConnectionInfoGetter,
	name string,
	opts *api.PodLogOptions,
) (*url.URL, http.RoundTripper, error)
    LogLocation returns the log URL for a pod container. If opts.Container is
    blank and only one container is present in the pod, that container is used.
```



```
LogLocation返回pod容器的日志URL。如果选择容器是

空的并且只有一个容器存在于吊舱中，则使用该容器
```



### PodAttachOptions

```
go doc k8s.io/kubernetes/pkg/apis/core.PodAttachOptions
```



```
package core // import "k8s.io/kubernetes/pkg/apis/core"

type PodAttachOptions struct {
	metav1.TypeMeta

	// Stdin if true indicates that stdin is to be redirected for the attach call
	// +optional
	Stdin bool

	// Stdout if true indicates that stdout is to be redirected for the attach call
	// +optional
	Stdout bool

	// Stderr if true indicates that stderr is to be redirected for the attach call
	// +optional
	Stderr bool

	// TTY if true indicates that a tty will be allocated for the attach call
	// +optional
	TTY bool

	// Container to attach to.
	// +optional
	Container string
}
    PodAttachOptions is the query options to a Pod's remote attach call TODO:
    merge w/ PodExecOptions below for stdin, stdout, etc

func (in *PodAttachOptions) DeepCopy() *PodAttachOptions
func (in *PodAttachOptions) DeepCopyInto(out *PodAttachOptions)
func (in *PodAttachOptions) DeepCopyObject() runtime.Object
```



### LocationStreamer

```
go doc k8s.io/apiserver/pkg/registry/generic/rest.LocationStreamer
```



```
package rest // import "k8s.io/apiserver/pkg/registry/generic/rest"

WARNING: package source is installed in "k8s.io/kubernetes/staging/src/k8s.io/apiserver/pkg/registry/generic/rest"
type LocationStreamer struct {
	Location        *url.URL
	Transport       http.RoundTripper
	ContentType     string
	Flush           bool
	ResponseChecker HttpResponseChecker
	RedirectChecker func(req *http.Request, via []*http.Request) error
}
    LocationStreamer is a resource that streams the contents of a particular
    location URL.

func (obj *LocationStreamer) DeepCopyObject() runtime.Object
func (obj *LocationStreamer) GetObjectKind() schema.ObjectKind
func (s *LocationStreamer) InputStream(ctx context.Context, apiVersion, acceptHeader string) (stream io.ReadCloser, flush bool, contentType string, err error)
```





### http.RoundTripper



```
go doc http.RoundTripper
```



```
package http // import "net/http"

type RoundTripper interface {
	// RoundTrip executes a single HTTP transaction, returning
	// a Response for the provided Request.
	//
	// RoundTrip should not attempt to interpret the response. In
	// particular, RoundTrip must return err == nil if it obtained
	// a response, regardless of the response's HTTP status code.
	// A non-nil err should be reserved for failure to obtain a
	// response. Similarly, RoundTrip should not attempt to
	// handle higher-level protocol details such as redirects,
	// authentication, or cookies.
	//
	// RoundTrip should not modify the request, except for
	// consuming and closing the Request's Body. RoundTrip may
	// read fields of the request in a separate goroutine. Callers
	// should not mutate or reuse the request until the Response's
	// Body has been closed.
	//
	// RoundTrip must always close the body, including on errors,
	// but depending on the implementation may do so in a separate
	// goroutine even after RoundTrip returns. This means that
	// callers wanting to reuse the body for subsequent requests
	// must arrange to wait for the Close call before doing so.
	//
	// The Request's URL and Header fields must be initialized.
	RoundTrip(*Request) (*Response, error)
}
    RoundTripper is an interface representing the ability to execute a single
    HTTP transaction, obtaining the Response for a given Request.

    A RoundTripper must be safe for concurrent use by multiple goroutines.

var DefaultTransport RoundTripper = &Transport{ ... }
func NewFileTransport(fs FileSystem) RoundTripper
```



###  rest.LogREST



```
go doc k8s.io/kubernetes/pkg/registry/core/pod/rest.LogREST
```



```
package rest // import "k8s.io/kubernetes/pkg/registry/core/pod/rest"

type LogREST struct {
	KubeletConn client.ConnectionInfoGetter
	Store       *genericregistry.Store
}
    LogREST implements the log endpoint for a Pod

func (r *LogREST) Get(ctx context.Context, name string, opts runtime.Object) (runtime.Object, error)
func (r *LogREST) New() runtime.Object
func (r *LogREST) NewGetOptions() (runtime.Object, bool, string)
func (r *LogREST) OverrideMetricsVerb(oldVerb string) (newVerb string)
func (r *LogREST) ProducesMIMETypes(verb string) []string
func (r *LogREST) ProducesObject(verb string) interface{}
```



### client.ConnectionInfoGetter



```
go doc k8s.io/kubernetes/pkg/kubelet/client.ConnectionInfoGetter
```



```
package client // import "k8s.io/kubernetes/pkg/kubelet/client"

type ConnectionInfoGetter interface {
	GetConnectionInfo(ctx context.Context, nodeName types.NodeName) (*ConnectionInfo, error)
}
    ConnectionInfoGetter provides ConnectionInfo for the kubelet running on a
    named node

func NewNodeConnectionInfoGetter(nodes NodeGetter, config KubeletClientConfig) (ConnectionInfoGetter, error)
```





### client.GetConnectionInfo

```
go doc k8s.io/kubernetes/pkg/kubelet/client.GetConnectionInfo
```



```
package client // import "k8s.io/kubernetes/pkg/kubelet/client"

func (k *NodeConnectionInfoGetter) GetConnectionInfo(ctx context.Context, nodeName types.NodeName) (*ConnectionInfo, error)
    GetConnectionInfo retrieves connection info from the status of a Node API
    object.
```



```
GetConnectionInfo从节点API的状态检索连接信息对象
```



```
vim /gopath/src/k8s.io/kubernetes/pkg/kubelet/client/kubelet_client.go
```



```
// GetConnectionInfo retrieves connection info from the status of a Node API object.
func (k *NodeConnectionInfoGetter) GetConnectionInfo(ctx context.Context, nodeName types.NodeName) (*ConnectionInfo, error) {
	node, err := k.nodes.Get(ctx, string(nodeName), metav1.GetOptions{})
	if err != nil {
		return nil, err
	}

	// Find a kubelet-reported address, using preferred address type
	host, err := nodeutil.GetPreferredNodeAddress(node, k.preferredAddressTypes)
	if err != nil {
		return nil, err
	}

	// Use the kubelet-reported port, if present
	port := int(node.Status.DaemonEndpoints.KubeletEndpoint.Port)
	if port <= 0 {
		port = k.defaultPort
	}

	return &ConnectionInfo{
		Scheme:                         k.scheme,
		Hostname:                       host,
		Port:                           strconv.Itoa(port),
		Transport:                      k.transport,
		InsecureSkipTLSVerifyTransport: k.insecureSkipTLSVerifyTransport,
	}, nil
}
```



###  client.MakeInsecureTransport


```
package client // import "k8s.io/kubernetes/pkg/kubelet/client"

func MakeInsecureTransport(config *KubeletClientConfig) (http.RoundTripper, error)
    MakeInsecureTransport creates an insecure RoundTripper for HTTP Transport.
```


### client.NodeConnectionInfoGetter





```
go doc k8s.io/kubernetes/pkg/kubelet/client.NodeConnectionInfoGetter
```

```
package client // import "k8s.io/kubernetes/pkg/kubelet/client"

type NodeConnectionInfoGetter struct {
	// Has unexported fields.
}
    NodeConnectionInfoGetter obtains connection info from the status of a Node
    API object

func (k *NodeConnectionInfoGetter) GetConnectionInfo(ctx context.Context, nodeName types.NodeName) (*ConnectionInfo, error)
```



```
type NodeConnectionInfoGetter struct {
        // nodes is used to look up Node objects
        nodes NodeGetter
        // scheme is the scheme to use to connect to all kubelets
        scheme string
        // defaultPort is the port to use if no Kubelet endpoint port is recorded in the node status
        defaultPort int
        // transport is the transport to use to send a request to all kubelets
        transport http.RoundTripper
        // insecureSkipTLSVerifyTransport is the transport to use if the kube-apiserver wants to skip verifying the TLS certificate of the kubelet
        insecureSkipTLSVerifyTransport http.RoundTripper
        // preferredAddressTypes specifies the preferred order to use to find a node address
        preferredAddressTypes []v1.NodeAddressType
}
```



### client.NodeGetter

```
go doc k8s.io/kubernetes/pkg/kubelet/client.NodeGetter
```



```
package client // import "k8s.io/kubernetes/pkg/kubelet/client"

type NodeGetter interface {
	Get(ctx context.Context, name string, options metav1.GetOptions) (*v1.Node, error)
}
    NodeGetter defines an interface for looking up a node by name
```



### v1.Node

```
go doc k8s.io/api/core/v1.Node
```



```
package v1 // import "k8s.io/api/core/v1"

WARNING: package source is installed in "k8s.io/kubernetes/staging/src/k8s.io/api/core/v1"
type Node struct {
	metav1.TypeMeta `json:",inline"`
	// Standard object's metadata.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
	// +optional
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

	// Spec defines the behavior of a node.
	// https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
	// +optional
	Spec NodeSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`

	// Most recently observed status of the node.
	// Populated by the system.
	// Read-only.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
	// +optional
	Status NodeStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
    Node is a worker node in Kubernetes. Each node will have a unique identifier
    in the cache (i.e. in etcd).

func (in *Node) DeepCopy() *Node
func (in *Node) DeepCopyInto(out *Node)
func (in *Node) DeepCopyObject() runtime.Object
func (*Node) Descriptor() ([]byte, []int)
func (m *Node) Marshal() (dAtA []byte, err error)
func (m *Node) MarshalTo(dAtA []byte) (int, error)
func (m *Node) MarshalToSizedBuffer(dAtA []byte) (int, error)
func (*Node) ProtoMessage()
func (m *Node) Reset()
func (m *Node) Size() (n int)
func (this *Node) String() string
func (Node) SwaggerDoc() map[string]string
func (m *Node) Unmarshal(dAtA []byte) error
func (m *Node) XXX_DiscardUnknown()
func (m *Node) XXX_Marshal(b []byte, deterministic bool) ([]byte, error)
func (m *Node) XXX_Merge(src proto.Message)
func (m *Node) XXX_Size() int
func (m *Node) XXX_Unmarshal(b []byte) error
```





###  features

```
go  doc k8s.io/kubernetes/pkg/features
```







###  features.AllowInsecureBackendProxy

```
go  doc k8s.io/kubernetes/pkg/features.AllowInsecureBackendProxy
```

```
// Enables the users to skip TLS verification of kubelets on pod logs requests
        AllowInsecureBackendProxy featuregate.Feature = "AllowInsecureBackendProxy"
```



```
        AllowInsecureBackendProxy:                      {Default: true, PreRelease: featuregate.Beta},
```



> https://kubernetes.cn/docs/reference/command-line-tools-reference/kube-apiserver/



```
AllowInsecureBackendProxy=true|false (BETA - default=true)
```





> https://v1-18.docs.kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/

```
AllowInsecureBackendProxy=true|false (BETA - default=true)
```



##  PodExec





### core.PodExecOptions



```
go  doc k8s.io/kubernetes/pkg/apis/core.PodExecOptions
```



```
package core // import "k8s.io/kubernetes/pkg/apis/core"

type PodExecOptions struct {
	metav1.TypeMeta

	// Stdin if true indicates that stdin is to be redirected for the exec call
	Stdin bool

	// Stdout if true indicates that stdout is to be redirected for the exec call
	Stdout bool

	// Stderr if true indicates that stderr is to be redirected for the exec call
	Stderr bool

	// TTY if true indicates that a tty will be allocated for the exec call
	TTY bool

	// Container in which to execute the command.
	Container string

	// Command is the remote command to execute; argv array; not executed within a shell.
	Command []string
}
    PodExecOptions is the query options to a Pod's remote exec call

func (in *PodExecOptions) DeepCopy() *PodExecOptions
func (in *PodExecOptions) DeepCopyInto(out *PodExecOptions)
func (in *PodExecOptions) DeepCopyObject() runtime.Object
```



> https://mp.weixin.qq.com/s/E5sKjCDj99Miy2XeklpSeQ





###  rest.ExecREST



```
go  doc k8s.io/kubernetes/pkg/registry/core/pod/rest.ExecREST
```



```
package rest // import "k8s.io/kubernetes/pkg/registry/core/pod/rest"

type ExecREST struct {
	Store       *genericregistry.Store
	KubeletConn client.ConnectionInfoGetter
}
    ExecREST implements the exec subresource for a Pod

func (r *ExecREST) Connect(ctx context.Context, name string, opts runtime.Object, ...) (http.Handler, error)
func (r *ExecREST) ConnectMethods() []string
func (r *ExecREST) New() runtime.Object
func (r *ExecREST) NewConnectOptions() (runtime.Object, bool, string)
```



### rest.ExecREST.Connect



```
go  doc k8s.io/kubernetes/pkg/registry/core/pod/rest.ExecREST.Connect
```



```
package rest // import "k8s.io/kubernetes/pkg/registry/core/pod/rest"

func (r *ExecREST) Connect(ctx context.Context, name string, opts runtime.Object, responder rest.Responder) (http.Handler, error)
    Connect returns a handler for the pod exec proxy
```





### rest.ExecREST.New



```
go  doc k8s.io/kubernetes/pkg/registry/core/pod/rest.ExecREST.New
```



```
package rest // import "k8s.io/kubernetes/pkg/registry/core/pod/rest"

func (r *ExecREST) New() runtime.Object
    New creates a new podExecOptions object.
```





