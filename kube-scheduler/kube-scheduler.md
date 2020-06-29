# 获取程序



```
docker run -it --name=scheduler  registry.aliyuncs.com/google_containers/kube-scheduler:v1.18.3 /bin/sh
```



```
docker cp scheduler:/usr/local/bin/kube-scheduler /opt
```

```
docker stop scheduler
docker rm   scheduler
```



```
cp /opt/kube-scheduler  /usr/local/bin/
```



```
kube-scheduler --version
```



```
Kubernetes v1.18.3
```

# 查看当前启动参数

```
ps -ef | grep kube-scheduler
```

```
kube-scheduler --authentication-kubeconfig=/etc/kubernetes/scheduler.conf --authorization-kubeconfig=/etc/kubernetes/scheduler.conf --bind-address=127.0.0.1 --kubeconfig=/etc/kubernetes/scheduler.conf --leader-elect=true
```

------------



```
cat /etc/kubernetes/manifests/kube-scheduler.yaml
```

```
    - kube-scheduler
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=true
```





# 查看kubeconfig 里面的证书



```
cat /etc/kubernetes/scheduler.conf
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
    user: system:kube-scheduler
  name: system:kube-scheduler@kubernetes
current-context: system:kube-scheduler@kubernetes
kind: Config
preferences: {}
users:
- name: system:kube-scheduler
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMzakNDQWNhZ0F3SUJBZ0lJYi9lZkNZc1dtbGt3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURBMk1qZ3dOVFEyTVRCYUZ3MHlNVEEyTWpnd05UUTJNVEphTUNBeApIakFjQmdOVkJBTVRGWE41YzNSbGJUcHJkV0psTFhOamFHVmtkV3hsY2pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCCkJRQURnZ0VQQURDQ0FRb0NnZ0VCQUxLU2FiYUNMUkZhc21TREwwb3lSSXhEZzUvM2lGZkJuTVFnQUF6ejVjOHIKTEFIMUwyWnFxbTA2V1NUbFB4VjJQcU5ZY3hlMkc4MDR3cXFsTUpuWFI2ZkJlalp4N0s0RC9saVZybkNFV292dwpad2VJY3A4Rk4ra0l2cHRrMm1LRmVsa0pudWZQejdjdFFxbEx0ZE1TZGNPZlp5WFkycjNhZVZuS3NBVHJXUFRlCit5UDBXMFpVL1RuRXpPVUt4aHBXb3lKN2QxcXA1WHJqNTNRMFhXOVE5dVRBS1JMcStxT0k2UDlpR2dNa1BFQmEKcHNGTTdUcmZBcnpXV1RVUzVYT3lZVTBmeTk3NlZmVHF4N2FBNEdxSTBxN09QNVc2d3NGb2pDamVScTBzd282awpyeW5nVGk2SGJVRnl1WGRHS0NoVTVBUWM4ci8vbEcyTFczcE4xVE5KZTQ4Q0F3RUFBYU1uTUNVd0RnWURWUjBQCkFRSC9CQVFEQWdXZ01CTUdBMVVkSlFRTU1Bb0dDQ3NHQVFVRkJ3TUNNQTBHQ1NxR1NJYjNEUUVCQ3dVQUE0SUIKQVFBbys3c0RqT1c2SklmcGRqak9BbCtFZXlNbUFQMjZlYTBNc3MxTGlYZGRyb3drRGdpOWNXWlNSbGt6RkFRbwprbU9CdDQwc1dHd3ZWaXBXQ1VjbEM5Tk11Zlk4RUJ5Q2Nzd2s3ZzBzNVFGRkQxcmVkZEF0OEdQZFZvRkNnb0xMClNhZGhsWXc4emRZeGg0dEZIZnh3VHU2MkhyWTV1Z3IyeklqeW8vVTAvWTBHek9GaGxmY21tVk9qZkZUdm5qM3IKMVRRZi96cVpzZUg1UjFKYW52a3BOM0hiTW1wVTJwelBHNVBZOTVhZGh6Skw3eDFWRG80TFZ5ejJDNHRRaHVKOQpxVnZTOW81K3pVRnplVGIxbXd3NGV4OFhlYXFyWk1wTTd1NEZvN2hiVmp6NTFVMzl5eGd6UzhPR3dkTURscnNtCkR3cTQvOWFFMXN6cXBWQXdqMTFCS2VrYwotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcFFJQkFBS0NBUUVBc3BKcHRvSXRFVnF5WklNdlNqSkVqRU9Ebi9lSVY4R2N4Q0FBRFBQbHp5c3NBZlV2ClptcXFiVHBaSk9VL0ZYWStvMWh6RjdZYnpUakNxcVV3bWRkSHA4RjZObkhzcmdQK1dKV3VjSVJhaS9CbkI0aHkKbndVMzZRaSttMlRhWW9WNldRbWU1OC9QdHkxQ3FVdTEweEoxdzU5bkpkamF2ZHA1V2Nxd0JPdFk5Tjc3SS9SYgpSbFQ5T2NUTTVRckdHbGFqSW50M1dxbmxldVBuZERSZGIxRDI1TUFwRXVyNm80am8vMklhQXlROFFGcW13VXp0Ck90OEN2TlpaTlJMbGM3SmhUUi9MM3ZwVjlPckh0b0RnYW9qU3JzNC9sYnJDd1dpTUtONUdyU3pDanFTdktlQk8KTG9kdFFYSzVkMFlvS0ZUa0JCenl2LytVYll0YmVrM1ZNMGw3andJREFRQUJBb0lCQVFDTk1rLzhTN291K3JRTAozZDdLb0N5cmE4YnIrZUlJNGNKL0lYNW92NEY2NmZ3R0lFUzJpcWp2YlMrSGlPejBuMmF2NmdRM1AzdUVMZGxlCjdQY2M3YWh1OFFFZGc3OU5hVUwzOElMWjNRMXJrVThtR2JIb0kwd3VLd2ZyL3piZXFBUXgydldXL2k2VC9HbTEKUzNRZHpYN29pMllYK3Z3YkdtRFJxdjY3SlF4VnNaZittR3Z1L2Z6NEE2ZjlVOVFxOVhhUXZYMU9vTXhTSHpJQgpNR3l0V3ZtZFdCSG9aNXlNN0RpNG5vVE8wTWgvdCtsT0ZQNzRySU1YQTZMZTlKZEZUcWtXczd6QUdQcHZKRU1NCndwMi95L0hSckNqcnJJMHBFWk5LRjZ3MFZjTU5YUHdFRnRQTEh6aVBxS3lOVG1tK1ZSSGRVVzVKTk51TmcrYkwKN2VPUDhKMWhBb0dCQU10YkRwaUpVeWN6VitodVJnZG9FblVEQzh4Z0xwVWcvRU83SW5TRGRkd0JZcWJsZUlWbwpFRFJrRlNtcVpRcjRzY0pwRHoxSDl3NFlzZG4xM3Y3NXdFS2lObk9PbXV2SmdzWHRGT2NaQlhCK21GSDhJSWtECjZmeUc1VUZ6MnFBSjRCclBsdzhPUVE0ZE9tWjJ3T1pJeTdtcXk3WTg3eE9rK3lDTGt0ekJyb0JGQW9HQkFPRE0KM2dxaGM0VUVIRHFzMEtJcStmaWVvcmd0a283Q3h2NkZnSlNTNVJiYStuaTF0WjZ3Q1Q4NVQ1bDQxSk8wbmthQgpqSnM3TzVxWURpaDhESEpwamZyK0tNZVJURUVNQTFDNXpYWDZFRThYYThOa1Z3NEZnamNNRGFpMFNESy9HZkJmCmZTaVcyQkdsUUZpVm85VHl6dkNFYXo1UklpVkhsb00zMG1PeUpKdkRBb0dBWXdYM0tJNE9ZTk5lcGo4MGVKelUKQ0Fpd3NSZlE5eXQxeStHUFdKOC9RQitvazA3QWptM3JIaWZ5S2pUZ09TUjdJd2tYczZhY2hrKytJejNZRmQ4MgpJUHh1ZVh6aXNaaVJ4cUc1QVFPdEkyZHg4dEpNWVl2M1g5R3NSMkFNQU14dVJYLzZ0Z2toNHFhVzdwZzdQS2dNCkZHQTRESWpGZnBKaSt5a2NIY1Z1bk9VQ2dZRUF1cVMrRWx0OE8wdHZXTFFWUVIrbmplbkFObVQ0RXZuYkdJV2wKZlRYOWFSMkU2bVlNRm1ZWU4xc1JJTjUydVBBMG5WdUFiMzRkZmJ5VHZMOUo0bENMWm9KUlAyait3OThDZmFyVwowUVkvTmp1KzZHck44TUZZSFBZdi9RczZDcEFxTEM1TUQwQTJ3MmZONWY3UUdNVkVWZVBMMnVDb0ZnVzdETlZ6CkkvMUxjZHNDZ1lFQW8vUm0xZkNiNEVWQU1INzVLcXY4d2QrckNTaWhJeGVmQjd3SFMzVXZLMlBML3luV2tzaVcKU2hvYXJGZ1VoUmRET3JpKzNucWJLUkUveDdZQU81SnRlM0xFZnRvMWRXRkZpekFZWkV0VzN0LzZHL1Zja0tzbgplRHJUTzJkRERQbGw1T3ZheEt2K3BCMHRiQjE0ZHZmZjBxd2tDQ1NRbVlWRndhM3diMzU0YW1jPQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
```



