# 获取程序



```
docker run -it --name=controller-manager  registry.aliyuncs.com/google_containers/kube-controller-manager:v1.18.3 /bin/sh
```



```
docker cp controller-manager:/usr/local/bin/kube-controller-manager /opt
```

```
docker stop controller-manager
docker rm   controller-manager
```

```
cp /opt/kube-controller-manager  /usr/local/bin/
```

-------

```
/usr/local/bin/kube-controller-manager --version
```

```
Kubernetes v1.18.3
```



# 查看启动参数



```
ps -ef | grep kube-controller-manager
```



```
kube-controller-manager --allocate-node-cidrs=true --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf --bind-address=127.0.0.1 --client-ca-file=/etc/kubernetes/pki/ca.crt --cluster-cidr=10.244.0.0/16 --cluster-name=kubernetes --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt --cluster-signing-key-file=/etc/kubernetes/pki/ca.key --controllers=*,bootstrapsigner,tokencleaner --kubeconfig=/etc/kubernetes/controller-manager.conf --leader-elect=true --node-cidr-mask-size=24 --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --root-ca-file=/etc/kubernetes/pki/ca.crt --service-account-private-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --use-service-account-credentials=true
```

-------------

```
cat /etc/kubernetes/manifests/kube-controller-manager.yaml
```

```
    - kube-controller-manager
    - --allocate-node-cidrs=true
    - --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --bind-address=127.0.0.1
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --cluster-cidr=10.244.0.0/16
    - --cluster-name=kubernetes
    - --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
    - --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
    - --controllers=*,bootstrapsigner,tokencleaner
    - --kubeconfig=/etc/kubernetes/controller-manager.conf
    - --leader-elect=true
    - --node-cidr-mask-size=24
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --root-ca-file=/etc/kubernetes/pki/ca.crt
    - --service-account-private-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.96.0.0/12
    - --use-service-account-credentials=true
```



# 查看当前kubeconfig 文件



##  查看kubeconfig 里面的证书

```
cat /etc/kubernetes/controller-manager.conf
```

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01EWXlPREExTkRZeE1Gb1hEVE13TURZeU5qQTFORFl4TUZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTG5jClVIZmFiQzljcFk4MXNKdmwvTUlnU1ZWNHRMUVNoVjI0U280MWNuVy9HMWFWTUtYNk5kR0wwWDl2YTBsS1I4SFMKV25hcVo4K3BnN3hmYVgvVVBMbzU4dlptVDhvZC9DZ0pQaG9iczA5SFc0TklNNmlGOFVkMkVUbkNiUDJkTkNMRApZd29FYUgwTEdhVHE4ejh1ZzE5Y1pXUHI1UGtQdkZxV3JneGQwUWxyRzhyd2R2SkpLVWlFL3hqOS81VkNNTG9xCmVndVo5YjlXWmZEMktMTlhFamFEb25pbzRhMmdxaVBFdEJpaFVhOFdZWU5URHptWVFtRVI3VjRORkx3bW5KV3YKbjNRUGs2aXBOOEI1VmFlbCtaZ1B0bmQwc3VmajBKSG1rVk1Ra2pVdnpvZjRKUEIvVUFXUTRrLzNIaWhLRE9VVQpSeXYvREU5M0h5ZCtuVW9iTG1FQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFJdXlzU3NmM1E1OW9SZ0NyakQvQmV5N0V1KzYKZ3kxZ0FQNzI3OGxBRUV3K0pTclhrenN6MUk4QXBwbmJxb0tKbjlPNmw1UGxLRlA1NWFkS3JZT2Q0TWJKcWtXcQpIT1h3YUVOWVdESldHTjhONVJydXJMMXA5ckQ2K0NPa3ZMSWNMb21vTW0ya2JpRldJMFJ0MVdMaitiK2ZWK2d0ClpqT1orVFE1TmFmeUVRM3BTVUROWFNKYlQ0Y2lxSXR2Z05aNzVvTmYwWmozNDBVeE9jSVhtYityODc2ZDlJMFAKNHNQTkRDVGJSeUY1YXd1VWhXRmtBOXhCREdFS1hOTHVkV29sREowOVJWTU8xQ1A5eStheDJiQnk5TTlNWWtNTQpQa05NanFhYzlwcVB0dVVJUlJkdkZ4QlZZVVZJQnVxV2JTdnNwcWw0L2F0OXN1c1NxV2J0QU5nVGlFST0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    server: https://192.168.3.101:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: system:kube-controller-manager
  name: system:kube-controller-manager@kubernetes
