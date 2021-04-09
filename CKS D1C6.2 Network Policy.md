# CKS D1C6.2 Network Policy
###### tags: `CKS Day1`
参考链接:
[K8s.io Network Policy](https://github.com/ahmetb/kubernetes-network-policy-recipes)
[Example Network Policy
](https://github.com/ahmetb/kubernetes-network-policy-recipes)

Network Policy 示意图:
![](https://i.imgur.com/pHvhMTP.gif)

官方示例:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

多段入向条件选择:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-app2web
spec:
  podSelector:
    matchLabels:
      app: kuard
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          project: default
    - podSelector:
        matchLabels:
          app: nettools
          tier: web
    ports:
    - protocol: TCP
      port: 8080
  
  - from:
    - ipBlock:
        cidr: 100.64.192.0/24
    ports:
    - protocol: TCP
      port: 8080
```