### 解码certificate-authority-data



```
echo 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01EWXlPREExTkRZeE1Gb1hEVE13TURZeU5qQTFORFl4TUZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTG5jClVIZmFiQzljcFk4MXNKdmwvTUlnU1ZWNHRMUVNoVjI0U280MWNuVy9HMWFWTUtYNk5kR0wwWDl2YTBsS1I4SFMKV25hcVo4K3BnN3hmYVgvVVBMbzU4dlptVDhvZC9DZ0pQaG9iczA5SFc0TklNNmlGOFVkMkVUbkNiUDJkTkNMRApZd29FYUgwTEdhVHE4ejh1ZzE5Y1pXUHI1UGtQdkZxV3JneGQwUWxyRzhyd2R2SkpLVWlFL3hqOS81VkNNTG9xCmVndVo5YjlXWmZEMktMTlhFamFEb25pbzRhMmdxaVBFdEJpaFVhOFdZWU5URHptWVFtRVI3VjRORkx3bW5KV3YKbjNRUGs2aXBOOEI1VmFlbCtaZ1B0bmQwc3VmajBKSG1rVk1Ra2pVdnpvZjRKUEIvVUFXUTRrLzNIaWhLRE9VVQpSeXYvREU5M0h5ZCtuVW9iTG1FQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFJdXlzU3NmM1E1OW9SZ0NyakQvQmV5N0V1KzYKZ3kxZ0FQNzI3OGxBRUV3K0pTclhrenN6MUk4QXBwbmJxb0tKbjlPNmw1UGxLRlA1NWFkS3JZT2Q0TWJKcWtXcQpIT1h3YUVOWVdESldHTjhONVJydXJMMXA5ckQ2K0NPa3ZMSWNMb21vTW0ya2JpRldJMFJ0MVdMaitiK2ZWK2d0ClpqT1orVFE1TmFmeUVRM3BTVUROWFNKYlQ0Y2lxSXR2Z05aNzVvTmYwWmozNDBVeE9jSVhtYityODc2ZDlJMFAKNHNQTkRDVGJSeUY1YXd1VWhXRmtBOXhCREdFS1hOTHVkV29sREowOVJWTU8xQ1A5eStheDJiQnk5TTlNWWtNTQpQa05NanFhYzlwcVB0dVVJUlJkdkZ4QlZZVVZJQnVxV2JTdnNwcWw0L2F0OXN1c1NxV2J0QU5nVGlFST0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=' | base64 -d
```