current-context: system:kube-controller-manager@kubernetes
kind: Config
preferences: {}
users:
- name: system:kube-controller-manager
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lJYkdhVmVrdUdDZll3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURBMk1qZ3dOVFEyTVRCYUZ3MHlNVEEyTWpnd05UUTJNVEZhTUNreApKekFsQmdOVkJBTVRIbk41YzNSbGJUcHJkV0psTFdOdmJuUnliMnhzWlhJdGJXRnVZV2RsY2pDQ0FTSXdEUVlKCktvWklodmNOQVFFQkJRQURnZ0VQQURDQ0FRb0NnZ0VCQUxjTWoraVB3cmFZK1FpVU9pVWEwRWk3RnFHWVIzeXoKUTBtMWMrWW1TOUtGU2tLWkRINnR0VnYxbDg2MGc3RUVwU3dEWE9NKzBwa0xlL1NvUlZNRzZsbzZmRVo1UXJIZQpMQzg3S0xoaDVuV216NzNQTlBEQ1h3MFVQaXJnOXlzOGUvc3NkZ3l5V0tTTkppelFYaWpSZjcrOVhpck5ncnI5CityRyt3elZyVGgxOGJhc2cyQkgrdGNwVlpsZWFvQkdUeUoxS1BVQ01pRk5GOG51a3d6SEdUc1NtSlp4Z29mbEoKZFFVZmZ5YlRkY0FOUU51VmdEcGZvK3J0QXBlb0RpYzh2dmdqNDZaNHlxM2ZYOTZCdXd5Tks2NFZuT294MGxMKwppci9kQ0tkUjZzWmU3QzNGdnJKZXpTRUtDMFgzbDdMeHRzakMwWWxrczZxallrVDVzZnhVVDFrQ0F3RUFBYU1uCk1DVXdEZ1lEVlIwUEFRSC9CQVFEQWdXZ01CTUdBMVVkSlFRTU1Bb0dDQ3NHQVFVRkJ3TUNNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFDbGNtV0tPbndoRW5DTnBQSlNFRzAwOHlHcTBtQ3Q2R3dXTWl3amlSZVYrVTRXU1JFbgp2K0thZ3VNazdveWoxSVU5RFkxa280VFJ3ZXNndHJmT1ZqTWhwUytSelErRjA5cTBxRDRPTHZ5dE5DSXMxRGVCCkdydkRJSGhVNHVxUm1DSU5rRUVTT1dxVUNncXJ3RWJQVi9KKzRTVTF1UkVvVEN2a1krb3Y3eHprcDlYVFc4blMKdCtJTFhrS25TazRaSlJxRjZzai9jSUlsYWVZamNaWGZnc0p6QkJxSjZMMzRhaHdndlIrT2FPRGd1THlvMFhjZApQakorQkxXdmZzMC83OFRHSXNyejhrTEJMU3VQSE5sRjdPZi81VjBUZWRDV2ZzcTJWSG93SENJMEF0eXpYdTY2CkhJVkY4eEhFNHNRRFdnS2ZVNGhOSHlrSUJxVUJ4NTE0VXppNgotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBdHd5UDZJL0N0cGo1Q0pRNkpSclFTTHNXb1poSGZMTkRTYlZ6NWlaTDBvVktRcGtNCmZxMjFXL1dYenJTRHNRU2xMQU5jNHo3U21RdDc5S2hGVXdicVdqcDhSbmxDc2Q0c0x6c291R0htZGFiUHZjODAKOE1KZkRSUStLdUQzS3p4Nyt5eDJETEpZcEkwbUxOQmVLTkYvdjcxZUtzMkN1djM2c2I3RE5XdE9IWHh0cXlEWQpFZjYxeWxWbVY1cWdFWlBJblVvOVFJeUlVMFh5ZTZURE1jWk94S1lsbkdDaCtVbDFCUjkvSnROMXdBMUEyNVdBCk9sK2o2dTBDbDZnT0p6eSsrQ1BqcG5qS3JkOWYzb0c3REkwcnJoV2M2akhTVXY2S3Y5MElwMUhxeGw3c0xjVysKc2w3TklRb0xSZmVYc3ZHMnlNTFJpV1N6cXFOaVJQbXgvRlJQV1FJREFRQUJBb0lCQUFjVDMwU1l1bWloQlpBRgpXekl3RlRtYXNrZFJRZ0phVkJHM2lHR3Z2V0xJY0pTZW9sTUxtR1dUMjJqTXBnTGtNUmJBa29qZTF2bS83ZTBKCnpKUm5RZ3gzRW5NUElUc0xZaDM1Wlp1cmZXT3pMWGtqVitLdlFVbWFMTVV2cVo3c1djYmVjem9PYzByNWdpNWQKYUNhZjR4YWcxZEZGM1BZcDk5V0RrTHl5QjNVd1Y2NVlCRjBQbXpVNDBYejEzMWk2RlhTZG9taDhFMlJuY09vZgo3QUdWS09TWExqNzkxcUFjS1RhK0laY2JNUHFTL0NlMldDbko5TjFrQzNkbHlMWTg4NE5vSnpiaFhtQWpKd0ROClBwMEtIL0hpbkFpRFRkUDRxbEVNY0FoSmlBa3l4UFpHWU1nbTZvSzNJeE90ZDJDUVJvSGNIekZVUVYwcG1Wb2QKNERTMUsxa0NnWUVBeXA5dUNzS01weGRMMUJtYTdJam4rblgrSG0rNkhLK2x0cnFxWnVxZVRFblFta2NuRmovSQpMMzJoMDFLOHdnaDdpZkFuNkVFaDIwbGE0Z2pLWk9RcTZ4OTJwTG5RZlZrdklud3hNVkRNRWJ3dkdEbHJpS0JKClVrbktsZXNkbitvWWJsZVFqQjBMRGdQZ3BmM2lKd3lIZkEyOVZyZFh4T3lsc0VUcEpkUHFlc3NDZ1lFQTUwVWMKbXhldXBWaTNxeDN2NlFSaWNTT000Q3JqWjhCdlo5aFUzZGZvNitBT0NNbzBWTVdNYytqeU9MeGVuN3pUZ29TSQplcW1lNTV4dGVCcGJoMUNrSTJzeXZVMzgxY1FSSEF6U1VJWWl5NHcyVms2azZqTmRnRGYzUTVWMFQyOFYrNXdWClhtQUVFakhPZmtEMUFWRDdaWTFMU0pDa014ZzJCQS9uR0I3NjVlc0NnWUFEZEJZdkRzUFE4VCswbkw4Y092VWgKT3JPYkZ6Sm4zTUtKUzhNdHY5LzAwdWxBUitndG8rYW9rSTZhaUhWNUpTWGQ0djc3SVdrUFVML0F6SCtPbXFqMAptdk90dVJFSm9lU0F4UGNkclEvZFdZUy85L0tTUUpFZld1eWVBNFRjdmVPdXRjVmI3ZjdMUFZ1dDJKYnJMWFo5CnNjcEJXUnlnMlp1MVZtZFc0cmJEWXdLQmdRRE1qWlpsbnlhNzNLSm5XWTFQUHErTGZuUWwrZ2sxUlVIRVNkV1cKZWxmcitUcXdqNWlGdWswbVlFMk4zUjZjanJsTlljZ05KbVlFV1ptQmQxNnBhcXdqSDdlN05IV0M1VzUwcnVwKwppb1hRSDI0WUhHdEZNclZxcVJXczAwNFN6Q0JYY1pCODd0UHErOTYyVU9Iamppc3RnVEdyTnpQa2RXK2hYQ2Q4CmNEcGVqUUtCZ0VHcmRnRUdqMVJjYUllNWRkMG5EMFh4ZHZ3dmlkZWNhVnl3bHprbWUzY1ZzNUk5SU4rRnNaYXAKWmxRYnJGenFhUnAzUDFIcnlISnpzN3ZXU1ZtdjI2cUFReXd0SzdUOVR2aDhXcU45QTRCVjQ5d1F2VnI0Z0dTKwpUWEQ2SGQ5WGpZUUhIdHF3cTFtdE5DVFpzRlUweWkwNUh3d1V0ZzI5dTZ3RmZlYkFNMFJzCi0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
```



----------------

#### 解码base64   certificate-authority-data




```
echo 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01EWXlPREExTkRZeE1Gb1hEVE13TURZeU5qQTFORFl4TUZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTG5jClVIZmFiQzljcFk4MXNKdmwvTUlnU1ZWNHRMUVNoVjI0U280MWNuVy9HMWFWTUtYNk5kR0wwWDl2YTBsS1I4SFMKV25hcVo4K3BnN3hmYVgvVVBMbzU4dlptVDhvZC9DZ0pQaG9iczA5SFc0TklNNmlGOFVkMkVUbkNiUDJkTkNMRApZd29FYUgwTEdhVHE4ejh1ZzE5Y1pXUHI1UGtQdkZxV3JneGQwUWxyRzhyd2R2SkpLVWlFL3hqOS81VkNNTG9xCmVndVo5YjlXWmZEMktMTlhFamFEb25pbzRhMmdxaVBFdEJpaFVhOFdZWU5URHptWVFtRVI3VjRORkx3bW5KV3YKbjNRUGs2aXBOOEI1VmFlbCtaZ1B0bmQwc3VmajBKSG1rVk1Ra2pVdnpvZjRKUEIvVUFXUTRrLzNIaWhLRE9VVQpSeXYvREU5M0h5ZCtuVW9iTG1FQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFJdXlzU3NmM1E1OW9SZ0NyakQvQmV5N0V1KzYKZ3kxZ0FQNzI3OGxBRUV3K0pTclhrenN6MUk4QXBwbmJxb0tKbjlPNmw1UGxLRlA1NWFkS3JZT2Q0TWJKcWtXcQpIT1h3YUVOWVdESldHTjhONVJydXJMMXA5ckQ2K0NPa3ZMSWNMb21vTW0ya2JpRldJMFJ0MVdMaitiK2ZWK2d0ClpqT1orVFE1TmFmeUVRM3BTVUROWFNKYlQ0Y2lxSXR2Z05aNzVvTmYwWmozNDBVeE9jSVhtYityODc2ZDlJMFAKNHNQTkRDVGJSeUY1YXd1VWhXRmtBOXhCREdFS1hOTHVkV29sREowOVJWTU8xQ1A5eStheDJiQnk5TTlNWWtNTQpQa05NanFhYzlwcVB0dVVJUlJkdkZ4QlZZVVZJQnVxV2JTdnNwcWw0L2F0OXN1c1NxV2J0QU5nVGlFST0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=' | base64 -d
```





```
-----BEGIN CERTIFICATE-----
MIICyDCCAbCgAwIBAgIBADANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwprdWJl
cm5ldGVzMB4XDTIwMDYyODA1NDYxMFoXDTMwMDYyNjA1NDYxMFowFTETMBEGA1UE
AxMKa3ViZXJuZXRlczCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBALnc
UHfabC9cpY81sJvl/MIgSVV4tLQShV24So41cnW/G1aVMKX6NdGL0X9va0lKR8HS
WnaqZ8+pg7xfaX/UPLo58vZmT8od/CgJPhobs09HW4NIM6iF8Ud2ETnCbP2dNCLD
YwoEaH0LGaTq8z8ug19cZWPr5PkPvFqWrgxd0QlrG8rwdvJJKUiE/xj9/5VCMLoq
eguZ9b9WZfD2KLNXEjaDonio4a2gqiPEtBihUa8WYYNTDzmYQmER7V4NFLwmnJWv
n3QPk6ipN8B5Vael+ZgPtnd0sufj0JHmkVMQkjUvzof4JPB/UAWQ4k/3HihKDOUU
Ryv/DE93Hyd+nUobLmECAwEAAaMjMCEwDgYDVR0PAQH/BAQDAgKkMA8GA1UdEwEB
/wQFMAMBAf8wDQYJKoZIhvcNAQELBQADggEBAIuysSsf3Q59oRgCrjD/Bey7Eu+6
gy1gAP7278lAEEw+JSrXkzsz1I8AppnbqoKJn9O6l5PlKFP55adKrYOd4MbJqkWq
HOXwaENYWDJWGN8N5RrurL1p9rD6+COkvLIcLomoMm2kbiFWI0Rt1WLj+b+fV+gt
ZjOZ+TQ5NafyEQ3pSUDNXSJbT4ciqItvgNZ75oNf0Zj340UxOcIXmb+r876d9I0P
4sPNDCTbRyF5awuUhWFkA9xBDGEKXNLudWolDJ09RVMO1CP9y+ax2bBy9M9MYkMM
PkNMjqac9pqPtuUIRRdvFxBVYUVIBuqWbSvspql4/at9susSqWbtANgTiEI=
-----END CERTIFICATE-----
```

------------

直接将输出写入文件



```
echo 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01EWXlPREExTkRZeE1Gb1hEVE13TURZeU5qQTFORFl4TUZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTG5jClVIZmFiQzljcFk4MXNKdmwvTUlnU1ZWNHRMUVNoVjI0U280MWNuVy9HMWFWTUtYNk5kR0wwWDl2YTBsS1I4SFMKV25hcVo4K3BnN3hmYVgvVVBMbzU4dlptVDhvZC9DZ0pQaG9iczA5SFc0TklNNmlGOFVkMkVUbkNiUDJkTkNMRApZd29FYUgwTEdhVHE4ejh1ZzE5Y1pXUHI1UGtQdkZxV3JneGQwUWxyRzhyd2R2SkpLVWlFL3hqOS81VkNNTG9xCmVndVo5YjlXWmZEMktMTlhFamFEb25pbzRhMmdxaVBFdEJpaFVhOFdZWU5URHptWVFtRVI3VjRORkx3bW5KV3YKbjNRUGs2aXBOOEI1VmFlbCtaZ1B0bmQwc3VmajBKSG1rVk1Ra2pVdnpvZjRKUEIvVUFXUTRrLzNIaWhLRE9VVQpSeXYvREU5M0h5ZCtuVW9iTG1FQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFJdXlzU3NmM1E1OW9SZ0NyakQvQmV5N0V1KzYKZ3kxZ0FQNzI3OGxBRUV3K0pTclhrenN6MUk4QXBwbmJxb0tKbjlPNmw1UGxLRlA1NWFkS3JZT2Q0TWJKcWtXcQpIT1h3YUVOWVdESldHTjhONVJydXJMMXA5ckQ2K0NPa3ZMSWNMb21vTW0ya2JpRldJMFJ0MVdMaitiK2ZWK2d0ClpqT1orVFE1TmFmeUVRM3BTVUROWFNKYlQ0Y2lxSXR2Z05aNzVvTmYwWmozNDBVeE9jSVhtYityODc2ZDlJMFAKNHNQTkRDVGJSeUY1YXd1VWhXRmtBOXhCREdFS1hOTHVkV29sREowOVJWTU8xQ1A5eStheDJiQnk5TTlNWWtNTQpQa05NanFhYzlwcVB0dVVJUlJkdkZ4QlZZVVZJQnVxV2JTdnNwcWw0L2F0OXN1c1NxV2J0QU5nVGlFST0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=' | base64 -d  > /opt/ca.crt
```



查看此证书



```
openssl x509 -in /opt/ca.crt  -noout -text
```

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 0 (0x0)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=kubernetes
        Validity
            Not Before: Jun 28 05:46:10 2020 GMT
            Not After : Jun 26 05:46:10 2030 GMT
        Subject: CN=kubernetes
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:b9:dc:50:77:da:6c:2f:5c:a5:8f:35:b0:9b:e5:
                    fc:c2:20:49:55:78:b4:b4:12:85:5d:b8:4a:8e:35:
                    72:75:bf:1b:56:95:30:a5:fa:35:d1:8b:d1:7f:6f:
                    6b:49:4a:47:c1:d2:5a:76:aa:67:cf:a9:83:bc:5f:
                    69:7f:d4:3c:ba:39:f2:f6:66:4f:ca:1d:fc:28:09:
                    3e:1a:1b:b3:4f:47:5b:83:48:33:a8:85:f1:47:76:
                    11:39:c2:6c:fd:9d:34:22:c3:63:0a:04:68:7d:0b:
                    19:a4:ea:f3:3f:2e:83:5f:5c:65:63:eb:e4:f9:0f:
                    bc:5a:96:ae:0c:5d:d1:09:6b:1b:ca:f0:76:f2:49:
                    29:48:84:ff:18:fd:ff:95:42:30:ba:2a:7a:0b:99:
                    f5:bf:56:65:f0:f6:28:b3:57:12:36:83:a2:78:a8:
                    e1:ad:a0:aa:23:c4:b4:18:a1:51:af:16:61:83:53:
                    0f:39:98:42:61:11:ed:5e:0d:14:bc:26:9c:95:af:
                    9f:74:0f:93:a8:a9:37:c0:79:55:a7:a5:f9:98:0f:
                    b6:77:74:b2:e7:e3:d0:91:e6:91:53:10:92:35:2f:
                    ce:87:f8:24:f0:7f:50:05:90:e2:4f:f7:1e:28:4a:
                    0c:e5:14:47:2b:ff:0c:4f:77:1f:27:7e:9d:4a:1b:
                    2e:61
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment, Certificate Sign
            X509v3 Basic Constraints: critical
                CA:TRUE
    Signature Algorithm: sha256WithRSAEncryption
         8b:b2:b1:2b:1f:dd:0e:7d:a1:18:02:ae:30:ff:05:ec:bb:12:
         ef:ba:83:2d:60:00:fe:f6:ef:c9:40:10:4c:3e:25:2a:d7:93:
         3b:33:d4:8f:00:a6:99:db:aa:82:89:9f:d3:ba:97:93:e5:28:
         53:f9:e5:a7:4a:ad:83:9d:e0:c6:c9:aa:45:aa:1c:e5:f0:68:
         43:58:58:32:56:18:df:0d:e5:1a:ee:ac:bd:69:f6:b0:fa:f8:
         23:a4:bc:b2:1c:2e:89:a8:32:6d:a4:6e:21:56:23:44:6d:d5:
         62:e3:f9:bf:9f:57:e8:2d:66:33:99:f9:34:39:35:a7:f2:11:
         0d:e9:49:40:cd:5d:22:5b:4f:87:22:a8:8b:6f:80:d6:7b:e6:
         83:5f:d1:98:f7:e3:45:31:39:c2:17:99:bf:ab:f3:be:9d:f4:
         8d:0f:e2:c3:cd:0c:24:db:47:21:79:6b:0b:94:85:61:64:03:
         dc:41:0c:61:0a:5c:d2:ee:75:6a:25:0c:9d:3d:45:53:0e:d4:
         23:fd:cb:e6:b1:d9:b0:72:f4:cf:4c:62:43:0c:3e:43:4c:8e:
         a6:9c:f6:9a:8f:b6:e5:08:45:17:6f:17:10:55:61:45:48:06:
         ea:96:6d:2b:ec:a6:a9:78:fd:ab:7d:b2:eb:12:a9:66:ed:00:
         d8:13:88:42
```



