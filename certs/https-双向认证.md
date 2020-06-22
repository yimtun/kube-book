# 本节学习目标

可以灵活的给k8s服务创建证书 



# 前置补充



参考链接



> https://www.51know.info/docs/security/other/openssl/



客户端默认是相信权威CA机构的 操作系统内置了 CA 证书   这一点是 基础，也就是说操作系统默认就有了

CA 证书的公钥

```
centos 下 被信任的证书在此文件中
/etc/pki/tls/certs/ca-bundle.crt
```





```
服务方S向第三方机构CA提交公钥、组织信息、个人信息(域名)等信息并申请认证;

CA通过线上、线下等多种手段验证申请者提供信息的真实性，如组织是否存在、企业是否合法，是否拥有域名的所有权等;

如信息审核通过，CA会向申请者签发认证文件-证书。 证书包含以下信息：申请者公钥、申请者的组织信息和个人信息、签发机构 CA的信息、有效时间、证书序列号等信息的明文，同时包含一个签名; 签名的产生算法：首先，使用散列函数计算公开的明文信息的信息摘要，然后，采用 CA的私钥对信息摘要进行加密，密文即签名;

客户端 C 向服务器 S 发出请求时，S 返回证书文件;

客户端 C读取证书中的相关的明文信息，采用相同的散列函数计算得到信息摘要，然后，利用对应 CA的公钥解密签名数据，对比证书的信息摘要，如果一致，则可以确认证书的合法性，即公钥合法;

客户端然后验证证书相关的域名信息、有效时间等信息;

```







### CA

CA是证书的签发机构



```
CA也分级别，最高级别的CA叫RootCA，即根证书  低一级别的CA的证书由它来颁发和签名。
```



```
这样我们信任RootCA，那些由RootCA签名过的证书的CA就可以来颁发证书给实体或其它CA了。那谁给RootCA签名呢？他们自己给自己签名，叫自签名。
```





```
openssl s_client -connect www.baidu.com:443
```





### issuer

```
代表 CA 机构 的 名称， issuer 对应 的 Name 类型 很重 要， 简称 为 DN（ Distinguished Name， 可分 辨 名称）。 issuer=/C=BE/O=GlobalSign nv-sa/CN=GlobalSign Organization Validation CA - SHA256 - G2
```



### subject



```
代表 服务器 实体 的 名称， 该 组织 向 CA 机构 申请 证书， 其 对应 的 Name 类型 和 issuer 的 Name 类型 是 一样 的， CN 表示 服务器 实体 的 域名（ 比如 CN = www. example. com）。 对于 Web 网 站 来说， 每个 网 站 都有 一个 域名， 证书 和 域名 息息相关， 早期 证书 校验 方 校验 证书 的 时候 是将 URL 中的 域名 和 证书 subject 值 中的 CN 比较， 如果 一致， 代表 证书 校验 成功。 随着 时间 的 推移， 一张 证书 可能 包含 多个 域名， 所以 不再 使用 CN 来 校验 证书 域名 了， 而使 用 SAN 证书 扩展 进行 域名
```







# sys init





```
sed -i 's/enforcing/disabled/g'  /etc/selinux/config
setenforce 0
sed -i 's/#UseDNS yes/UseDNS no/g'   /etc/ssh/sshd_config
systemctl   restart sshd
grep DNS               /etc/ssh/sshd_config
grep SELINUX=disabled  /etc/selinux/config 
systemctl  disable firewalld  NetworkManager
systemctl  stop    firewalld    NetworkManager
mv /etc/yum.repos.d/CentOS-Base.repo      /etc/yum.repos.d/CentOS-Base.repo.backup
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
curl  -o /etc/yum.repos.d/epel.repo       http://mirrors.aliyun.com/repo/epel-7.repo
```



#  nginx https  自签证书




```
yum -y install nginx
```



```
cat > /etc/nginx/conf.d/test.conf  << EOF
server {
       listen       443;
        server_name  _;
        root         /usr/share/nginx/html;
        ssl  on;  
        ssl_certificate      /cert/server.crt;
        ssl_certificate_key  /cert/server.key;
        #ssl_client_certificate /cert/client.crt;
        #ssl_verify_client on;

        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;
        location / {
        }
        error_page 404 /404.html;
            location = /40x.html {
        }
        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
EOF
```





```
rm -rf /cert/
mkdir /cert
cd /cert/
```



```
openssl genrsa -out server.key 2048
openssl req -x509 -new -nodes -key server.key -subj "/CN=yandun.com" -days 10000 -out server.crt
```



```
openssl x509 -in server.crt   -noout -text
```



```
systemctl  restart nginx
```





### 直接访问服务端失败

```
curl   https://192.168.3.103
```



```
curl: (60) Peer's certificate issuer has been marked as not trusted by the user.
```



### 不验证证书直接访问

```
curl   https://192.168.3.103  -k
```



### 使用server.crt 作为ca证书验证服务端

```
curl   https://192.168.3.103 --cacert /cert/server.crt 
```



```
curl: (51) Unable to communicate securely with peer: requested domain name does not match the server's certificate.
```



证书绑定的域名和当前请求域名不匹配



####  解决方法1

```
curl   https://yandun.com --cacert /cert/server.crt  --resolve yandun.com:443:192.168.3.103
```



#### 解决方法2



让客户端解析 yandun.com 为 ip 192.168.3.103



备份配置

```
cp /etc/hosts /opt/
```