查看证书

```
echo 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01EWXlPREExTkRZeE1Gb1hEVE13TURZeU5qQTFORFl4TUZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTG5jClVIZmFiQzljcFk4MXNKdmwvTUlnU1ZWNHRMUVNoVjI0U280MWNuVy9HMWFWTUtYNk5kR0wwWDl2YTBsS1I4SFMKV25hcVo4K3BnN3hmYVgvVVBMbzU4dlptVDhvZC9DZ0pQaG9iczA5SFc0TklNNmlGOFVkMkVUbkNiUDJkTkNMRApZd29FYUgwTEdhVHE4ejh1ZzE5Y1pXUHI1UGtQdkZxV3JneGQwUWxyRzhyd2R2SkpLVWlFL3hqOS81VkNNTG9xCmVndVo5YjlXWmZEMktMTlhFamFEb25pbzRhMmdxaVBFdEJpaFVhOFdZWU5URHptWVFtRVI3VjRORkx3bW5KV3YKbjNRUGs2aXBOOEI1VmFlbCtaZ1B0bmQwc3VmajBKSG1rVk1Ra2pVdnpvZjRKUEIvVUFXUTRrLzNIaWhLRE9VVQpSeXYvREU5M0h5ZCtuVW9iTG1FQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFJdXlzU3NmM1E1OW9SZ0NyakQvQmV5N0V1KzYKZ3kxZ0FQNzI3OGxBRUV3K0pTclhrenN6MUk4QXBwbmJxb0tKbjlPNmw1UGxLRlA1NWFkS3JZT2Q0TWJKcWtXcQpIT1h3YUVOWVdESldHTjhONVJydXJMMXA5ckQ2K0NPa3ZMSWNMb21vTW0ya2JpRldJMFJ0MVdMaitiK2ZWK2d0ClpqT1orVFE1TmFmeUVRM3BTVUROWFNKYlQ0Y2lxSXR2Z05aNzVvTmYwWmozNDBVeE9jSVhtYityODc2ZDlJMFAKNHNQTkRDVGJSeUY1YXd1VWhXRmtBOXhCREdFS1hOTHVkV29sREowOVJWTU8xQ1A5eStheDJiQnk5TTlNWWtNTQpQa05NanFhYzlwcVB0dVVJUlJkdkZ4QlZZVVZJQnVxV2JTdnNwcWw0L2F0OXN1c1NxV2J0QU5nVGlFST0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=' | base64 -d > /opt/scheduler-ca.crt
```