#### 解码client-certificate-data



```
echo 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lJYkdhVmVrdUdDZll3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURBMk1qZ3dOVFEyTVRCYUZ3MHlNVEEyTWpnd05UUTJNVEZhTUNreApKekFsQmdOVkJBTVRIbk41YzNSbGJUcHJkV0psTFdOdmJuUnliMnhzWlhJdGJXRnVZV2RsY2pDQ0FTSXdEUVlKCktvWklodmNOQVFFQkJRQURnZ0VQQURDQ0FRb0NnZ0VCQUxjTWoraVB3cmFZK1FpVU9pVWEwRWk3RnFHWVIzeXoKUTBtMWMrWW1TOUtGU2tLWkRINnR0VnYxbDg2MGc3RUVwU3dEWE9NKzBwa0xlL1NvUlZNRzZsbzZmRVo1UXJIZQpMQzg3S0xoaDVuV216NzNQTlBEQ1h3MFVQaXJnOXlzOGUvc3NkZ3l5V0tTTkppelFYaWpSZjcrOVhpck5ncnI5CityRyt3elZyVGgxOGJhc2cyQkgrdGNwVlpsZWFvQkdUeUoxS1BVQ01pRk5GOG51a3d6SEdUc1NtSlp4Z29mbEoKZFFVZmZ5YlRkY0FOUU51VmdEcGZvK3J0QXBlb0RpYzh2dmdqNDZaNHlxM2ZYOTZCdXd5Tks2NFZuT294MGxMKwppci9kQ0tkUjZzWmU3QzNGdnJKZXpTRUtDMFgzbDdMeHRzakMwWWxrczZxallrVDVzZnhVVDFrQ0F3RUFBYU1uCk1DVXdEZ1lEVlIwUEFRSC9CQVFEQWdXZ01CTUdBMVVkSlFRTU1Bb0dDQ3NHQVFVRkJ3TUNNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFDbGNtV0tPbndoRW5DTnBQSlNFRzAwOHlHcTBtQ3Q2R3dXTWl3amlSZVYrVTRXU1JFbgp2K0thZ3VNazdveWoxSVU5RFkxa280VFJ3ZXNndHJmT1ZqTWhwUytSelErRjA5cTBxRDRPTHZ5dE5DSXMxRGVCCkdydkRJSGhVNHVxUm1DSU5rRUVTT1dxVUNncXJ3RWJQVi9KKzRTVTF1UkVvVEN2a1krb3Y3eHprcDlYVFc4blMKdCtJTFhrS25TazRaSlJxRjZzai9jSUlsYWVZamNaWGZnc0p6QkJxSjZMMzRhaHdndlIrT2FPRGd1THlvMFhjZApQakorQkxXdmZzMC83OFRHSXNyejhrTEJMU3VQSE5sRjdPZi81VjBUZWRDV2ZzcTJWSG93SENJMEF0eXpYdTY2CkhJVkY4eEhFNHNRRFdnS2ZVNGhOSHlrSUJxVUJ4NTE0VXppNgotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==' | base64 -d
```



