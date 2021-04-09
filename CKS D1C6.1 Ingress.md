# CKS D1C6.1 Ingress
###### tags: `CKS Day1`
## 生成RSA算法的自签名证书用于Ingress
Generating Certificate/Key RSA Pair For Your Ingress:
```
openssl req -x509 -newkey rsa:4096 -sha256 -days 365 -nodes \
  -keyout kuard.key -out kuard.crt -subj "/CN=cks.xiamen.vm" \
  -addext "subjectAltName=DNS:xiamen.vm,IP:10.79.230.9,IP:10.79.230.10,IP:10.79.230.19"
```
<br>

生成Secret对象, 调用证书与密钥. (注意Secret是Namespace级别对象)
```
kubectl create secret tls kuard-tls --cert=kuard.crt --key=kuard.key
```
校验证书
```
openssl x509 -in kuard.crt -text -noout
```
<br>

## Option, 生成ECC算法的自签名证书用于Ingress
```
openssl ecparam -genkey -name prime256v1 -out eccpk.key
openssl req -x509 -sha256 -days 365 -key eccpk.key -out xiamen.vm.crt -subj "/CN=cks.xiamen.vm" \
```
如果在生成ECC证书时报Cannot open file:../crypto/rand/randfile.c:88:Filename=/root/.rnd错误, 尝试执行如下命令再生成证书:
```
cd ~/; openssl rand -writerand .rnd
```
生成Secret对象, 调用ECC证书与密钥. (注意Secret是Namespace级别对象)


```
kubectl create secret tls kuard-ecc --cert=xiamen.vm.crt --key=eccpk.key
```

## Ingress With TLS Ingress

```
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: kuard
  labels:
    app: kuard
spec:
  tls:
  - hosts:
      - cks.xiamen.vm
    #secretName: kuard-tls
    secretName: kuard-ecc
  rules:
  - host: cks.xiamen.vm
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          serviceName: kuard
          servicePort: 80
EOF
```

<br>

Cli校验
```
curl --insecure -vvI https://cks.xiamen.vm:32742
```

浏览器校验
![](https://i.imgur.com/d2cqgNt.jpg)