```
openssl x509 -in /opt/scheduler-ca.crt  -noout -text
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



kubeadm 创建 scheduler 的 kubeconfig  时 将 /etc/kubernetes/pki/ca.crt  写入

```
diff /etc/kubernetes/pki/ca.crt  /opt/scheduler-ca.crt
```



### 解码client-certificate-data



```
echo 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMzakNDQWNhZ0F3SUJBZ0lJYi9lZkNZc1dtbGt3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURBMk1qZ3dOVFEyTVRCYUZ3MHlNVEEyTWpnd05UUTJNVEphTUNBeApIakFjQmdOVkJBTVRGWE41YzNSbGJUcHJkV0psTFhOamFHVmtkV3hsY2pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCCkJRQURnZ0VQQURDQ0FRb0NnZ0VCQUxLU2FiYUNMUkZhc21TREwwb3lSSXhEZzUvM2lGZkJuTVFnQUF6ejVjOHIKTEFIMUwyWnFxbTA2V1NUbFB4VjJQcU5ZY3hlMkc4MDR3cXFsTUpuWFI2ZkJlalp4N0s0RC9saVZybkNFV292dwpad2VJY3A4Rk4ra0l2cHRrMm1LRmVsa0pudWZQejdjdFFxbEx0ZE1TZGNPZlp5WFkycjNhZVZuS3NBVHJXUFRlCit5UDBXMFpVL1RuRXpPVUt4aHBXb3lKN2QxcXA1WHJqNTNRMFhXOVE5dVRBS1JMcStxT0k2UDlpR2dNa1BFQmEKcHNGTTdUcmZBcnpXV1RVUzVYT3lZVTBmeTk3NlZmVHF4N2FBNEdxSTBxN09QNVc2d3NGb2pDamVScTBzd282awpyeW5nVGk2SGJVRnl1WGRHS0NoVTVBUWM4ci8vbEcyTFczcE4xVE5KZTQ4Q0F3RUFBYU1uTUNVd0RnWURWUjBQCkFRSC9CQVFEQWdXZ01CTUdBMVVkSlFRTU1Bb0dDQ3NHQVFVRkJ3TUNNQTBHQ1NxR1NJYjNEUUVCQ3dVQUE0SUIKQVFBbys3c0RqT1c2SklmcGRqak9BbCtFZXlNbUFQMjZlYTBNc3MxTGlYZGRyb3drRGdpOWNXWlNSbGt6RkFRbwprbU9CdDQwc1dHd3ZWaXBXQ1VjbEM5Tk11Zlk4RUJ5Q2Nzd2s3ZzBzNVFGRkQxcmVkZEF0OEdQZFZvRkNnb0xMClNhZGhsWXc4emRZeGg0dEZIZnh3VHU2MkhyWTV1Z3IyeklqeW8vVTAvWTBHek9GaGxmY21tVk9qZkZUdm5qM3IKMVRRZi96cVpzZUg1UjFKYW52a3BOM0hiTW1wVTJwelBHNVBZOTVhZGh6Skw3eDFWRG80TFZ5ejJDNHRRaHVKOQpxVnZTOW81K3pVRnplVGIxbXd3NGV4OFhlYXFyWk1wTTd1NEZvN2hiVmp6NTFVMzl5eGd6UzhPR3dkTURscnNtCkR3cTQvOWFFMXN6cXBWQXdqMTFCS2VrYwotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==' | base64 -d
```



查看证书

```
echo 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMzakNDQWNhZ0F3SUJBZ0lJYi9lZkNZc1dtbGt3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURBMk1qZ3dOVFEyTVRCYUZ3MHlNVEEyTWpnd05UUTJNVEphTUNBeApIakFjQmdOVkJBTVRGWE41YzNSbGJUcHJkV0psTFhOamFHVmtkV3hsY2pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCCkJRQURnZ0VQQURDQ0FRb0NnZ0VCQUxLU2FiYUNMUkZhc21TREwwb3lSSXhEZzUvM2lGZkJuTVFnQUF6ejVjOHIKTEFIMUwyWnFxbTA2V1NUbFB4VjJQcU5ZY3hlMkc4MDR3cXFsTUpuWFI2ZkJlalp4N0s0RC9saVZybkNFV292dwpad2VJY3A4Rk4ra0l2cHRrMm1LRmVsa0pudWZQejdjdFFxbEx0ZE1TZGNPZlp5WFkycjNhZVZuS3NBVHJXUFRlCit5UDBXMFpVL1RuRXpPVUt4aHBXb3lKN2QxcXA1WHJqNTNRMFhXOVE5dVRBS1JMcStxT0k2UDlpR2dNa1BFQmEKcHNGTTdUcmZBcnpXV1RVUzVYT3lZVTBmeTk3NlZmVHF4N2FBNEdxSTBxN09QNVc2d3NGb2pDamVScTBzd282awpyeW5nVGk2SGJVRnl1WGRHS0NoVTVBUWM4ci8vbEcyTFczcE4xVE5KZTQ4Q0F3RUFBYU1uTUNVd0RnWURWUjBQCkFRSC9CQVFEQWdXZ01CTUdBMVVkSlFRTU1Bb0dDQ3NHQVFVRkJ3TUNNQTBHQ1NxR1NJYjNEUUVCQ3dVQUE0SUIKQVFBbys3c0RqT1c2SklmcGRqak9BbCtFZXlNbUFQMjZlYTBNc3MxTGlYZGRyb3drRGdpOWNXWlNSbGt6RkFRbwprbU9CdDQwc1dHd3ZWaXBXQ1VjbEM5Tk11Zlk4RUJ5Q2Nzd2s3ZzBzNVFGRkQxcmVkZEF0OEdQZFZvRkNnb0xMClNhZGhsWXc4emRZeGg0dEZIZnh3VHU2MkhyWTV1Z3IyeklqeW8vVTAvWTBHek9GaGxmY21tVk9qZkZUdm5qM3IKMVRRZi96cVpzZUg1UjFKYW52a3BOM0hiTW1wVTJwelBHNVBZOTVhZGh6Skw3eDFWRG80TFZ5ejJDNHRRaHVKOQpxVnZTOW81K3pVRnplVGIxbXd3NGV4OFhlYXFyWk1wTTd1NEZvN2hiVmp6NTFVMzl5eGd6UzhPR3dkTURscnNtCkR3cTQvOWFFMXN6cXBWQXdqMTFCS2VrYwotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==' | base64 -d
```



```
echo 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMzakNDQWNhZ0F3SUJBZ0lJYi9lZkNZc1dtbGt3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURBMk1qZ3dOVFEyTVRCYUZ3MHlNVEEyTWpnd05UUTJNVEphTUNBeApIakFjQmdOVkJBTVRGWE41YzNSbGJUcHJkV0psTFhOamFHVmtkV3hsY2pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCCkJRQURnZ0VQQURDQ0FRb0NnZ0VCQUxLU2FiYUNMUkZhc21TREwwb3lSSXhEZzUvM2lGZkJuTVFnQUF6ejVjOHIKTEFIMUwyWnFxbTA2V1NUbFB4VjJQcU5ZY3hlMkc4MDR3cXFsTUpuWFI2ZkJlalp4N0s0RC9saVZybkNFV292dwpad2VJY3A4Rk4ra0l2cHRrMm1LRmVsa0pudWZQejdjdFFxbEx0ZE1TZGNPZlp5WFkycjNhZVZuS3NBVHJXUFRlCit5UDBXMFpVL1RuRXpPVUt4aHBXb3lKN2QxcXA1WHJqNTNRMFhXOVE5dVRBS1JMcStxT0k2UDlpR2dNa1BFQmEKcHNGTTdUcmZBcnpXV1RVUzVYT3lZVTBmeTk3NlZmVHF4N2FBNEdxSTBxN09QNVc2d3NGb2pDamVScTBzd282awpyeW5nVGk2SGJVRnl1WGRHS0NoVTVBUWM4ci8vbEcyTFczcE4xVE5KZTQ4Q0F3RUFBYU1uTUNVd0RnWURWUjBQCkFRSC9CQVFEQWdXZ01CTUdBMVVkSlFRTU1Bb0dDQ3NHQVFVRkJ3TUNNQTBHQ1NxR1NJYjNEUUVCQ3dVQUE0SUIKQVFBbys3c0RqT1c2SklmcGRqak9BbCtFZXlNbUFQMjZlYTBNc3MxTGlYZGRyb3drRGdpOWNXWlNSbGt6RkFRbwprbU9CdDQwc1dHd3ZWaXBXQ1VjbEM5Tk11Zlk4RUJ5Q2Nzd2s3ZzBzNVFGRkQxcmVkZEF0OEdQZFZvRkNnb0xMClNhZGhsWXc4emRZeGg0dEZIZnh3VHU2MkhyWTV1Z3IyeklqeW8vVTAvWTBHek9GaGxmY21tVk9qZkZUdm5qM3IKMVRRZi96cVpzZUg1UjFKYW52a3BOM0hiTW1wVTJwelBHNVBZOTVhZGh6Skw3eDFWRG80TFZ5ejJDNHRRaHVKOQpxVnZTOW81K3pVRnplVGIxbXd3NGV4OFhlYXFyWk1wTTd1NEZvN2hiVmp6NTFVMzl5eGd6UzhPR3dkTURscnNtCkR3cTQvOWFFMXN6cXBWQXdqMTFCS2VrYwotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==' | base64 -d > /opt/scheduler-client.crt
```



```
openssl x509 -in /opt/scheduler-client.crt   -noout -text
```



```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 8068092120794569305 (0x6ff79f098b169a59)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=kubernetes
        Validity
            Not Before: Jun 28 05:46:10 2020 GMT
            Not After : Jun 28 05:46:12 2021 GMT
        Subject: CN=system:kube-scheduler
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:b2:92:69:b6:82:2d:11:5a:b2:64:83:2f:4a:32:
                    44:8c:43:83:9f:f7:88:57:c1:9c:c4:20:00:0c:f3:
                    e5:cf:2b:2c:01:f5:2f:66:6a:aa:6d:3a:59:24:e5:
                    3f:15:76:3e:a3:58:73:17:b6:1b:cd:38:c2:aa:a5:
                    30:99:d7:47:a7:c1:7a:36:71:ec:ae:03:fe:58:95:
                    ae:70:84:5a:8b:f0:67:07:88:72:9f:05:37:e9:08:
                    be:9b:64:da:62:85:7a:59:09:9e:e7:cf:cf:b7:2d:
                    42:a9:4b:b5:d3:12:75:c3:9f:67:25:d8:da:bd:da:
                    79:59:ca:b0:04:eb:58:f4:de:fb:23:f4:5b:46:54:
                    fd:39:c4:cc:e5:0a:c6:1a:56:a3:22:7b:77:5a:a9:
                    e5:7a:e3:e7:74:34:5d:6f:50:f6:e4:c0:29:12:ea:
                    fa:a3:88:e8:ff:62:1a:03:24:3c:40:5a:a6:c1:4c:
                    ed:3a:df:02:bc:d6:59:35:12:e5:73:b2:61:4d:1f:
                    cb:de:fa:55:f4:ea:c7:b6:80:e0:6a:88:d2:ae:ce:
                    3f:95:ba:c2:c1:68:8c:28:de:46:ad:2c:c2:8e:a4:
                    af:29:e0:4e:2e:87:6d:41:72:b9:77:46:28:28:54:
                    e4:04:1c:f2:bf:ff:94:6d:8b:5b:7a:4d:d5:33:49:
                    7b:8f
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Client Authentication
    Signature Algorithm: sha256WithRSAEncryption
         28:fb:bb:03:8c:e5:ba:24:87:e9:76:38:ce:02:5f:84:7b:23:
         26:00:fd:ba:79:ad:0c:b2:cd:4b:89:77:5d:ae:8c:24:0e:08:
         bd:71:66:52:46:59:33:14:04:28:92:63:81:b7:8d:2c:58:6c:
         2f:56:2a:56:09:47:25:0b:d3:4c:b9:f6:3c:10:1c:82:72:cc:
         24:ee:0d:2c:e5:01:45:0f:5a:de:75:d0:2d:f0:63:dd:56:81:
         42:82:82:cb:49:a7:61:95:8c:3c:cd:d6:31:87:8b:45:1d:fc:
         70:4e:ee:b6:1e:b6:39:ba:0a:f6:cc:88:f2:a3:f5:34:fd:8d:
         06:cc:e1:61:95:f7:26:99:53:a3:7c:54:ef:9e:3d:eb:d5:34:
         1f:ff:3a:99:b1:e1:f9:47:52:5a:9e:f9:29:37:71:db:32:6a:
         54:da:9c:cf:1b:93:d8:f7:96:9d:87:32:4b:ef:1d:55:0e:8e:
         0b:57:2c:f6:0b:8b:50:86:e2:7d:a9:5b:d2:f6:8e:7e:cd:41:
         73:79:36:f5:9b:0c:38:7b:1f:17:79:aa:ab:64:ca:4c:ee:ee:
         05:a3:b8:5b:56:3c:f9:d5:4d:fd:cb:18:33:4b:c3:86:c1:d3:
         03:96:bb:26:0f:0a:b8:ff:d6:84:d6:cc:ea:a5:50:30:8f:5d:
         41:29:e9:1c