```
-----BEGIN CERTIFICATE-----
MIIC5zCCAc+gAwIBAgIIbGaVekuGCfYwDQYJKoZIhvcNAQELBQAwFTETMBEGA1UE
AxMKa3ViZXJuZXRlczAeFw0yMDA2MjgwNTQ2MTBaFw0yMTA2MjgwNTQ2MTFaMCkx
JzAlBgNVBAMTHnN5c3RlbTprdWJlLWNvbnRyb2xsZXItbWFuYWdlcjCCASIwDQYJ
KoZIhvcNAQEBBQADggEPADCCAQoCggEBALcMj+iPwraY+QiUOiUa0Ei7FqGYR3yz
Q0m1c+YmS9KFSkKZDH6ttVv1l860g7EEpSwDXOM+0pkLe/SoRVMG6lo6fEZ5QrHe
LC87KLhh5nWmz73PNPDCXw0UPirg9ys8e/ssdgyyWKSNJizQXijRf7+9XirNgrr9
+rG+wzVrTh18basg2BH+tcpVZleaoBGTyJ1KPUCMiFNF8nukwzHGTsSmJZxgoflJ
dQUffybTdcANQNuVgDpfo+rtApeoDic8vvgj46Z4yq3fX96BuwyNK64VnOox0lL+
ir/dCKdR6sZe7C3FvrJezSEKC0X3l7LxtsjC0Ylks6qjYkT5sfxUT1kCAwEAAaMn
MCUwDgYDVR0PAQH/BAQDAgWgMBMGA1UdJQQMMAoGCCsGAQUFBwMCMA0GCSqGSIb3
DQEBCwUAA4IBAQClcmWKOnwhEnCNpPJSEG008yGq0mCt6GwWMiwjiReV+U4WSREn
v+KaguMk7oyj1IU9DY1ko4TRwesgtrfOVjMhpS+RzQ+F09q0qD4OLvytNCIs1DeB
GrvDIHhU4uqRmCINkEESOWqUCgqrwEbPV/J+4SU1uREoTCvkY+ov7xzkp9XTW8nS
t+ILXkKnSk4ZJRqF6sj/cIIlaeYjcZXfgsJzBBqJ6L34ahwgvR+OaODguLyo0Xcd
PjJ+BLWvfs0/78TGIsrz8kLBLSuPHNlF7Of/5V0TedCWfsq2VHowHCI0AtyzXu66
HIVF8xHE4sQDWgKfU4hNHykIBqUBx514Uzi6
-----END CERTIFICATE-----
```