```
echo   "192.168.3.103  yandun.com"   >> /etc/hosts
```



```
curl   https://yandun.com --cacert /cert/server.crt
```



测试后undo

```
cp /opt/hosts   /etc/
```



###  curl 验证服务端时 不指定证书路径



```
cp  /etc/pki/tls/certs/ca-bundle.crt  /opt/
```



```
cat /cert/server.crt  >> /etc/pki/tls/certs/ca-bundle.crt
```



```
curl   https://yandun.com
```



```
curl   https://yandun.com  --resolve yandun.com:443:192.168.3.103
```





undo

```
cp /opt/ca-bundle.crt  /etc/pki/tls/certs/
```









#  第二阶段  使用ca签发证书



```
rm -rf /cert/
mkdir /cert
cd /cert/
```



##   ca

```
openssl genrsa -out ca.key 2048
openssl req -x509 -new -nodes -key ca.key -subj "/CN=yandun-ca.com" -days 5000 -out ca.crt
```







```
openssl genrsa -out server.key 2048
openssl req -new -key server.key -subj "/CN=yandun.com" -out server.csr
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 5000
```







```
cat > /etc/nginx/conf.d/test.conf  << EOF
server {
       listen       443;
        server_name  _;
        root         /usr/share/nginx/html;
        ssl  on;  
        ssl_certificate      /cert/server.crt;
        ssl_certificate_key  /cert/server.key;
        #ssl_client_certificate /cert/client.crt;
        #ssl_verify_client on;

        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;
        location / {
        }
        error_page 404 /404.html;
            location = /40x.html {
        }
        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
EOF
```





```
systemctl  restart nginx
```



```
curl   https://yandun.com  --cacert /cert/server.crt  --resolve yandun.com:443:192.168.3.103
```



```
curl: (60) Peer's Certificate issuer is not recognized.
```





```
curl   https://yandun.com  --cacert /cert/ca.crt  --resolve yandun.com:443:192.168.3.103
```





```
curl   https://192.168.3.103  --cacert /cert/ca.crt
```



```
curl: (51) Unable to communicate securely with peer: requested domain name does not match the server's certificate.
```





# 客户端使用自签证书供服务端验证



###  第一阶段



```
cat > /etc/nginx/conf.d/test.conf  << EOF
server {
       listen       443;
        server_name  _;
        root         /usr/share/nginx/html;


        ssl  on;  
        ssl_certificate      /cert/server.crt;
        ssl_certificate_key  /cert/server.key;
        ssl_client_certificate /cert/client.crt;
        ssl_verify_client on;

        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;
       

        # Load configuration files for the default server block.

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
EOF
```







```
openssl genrsa -out client.key 2048
openssl req -x509 -new -nodes -key client.key -subj "/CN=client" -days 10000 -out client.crt
```



```
systemctl  restart nginx
```





```
curl   https://yandun.com  --cacert /cert/ca.crt  --resolve yandun.com:443:192.168.3.103
```



```
<html>
<head><title>400 No required SSL certificate was sent</title></head>
<body>
<center><h1>400 Bad Request</h1></center>
<center>No required SSL certificate was sent</center>
<hr><center>nginx/1.16.1</center>
</body>
</html>
```







```
curl   https://yandun.com  --cacert /cert/ca.crt  --cert /cert/client.crt  --key  /cert/client.key --resolve yandun.com:443:192.168.3.103
```









# 使用 根证书 签发客户端证书供服务端验证





nginx 配置文件



```
cat  > /etc/nginx/conf.d/test.conf  << EOF
server {
       listen       443;
        server_name  _;
        root         /usr/share/nginx/html;
        ssl  on;  
        ssl_certificate      /cert/server.crt;
        ssl_certificate_key  /cert/server.key;
        ssl_client_certificate /cert/client-ca.crt;
        ssl_verify_client on;

        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;
        location / {
        }
        error_page 404 /404.html;
            location = /40x.html {
        }
        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
EOF
```





创建一个 根证书   名字  client-ca.crt



```
openssl genrsa -out client-ca.key 2048
openssl req -x509 -new -nodes -key client-ca.key -subj "/CN=client-ca" -days 5000 -out client-ca.crt	
```





```
openssl genrsa -out client.key 2048
openssl req -new -key client.key -subj "/CN=client" -out client.csr
```





```
cat > client.ext << EOF
extendedKeyUsage=clientAuth
EOF
```





```
openssl x509 -req -in client.csr -CA client-ca.crt -CAkey client-ca.key -CAcreateserial -extfile client.ext -out client.crt -days 5000
```





```
systemctl  restart nginx
```



```
curl https://yandun.com  --cert /cert/client.crt  --key  /cert/client.key  --cacert /cert/ca.crt  --resolve yandun.com:443:192.168.3.103
```









#  扩展补充  直接使用ip 访问



```
curl https://192.168.3.103  --cert /cert/client.crt  --key  /cert/client.key  --cacert /cert/ca.crt 
```





```
curl: (51) Unable to communicate securely with peer: requested domain name does not match the server's certificate.
```







```
echo subjectAltName = IP:192.168.3.103,IP:127.0.0.1 > extfile.cnf
```



```
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 5000 -extfile extfile.cnf
```



```
systemctl  restart nginx
```



```
curl https://192.168.3.103  --cert /cert/client.crt  --key  /cert/client.key  --cacert /cert/ca.crt 
```

