```



```
Issuer: CN=kubernetes
Subject: CN=system:kube-scheduler
```



稍后通过   CN=system:kube-scheduler 追踪此服务使用到的权限信息



# 查看权限相关

> https://v1-17.docs.kubernetes.io/docs/setup/best-practices/certificates/#configure-certificates-for-user-accounts



上面链接可以查到 此服务的Default CN 是 system:kube-scheduler

通过上一节查看的信息 也已经得到此信息



#### 查看关联的 clusterrolebinding


```
kubectl get clusterrolebinding -o custom-columns='Name:metadata.name,Sub:subjects'  | grep 'system:kube-scheduler'
```



```
system:kube-scheduler                                  [map[apiGroup:rbac.authorization.k8s.io kind:User name:system:kube-scheduler]]
system:volume-scheduler                                [map[apiGroup:rbac.authorization.k8s.io kind:User name:system:kube-scheduler]]
```

--------------



```
kubectl  get clusterrolebinding system:kube-scheduler -o yaml
```



```
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-scheduler
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: system:kube-scheduler
```

---------



```
kubectl  get clusterrolebinding system:volume-scheduler -o yaml
```

```
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:volume-scheduler
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: system:kube-scheduler
```



#### 查看关联的 rolebinding



```
kubectl get rolebinding -A -o custom-columns='Name:metadata.name,Sub:subjects'  | grep 'system:kube-scheduler'
```

```
system::extension-apiserver-authentication-reader   [map[apiGroup:rbac.authorization.k8s.io kind:User name:system:kube-controller-manager] map[apiGroup:rbac.authorization.k8s.io kind:User name:system:kube-schedule]]
system::leader-locking-kube-scheduler               [map[apiGroup:rbac.authorization.k8s.io kind:User namesystem:kube-scheduler] map[kind:ServiceAccount name:kube-scheduler namespace:kube-system]]
```

----------

```
kubectl get rolebinding -n kube-system   system::extension-apiserver-authentication-reader -o yaml
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
kubectl get rolebinding -n kube-system system::leader-locking-kube-scheduler -o yaml
```



```
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: system::leader-locking-kube-scheduler
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: system:kube-scheduler
- kind: ServiceAccount
  name: kube-scheduler
  namespace: kube-system