---------------

直接将输出写入文件



```
echo 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lJYkdhVmVrdUdDZll3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURBMk1qZ3dOVFEyTVRCYUZ3MHlNVEEyTWpnd05UUTJNVEZhTUNreApKekFsQmdOVkJBTVRIbk41YzNSbGJUcHJkV0psTFdOdmJuUnliMnhzWlhJdGJXRnVZV2RsY2pDQ0FTSXdEUVlKCktvWklodmNOQVFFQkJRQURnZ0VQQURDQ0FRb0NnZ0VCQUxjTWoraVB3cmFZK1FpVU9pVWEwRWk3RnFHWVIzeXoKUTBtMWMrWW1TOUtGU2tLWkRINnR0VnYxbDg2MGc3RUVwU3dEWE9NKzBwa0xlL1NvUlZNRzZsbzZmRVo1UXJIZQpMQzg3S0xoaDVuV216NzNQTlBEQ1h3MFVQaXJnOXlzOGUvc3NkZ3l5V0tTTkppelFYaWpSZjcrOVhpck5ncnI5CityRyt3elZyVGgxOGJhc2cyQkgrdGNwVlpsZWFvQkdUeUoxS1BVQ01pRk5GOG51a3d6SEdUc1NtSlp4Z29mbEoKZFFVZmZ5YlRkY0FOUU51VmdEcGZvK3J0QXBlb0RpYzh2dmdqNDZaNHlxM2ZYOTZCdXd5Tks2NFZuT294MGxMKwppci9kQ0tkUjZzWmU3QzNGdnJKZXpTRUtDMFgzbDdMeHRzakMwWWxrczZxallrVDVzZnhVVDFrQ0F3RUFBYU1uCk1DVXdEZ1lEVlIwUEFRSC9CQVFEQWdXZ01CTUdBMVVkSlFRTU1Bb0dDQ3NHQVFVRkJ3TUNNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFDbGNtV0tPbndoRW5DTnBQSlNFRzAwOHlHcTBtQ3Q2R3dXTWl3amlSZVYrVTRXU1JFbgp2K0thZ3VNazdveWoxSVU5RFkxa280VFJ3ZXNndHJmT1ZqTWhwUytSelErRjA5cTBxRDRPTHZ5dE5DSXMxRGVCCkdydkRJSGhVNHVxUm1DSU5rRUVTT1dxVUNncXJ3RWJQVi9KKzRTVTF1UkVvVEN2a1krb3Y3eHprcDlYVFc4blMKdCtJTFhrS25TazRaSlJxRjZzai9jSUlsYWVZamNaWGZnc0p6QkJxSjZMMzRhaHdndlIrT2FPRGd1THlvMFhjZApQakorQkxXdmZzMC83OFRHSXNyejhrTEJMU3VQSE5sRjdPZi81VjBUZWRDV2ZzcTJWSG93SENJMEF0eXpYdTY2CkhJVkY4eEhFNHNRRFdnS2ZVNGhOSHlrSUJxVUJ4NTE0VXppNgotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==' | base64 -d > /opt/controller-manager.crt
```



查看证书

```
openssl x509 -in /opt/controller-manager.crt  -noout -text
```



```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 7811094956206328310 (0x6c66957a4b8609f6)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=kubernetes
        Validity
            Not Before: Jun 28 05:46:10 2020 GMT
            Not After : Jun 28 05:46:11 2021 GMT
        Subject: CN=system:kube-controller-manager
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:b7:0c:8f:e8:8f:c2:b6:98:f9:08:94:3a:25:1a:
                    d0:48:bb:16:a1:98:47:7c:b3:43:49:b5:73:e6:26:
                    4b:d2:85:4a:42:99:0c:7e:ad:b5:5b:f5:97:ce:b4:
                    83:b1:04:a5:2c:03:5c:e3:3e:d2:99:0b:7b:f4:a8:
                    45:53:06:ea:5a:3a:7c:46:79:42:b1:de:2c:2f:3b:
                    28:b8:61:e6:75:a6:cf:bd:cf:34:f0:c2:5f:0d:14:
                    3e:2a:e0:f7:2b:3c:7b:fb:2c:76:0c:b2:58:a4:8d:
                    26:2c:d0:5e:28:d1:7f:bf:bd:5e:2a:cd:82:ba:fd:
                    fa:b1:be:c3:35:6b:4e:1d:7c:6d:ab:20:d8:11:fe:
                    b5:ca:55:66:57:9a:a0:11:93:c8:9d:4a:3d:40:8c:
                    88:53:45:f2:7b:a4:c3:31:c6:4e:c4:a6:25:9c:60:
                    a1:f9:49:75:05:1f:7f:26:d3:75:c0:0d:40:db:95:
                    80:3a:5f:a3:ea:ed:02:97:a8:0e:27:3c:be:f8:23:
                    e3:a6:78:ca:ad:df:5f:de:81:bb:0c:8d:2b:ae:15:
                    9c:ea:31:d2:52:fe:8a:bf:dd:08:a7:51:ea:c6:5e:
                    ec:2d:c5:be:b2:5e:cd:21:0a:0b:45:f7:97:b2:f1:
                    b6:c8:c2:d1:89:64:b3:aa:a3:62:44:f9:b1:fc:54:
                    4f:59
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Client Authentication
    Signature Algorithm: sha256WithRSAEncryption
         a5:72:65:8a:3a:7c:21:12:70:8d:a4:f2:52:10:6d:34:f3:21:
         aa:d2:60:ad:e8:6c:16:32:2c:23:89:17:95:f9:4e:16:49:11:
         27:bf:e2:9a:82:e3:24:ee:8c:a3:d4:85:3d:0d:8d:64:a3:84:
         d1:c1:eb:20:b6:b7:ce:56:33:21:a5:2f:91:cd:0f:85:d3:da:
         b4:a8:3e:0e:2e:fc:ad:34:22:2c:d4:37:81:1a:bb:c3:20:78:
         54:e2:ea:91:98:22:0d:90:41:12:39:6a:94:0a:0a:ab:c0:46:
         cf:57:f2:7e:e1:25:35:b9:11:28:4c:2b:e4:63:ea:2f:ef:1c:
         e4:a7:d5:d3:5b:c9:d2:b7:e2:0b:5e:42:a7:4a:4e:19:25:1a:
         85:ea:c8:ff:70:82:25:69:e6:23:71:95:df:82:c2:73:04:1a:
         89:e8:bd:f8:6a:1c:20:bd:1f:8e:68:e0:e0:b8:bc:a8:d1:77:
         1d:3e:32:7e:04:b5:af:7e:cd:3f:ef:c4:c6:22:ca:f3:f2:42:
         c1:2d:2b:8f:1c:d9:45:ec:e7:ff:e5:5d:13:79:d0:96:7e:ca:
         b6:54:7a:30:1c:22:34:02:dc:b3:5e:ee:ba:1c:85:45:f3:11:
         c4:e2:c4:03:5a:02:9f:53:88:4d:1f:29:08:06:a5:01:c7:9d:
         78:53:38:ba
```



