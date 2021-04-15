# D3C4 AppArmor
###### tags: `CKS Day3`

参考链接:
[K8s.io - AppArmor](https://kubernetes.io/zh/docs/tutorials/clusters/apparmor/)

[开源AA策略生成器Bane](https://github.com/genuinetools/bane)

[AA官方文档](https://gitlab.com/apparmor/apparmor/-/wikis/Documentation)

## 配置示例

创建一个AA Profile:
profile k8s-apparmor-example-deny-write 即为该配置的名字
```
#include <tunables/global>
profile k8s-apparmor-example-deny-write flags=(attach_disconnected) {
  #include <abstractions/base>
  file,
  # Deny all file writes.
  deny /** w,
}
```
加载Profile:
```
apparmor_parser -q apparmor-profile
```
校验Profile以加载为执行模式:
```
apparmor_status
apparmor module is loaded.
18 profiles are loaded.
18 profiles are in enforce mode.
   /sbin/dhclient
   /usr/bin/lxc-start
   /usr/bin/man
   /usr/bin/ubuntu-core-launcher
   /usr/lib/NetworkManager/nm-dhcp-client.action
   /usr/lib/NetworkManager/nm-dhcp-helper
   /usr/lib/connman/scripts/dhclient-script
   /usr/lib/snapd/snap-confine
   /usr/lib/snapd/snap-confine//mount-namespace-capture-helper
   /usr/sbin/tcpdump
   docker-default
   k8s-apparmor-example-deny-write
   lxc-container-default
   lxc-container-default-cgns
   lxc-container-default-with-mounting
   lxc-container-default-with-nesting
   man_filter
   man_groff
0 profiles are in complain mode.
22 processes have profiles defined.
22 processes are in enforce mode.
   docker-default (3277)
   docker-default (3324)
   docker-default (3326)
   docker-default (3330)
   docker-default (3348)
   docker-default (3577)
   docker-default (3662)
   docker-default (4806)
   docker-default (5182)
   docker-default (5487)
   docker-default (5653)
   docker-default (5826)
   docker-default (5986)
   docker-default (6242)
   docker-default (6611)
   docker-default (8040)
   docker-default (9712)
   docker-default (15628)
   docker-default (15806)
   docker-default (16791)
   docker-default (16815)
   docker-default (29656)
0 processes are in complain mode.
0 processes are unconfined but have a profile defined.
```

创建Pod调用Profile, 注意分配到已加载Profile的节点上:
```
apiVersion: v1
kind: Pod
metadata:
  name: apparmor-demo
  labels:
    app: net-tools
  annotations:
    # Tell Kubernetes to apply the AppArmor profile "k8s-apparmor-example-deny-write".
    # Note that this is ignored if the Kubernetes node is not running version 1.4 or greater.
    container.apparmor.security.beta.kubernetes.io/apparmor-c1: localhost/k8s-apparmor-example-deny-write
spec:
  nodeName: ubuk8s-vm02
  containers:
  - name: apparmor-c1
    image: net-tools:v2
    command: [ "sh", "-c", "sleep 1h" ]
```