```



#  手动创建客户端证书

```
rm -rf  /kube-scheduler-certs
mkdir  /kube-scheduler-certs
cd  /kube-scheduler-certs
```



```
openssl genrsa -out kube-scheduler.key 2048
```

```
openssl req -new -key kube-scheduler.key  -subj "/CN=system:kube-scheduler" \
-out kube-scheduler.csr
```



```
cat > kube-scheduler-csr.conf  << EOF
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
openssl x509 -req -in kube-scheduler.csr  \
-CA    /etc/kubernetes/pki/ca.crt \
-CAkey /etc/kubernetes/pki/ca.key \
-CAcreateserial -out kube-scheduler.crt -days 10000 \
-extensions v3_ext -extfile kube-scheduler-csr.conf
```



----------------

查看证书

```
openssl x509 -in /kube-scheduler-certs/kube-scheduler.crt  -noout -text
```

```
Issuer: CN=kubernetes
Subject: CN=system:kube-scheduler
X509v3 Extended Key Usage: 
                TLS Web Client Authentication
```

# 创建kubeconfig



```
rm -rf /etc/kubernetes/scheduler.conf
```



```
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.crt \
  --embed-certs=true \
  --server=https://192.168.3.101:6443 \
  --kubeconfig=/etc/kubernetes/scheduler.conf