```
openssl x509 -in /opt/controller-manager.crt  -noout -subject -issuer
subject= /CN=system:kube-controller-manager
issuer= /CN=kubernetes
```

```
X509v3 Extended Key Usage: 
              TLS Web Client Authentication
```









#### 解码 client-key-data



```
echo 'LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBdHd5UDZJL0N0cGo1Q0pRNkpSclFTTHNXb1poSGZMTkRTYlZ6NWlaTDBvVktRcGtNCmZxMjFXL1dYenJTRHNRU2xMQU5jNHo3U21RdDc5S2hGVXdicVdqcDhSbmxDc2Q0c0x6c291R0htZGFiUHZjODAKOE1KZkRSUStLdUQzS3p4Nyt5eDJETEpZcEkwbUxOQmVLTkYvdjcxZUtzMkN1djM2c2I3RE5XdE9IWHh0cXlEWQpFZjYxeWxWbVY1cWdFWlBJblVvOVFJeUlVMFh5ZTZURE1jWk94S1lsbkdDaCtVbDFCUjkvSnROMXdBMUEyNVdBCk9sK2o2dTBDbDZnT0p6eSsrQ1BqcG5qS3JkOWYzb0c3REkwcnJoV2M2akhTVXY2S3Y5MElwMUhxeGw3c0xjVysKc2w3TklRb0xSZmVYc3ZHMnlNTFJpV1N6cXFOaVJQbXgvRlJQV1FJREFRQUJBb0lCQUFjVDMwU1l1bWloQlpBRgpXekl3RlRtYXNrZFJRZ0phVkJHM2lHR3Z2V0xJY0pTZW9sTUxtR1dUMjJqTXBnTGtNUmJBa29qZTF2bS83ZTBKCnpKUm5RZ3gzRW5NUElUc0xZaDM1Wlp1cmZXT3pMWGtqVitLdlFVbWFMTVV2cVo3c1djYmVjem9PYzByNWdpNWQKYUNhZjR4YWcxZEZGM1BZcDk5V0RrTHl5QjNVd1Y2NVlCRjBQbXpVNDBYejEzMWk2RlhTZG9taDhFMlJuY09vZgo3QUdWS09TWExqNzkxcUFjS1RhK0laY2JNUHFTL0NlMldDbko5TjFrQzNkbHlMWTg4NE5vSnpiaFhtQWpKd0ROClBwMEtIL0hpbkFpRFRkUDRxbEVNY0FoSmlBa3l4UFpHWU1nbTZvSzNJeE90ZDJDUVJvSGNIekZVUVYwcG1Wb2QKNERTMUsxa0NnWUVBeXA5dUNzS01weGRMMUJtYTdJam4rblgrSG0rNkhLK2x0cnFxWnVxZVRFblFta2NuRmovSQpMMzJoMDFLOHdnaDdpZkFuNkVFaDIwbGE0Z2pLWk9RcTZ4OTJwTG5RZlZrdklud3hNVkRNRWJ3dkdEbHJpS0JKClVrbktsZXNkbitvWWJsZVFqQjBMRGdQZ3BmM2lKd3lIZkEyOVZyZFh4T3lsc0VUcEpkUHFlc3NDZ1lFQTUwVWMKbXhldXBWaTNxeDN2NlFSaWNTT000Q3JqWjhCdlo5aFUzZGZvNitBT0NNbzBWTVdNYytqeU9MeGVuN3pUZ29TSQplcW1lNTV4dGVCcGJoMUNrSTJzeXZVMzgxY1FSSEF6U1VJWWl5NHcyVms2azZqTmRnRGYzUTVWMFQyOFYrNXdWClhtQUVFakhPZmtEMUFWRDdaWTFMU0pDa014ZzJCQS9uR0I3NjVlc0NnWUFEZEJZdkRzUFE4VCswbkw4Y092VWgKT3JPYkZ6Sm4zTUtKUzhNdHY5LzAwdWxBUitndG8rYW9rSTZhaUhWNUpTWGQ0djc3SVdrUFVML0F6SCtPbXFqMAptdk90dVJFSm9lU0F4UGNkclEvZFdZUy85L0tTUUpFZld1eWVBNFRjdmVPdXRjVmI3ZjdMUFZ1dDJKYnJMWFo5CnNjcEJXUnlnMlp1MVZtZFc0cmJEWXdLQmdRRE1qWlpsbnlhNzNLSm5XWTFQUHErTGZuUWwrZ2sxUlVIRVNkV1cKZWxmcitUcXdqNWlGdWswbVlFMk4zUjZjanJsTlljZ05KbVlFV1ptQmQxNnBhcXdqSDdlN05IV0M1VzUwcnVwKwppb1hRSDI0WUhHdEZNclZxcVJXczAwNFN6Q0JYY1pCODd0UHErOTYyVU9Iamppc3RnVEdyTnpQa2RXK2hYQ2Q4CmNEcGVqUUtCZ0VHcmRnRUdqMVJjYUllNWRkMG5EMFh4ZHZ3dmlkZWNhVnl3bHprbWUzY1ZzNUk5SU4rRnNaYXAKWmxRYnJGenFhUnAzUDFIcnlISnpzN3ZXU1ZtdjI2cUFReXd0SzdUOVR2aDhXcU45QTRCVjQ5d1F2VnI0Z0dTKwpUWEQ2SGQ5WGpZUUhIdHF3cTFtdE5DVFpzRlUweWkwNUh3d1V0ZzI5dTZ3RmZlYkFNMFJzCi0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==' | base64 -d
```



```
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAtwyP6I/Ctpj5CJQ6JRrQSLsWoZhHfLNDSbVz5iZL0oVKQpkM
fq21W/WXzrSDsQSlLANc4z7SmQt79KhFUwbqWjp8RnlCsd4sLzsouGHmdabPvc80
8MJfDRQ+KuD3Kzx7+yx2DLJYpI0mLNBeKNF/v71eKs2Cuv36sb7DNWtOHXxtqyDY
Ef61ylVmV5qgEZPInUo9QIyIU0Xye6TDMcZOxKYlnGCh+Ul1BR9/JtN1wA1A25WA
Ol+j6u0Cl6gOJzy++CPjpnjKrd9f3oG7DI0rrhWc6jHSUv6Kv90Ip1Hqxl7sLcW+
sl7NIQoLRfeXsvG2yMLRiWSzqqNiRPmx/FRPWQIDAQABAoIBAAcT30SYumihBZAF
WzIwFTmaskdRQgJaVBG3iGGvvWLIcJSeolMLmGWT22jMpgLkMRbAkoje1vm/7e0J
zJRnQgx3EnMPITsLYh35ZZurfWOzLXkjV+KvQUmaLMUvqZ7sWcbeczoOc0r5gi5d
aCaf4xag1dFF3PYp99WDkLyyB3UwV65YBF0PmzU40Xz131i6FXSdomh8E2RncOof
7AGVKOSXLj791qAcKTa+IZcbMPqS/Ce2WCnJ9N1kC3dlyLY884NoJzbhXmAjJwDN
Pp0KH/HinAiDTdP4qlEMcAhJiAkyxPZGYMgm6oK3IxOtd2CQRoHcHzFUQV0pmVod
4DS1K1kCgYEAyp9uCsKMpxdL1Bma7Ijn+nX+Hm+6HK+ltrqqZuqeTEnQmkcnFj/I
L32h01K8wgh7ifAn6EEh20la4gjKZOQq6x92pLnQfVkvInwxMVDMEbwvGDlriKBJ
UknKlesdn+oYbleQjB0LDgPgpf3iJwyHfA29VrdXxOylsETpJdPqessCgYEA50Uc
mxeupVi3qx3v6QRicSOM4CrjZ8BvZ9hU3dfo6+AOCMo0VMWMc+jyOLxen7zTgoSI
eqme55xteBpbh1CkI2syvU381cQRHAzSUIYiy4w2Vk6k6jNdgDf3Q5V0T28V+5wV
XmAEEjHOfkD1AVD7ZY1LSJCkMxg2BA/nGB765esCgYADdBYvDsPQ8T+0nL8cOvUh
OrObFzJn3MKJS8Mtv9/00ulAR+gto+aokI6aiHV5JSXd4v77IWkPUL/AzH+Omqj0
mvOtuREJoeSAxPcdrQ/dWYS/9/KSQJEfWuyeA4TcveOutcVb7f7LPVut2JbrLXZ9
scpBWRyg2Zu1VmdW4rbDYwKBgQDMjZZlnya73KJnWY1PPq+LfnQl+gk1RUHESdWW
elfr+Tqwj5iFuk0mYE2N3R6cjrlNYcgNJmYEWZmBd16paqwjH7e7NHWC5W50rup+
ioXQH24YHGtFMrVqqRWs004SzCBXcZB87tPq+962UOHjjistgTGrNzPkdW+hXCd8
cDpejQKBgEGrdgEGj1RcaIe5dd0nD0XxdvwvidecaVywlzkme3cVs5I9IN+FsZap
ZlQbrFzqaRp3P1HryHJzs7vWSVmv26qAQywtK7T9Tvh8WqN9A4BV49wQvVr4gGS+
TXD6Hd9XjYQHHtqwq1mtNCTZsFU0yi05HwwUtg29u6wFfebAM0Rs
-----END RSA PRIVATE KEY-----
```



------------------



直接将输出写入文件

```
echo 'LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBdHd5UDZJL0N0cGo1Q0pRNkpSclFTTHNXb1poSGZMTkRTYlZ6NWlaTDBvVktRcGtNCmZxMjFXL1dYenJTRHNRU2xMQU5jNHo3U21RdDc5S2hGVXdicVdqcDhSbmxDc2Q0c0x6c291R0htZGFiUHZjODAKOE1KZkRSUStLdUQzS3p4Nyt5eDJETEpZcEkwbUxOQmVLTkYvdjcxZUtzMkN1djM2c2I3RE5XdE9IWHh0cXlEWQpFZjYxeWxWbVY1cWdFWlBJblVvOVFJeUlVMFh5ZTZURE1jWk94S1lsbkdDaCtVbDFCUjkvSnROMXdBMUEyNVdBCk9sK2o2dTBDbDZnT0p6eSsrQ1BqcG5qS3JkOWYzb0c3REkwcnJoV2M2akhTVXY2S3Y5MElwMUhxeGw3c0xjVysKc2w3TklRb0xSZmVYc3ZHMnlNTFJpV1N6cXFOaVJQbXgvRlJQV1FJREFRQUJBb0lCQUFjVDMwU1l1bWloQlpBRgpXekl3RlRtYXNrZFJRZ0phVkJHM2lHR3Z2V0xJY0pTZW9sTUxtR1dUMjJqTXBnTGtNUmJBa29qZTF2bS83ZTBKCnpKUm5RZ3gzRW5NUElUc0xZaDM1Wlp1cmZXT3pMWGtqVitLdlFVbWFMTVV2cVo3c1djYmVjem9PYzByNWdpNWQKYUNhZjR4YWcxZEZGM1BZcDk5V0RrTHl5QjNVd1Y2NVlCRjBQbXpVNDBYejEzMWk2RlhTZG9taDhFMlJuY09vZgo3QUdWS09TWExqNzkxcUFjS1RhK0laY2JNUHFTL0NlMldDbko5TjFrQzNkbHlMWTg4NE5vSnpiaFhtQWpKd0ROClBwMEtIL0hpbkFpRFRkUDRxbEVNY0FoSmlBa3l4UFpHWU1nbTZvSzNJeE90ZDJDUVJvSGNIekZVUVYwcG1Wb2QKNERTMUsxa0NnWUVBeXA5dUNzS01weGRMMUJtYTdJam4rblgrSG0rNkhLK2x0cnFxWnVxZVRFblFta2NuRmovSQpMMzJoMDFLOHdnaDdpZkFuNkVFaDIwbGE0Z2pLWk9RcTZ4OTJwTG5RZlZrdklud3hNVkRNRWJ3dkdEbHJpS0JKClVrbktsZXNkbitvWWJsZVFqQjBMRGdQZ3BmM2lKd3lIZkEyOVZyZFh4T3lsc0VUcEpkUHFlc3NDZ1lFQTUwVWMKbXhldXBWaTNxeDN2NlFSaWNTT000Q3JqWjhCdlo5aFUzZGZvNitBT0NNbzBWTVdNYytqeU9MeGVuN3pUZ29TSQplcW1lNTV4dGVCcGJoMUNrSTJzeXZVMzgxY1FSSEF6U1VJWWl5NHcyVms2azZqTmRnRGYzUTVWMFQyOFYrNXdWClhtQUVFakhPZmtEMUFWRDdaWTFMU0pDa014ZzJCQS9uR0I3NjVlc0NnWUFEZEJZdkRzUFE4VCswbkw4Y092VWgKT3JPYkZ6Sm4zTUtKUzhNdHY5LzAwdWxBUitndG8rYW9rSTZhaUhWNUpTWGQ0djc3SVdrUFVML0F6SCtPbXFqMAptdk90dVJFSm9lU0F4UGNkclEvZFdZUy85L0tTUUpFZld1eWVBNFRjdmVPdXRjVmI3ZjdMUFZ1dDJKYnJMWFo5CnNjcEJXUnlnMlp1MVZtZFc0cmJEWXdLQmdRRE1qWlpsbnlhNzNLSm5XWTFQUHErTGZuUWwrZ2sxUlVIRVNkV1cKZWxmcitUcXdqNWlGdWswbVlFMk4zUjZjanJsTlljZ05KbVlFV1ptQmQxNnBhcXdqSDdlN05IV0M1VzUwcnVwKwppb1hRSDI0WUhHdEZNclZxcVJXczAwNFN6Q0JYY1pCODd0UHErOTYyVU9Iamppc3RnVEdyTnpQa2RXK2hYQ2Q4CmNEcGVqUUtCZ0VHcmRnRUdqMVJjYUllNWRkMG5EMFh4ZHZ3dmlkZWNhVnl3bHprbWUzY1ZzNUk5SU4rRnNaYXAKWmxRYnJGenFhUnAzUDFIcnlISnpzN3ZXU1ZtdjI2cUFReXd0SzdUOVR2aDhXcU45QTRCVjQ5d1F2VnI0Z0dTKwpUWEQ2SGQ5WGpZUUhIdHF3cTFtdE5DVFpzRlUweWkwNUh3d1V0ZzI5dTZ3RmZlYkFNMFJzCi0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==' | base64 -d > /opt/controller-manager.key
```