```



```
kubectl config set-credentials system:kube-scheduler \
  --client-certificate=/kube-scheduler-certs/kube-scheduler.crt \
  --client-key=/kube-scheduler-certs/kube-scheduler.key \
  --embed-certs=true \
  --kubeconfig=/etc/kubernetes/scheduler.conf
```



```
kubectl config set-context system:kube-scheduler@kubernetes \
  --cluster=kubernetes \
  --user=system:kube-scheduler \
  --kubeconfig=/etc/kubernetes/scheduler.conf
```



```
kubectl config use-context system:kube-scheduler@kubernetes \
--kubeconfig=/etc/kubernetes/scheduler.conf
```



# 验证kubeconfig

```
kubectl  get pod   -A --kubeconfig=/etc/kubernetes/scheduler.conf
```





# systemd

```
cat > /usr/lib/systemd/system/kube-scheduler.service << EOF
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
--authentication-kubeconfig=/etc/kubernetes/scheduler.conf \\
--authorization-kubeconfig=/etc/kubernetes/scheduler.conf \\
--bind-address=127.0.0.1 \\
--kubeconfig=/etc/kubernetes/scheduler.conf \\
--leader-elect=true
Restart=always
RestartSec=10s
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```





```
cat /usr/lib/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-scheduler \
--authentication-kubeconfig=/etc/kubernetes/scheduler.conf \
--authorization-kubeconfig=/etc/kubernetes/scheduler.conf \
--bind-address=127.0.0.1 \
--kubeconfig=/etc/kubernetes/scheduler.conf \
--leader-elect=true
Restart=always
RestartSec=10s
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```





# 删除static pod

```
mv /etc/kubernetes/manifests/kube-scheduler.yaml  /opt/
```





# 启动



```
systemctl  start kube-scheduler
```

```
systemctl  status  kube-scheduler
```

```
systemctl  enable  kube-scheduler
```



# 重启测试

```
reboot
```