# 查看权限相关



> https://v1-17.docs.kubernetes.io/docs/setup/best-practices/certificates/#configure-certificates-for-user-accounts



cn

system:kube-controller-manager



###  关联的clusterrolebinding

```
kubectl get clusterrolebinding -o custom-columns='Name:metadata.name,Sub:subjects'  | grep 'system:kube-controller-manage'
```



```
system:kube-controller-manager                         [map[apiGroup:rbac.authorization.k8s.io kind:User name:system:kube-controller-manager]]
```

--------------



```
kubectl  get clusterrolebinding  system:kube-controller-manager -o yaml
```



```
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-controller-manager
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: system:kube-controller-manager
```





### 关联的 rrolebinding

```
kubectl get rolebinding -o custom-columns='Name:metadata.name,Sub:subjects'  -A | grep 'system:kube-controller-manage'
```

```
system::extension-apiserver-authentication-reader   [map[apiGroup:rbac.authorization.k8s.io kind:User namesystem:kube-controller-manager] map[apiGroup:rbac.authorization.k8s.io kind:User name:system:kube-scheduler]]
system::leader-locking-kube-controller-manager      [map[apiGroup:rbac.authorization.k8s.io kind:User namesystem:kube-controller-manager] map[kind:ServiceAccount name:kube-controller-manager namespace:kube-system]]
```



```
kubectl  get rolebindings system::extension-apiserver-authentication-reader -n kube-system  -o yaml
```



```
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: system:kube-controller-manager
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: system:kube-scheduler
```

---------------



```
kubectl  get rolebindings system::leader-locking-kube-controller-manager -n kube-system  -o yaml
```



```
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: system::leader-locking-kube-controller-manager
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: system:kube-controller-manager
- kind: ServiceAccount
  name: kube-controller-manager
  namespace: kube-system
```

# 手动创建 客户端证书



```
rm -rf  /kube-controller-manager-certs
mkdir  /kube-controller-manager-certs
cd  /kube-controller-manager-certs
```



```
openssl genrsa -out kube-controller-manager.key 2048
```



```
openssl req -new -key kube-controller-manager.key  -subj "/CN=system:kube-controller-manager" -out kube-controller-manager.csr
```



```
cat > kube-controller-manager-csr.conf  << EOF
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
openssl x509 -req -in kube-controller-manager.csr  \
-CA    /etc/kubernetes/pki/ca.crt \
-CAkey /etc/kubernetes/pki/ca.key \
-CAcreateserial -out kube-controller-manager.crt -days 10000 \
-extensions v3_ext -extfile kube-controller-manager-csr.conf
```



------------



```
openssl x509 -in /kube-controller-manager-certs/kube-controller-manager.crt  -noout -text
```

```
Issuer: CN=kubernetes
Subject: CN=system:kube-controller-manager
```



# 创建kubeconfig



```
rm  -rf /etc/kubernetes/controller-manager.conf
```



```
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.crt \
  --embed-certs=true \
  --server=https://192.168.3.101:6443 \
  --kubeconfig=/etc/kubernetes/controller-manager.conf
```





```
kubectl config set-credentials system:kube-controller-manager \
  --client-certificate=/kube-controller-manager-certs/kube-controller-manager.crt \
  --client-key=/kube-controller-manager-certs/kube-controller-manager.key \
  --embed-certs=true \
  --kubeconfig=/etc/kubernetes/controller-manager.conf
```



```
kubectl config set-context system:kube-controller-manager@kubernetes \
  --cluster=kubernetes \
  --user=system:kube-controller-manager \
  --kubeconfig=/etc/kubernetes/controller-manager.conf
```



```
kubectl config use-context system:kube-controller-manager@kubernetes \
--kubeconfig=/etc/kubernetes/controller-manager.conf
```



------------







# 验证kubeconfig 



验证kubeconfig



```
kubectl  get all  --kubeconfig=/etc/kubernetes/controller-manager.conf
```



# systemd



```
cat > /usr/lib/systemd/system/kube-controller-manager.service << EOF
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
--allocate-node-cidrs=true \\
--authentication-kubeconfig=/etc/kubernetes/controller-manager.conf \\
--authorization-kubeconfig=/etc/kubernetes/controller-manager.conf \\
--bind-address=127.0.0.1 \\
--client-ca-file=/etc/kubernetes/pki/ca.crt \\
--cluster-cidr=10.244.0.0/16 \\
--cluster-name=kubernetes \\
--cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt \\
--cluster-signing-key-file=/etc/kubernetes/pki/ca.key \\
--controllers=*,bootstrapsigner,tokencleaner \\
--kubeconfig=/etc/kubernetes/controller-manager.conf \\
--leader-elect=true \\
--node-cidr-mask-size=24 \\
--requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt \\
--root-ca-file=/etc/kubernetes/pki/ca.crt \\
--service-account-private-key-file=/etc/kubernetes/pki/sa.key \\
--service-cluster-ip-range=10.96.0.0/12 \\
--use-service-account-credentials=true
Restart=always
RestartSec=10s
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

----------



```
cat /usr/lib/systemd/system/kube-controller-manager.service
```



```
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \
--allocate-node-cidrs=true \
--authentication-kubeconfig=/etc/kubernetes/controller-manager.conf \
--authorization-kubeconfig=/etc/kubernetes/controller-manager.conf \
--bind-address=127.0.0.1 \
--client-ca-file=/etc/kubernetes/pki/ca.crt \
--cluster-cidr=10.244.0.0/16 \
--cluster-name=kubernetes \
--cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt \
--cluster-signing-key-file=/etc/kubernetes/pki/ca.key \
--controllers=*,bootstrapsigner,tokencleaner \
--kubeconfig=/etc/kubernetes/controller-manager.conf \
--leader-elect=true \
--node-cidr-mask-size=24 \
--requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt \
--root-ca-file=/etc/kubernetes/pki/ca.crt \
--service-account-private-key-file=/etc/kubernetes/pki/sa.key \
--service-cluster-ip-range=10.96.0.0/12 \
--use-service-account-credentials=true
Restart=always
RestartSec=10s
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```



# 删除static pod

```
mv /etc/kubernetes/manifests/kube-controller-manager.yaml  /opt/
```



```
kubectl  get componentstatuses  controller-manager
```



```
NAME                 STATUS      MESSAGE                                                                                     ERROR
controller-manager   Unhealthy   Get http://127.0.0.1:10252/healthz: dial tcp 127.0.0.1:10252: connect: connection refused   
```





#  启动服务

```
systemctl  start kube-controller-manager
```

```
systemctl  status kube-controller-manager
```



----------------

```
kubectl  get componentstatuses  controller-manager 
```



```
NAME                 STATUS    MESSAGE   ERROR
controller-manager   Healthy   ok    
```

---------------



```
systemctl  enable  kube-controller-manager
```



# 重启测试

```
reboot
```

