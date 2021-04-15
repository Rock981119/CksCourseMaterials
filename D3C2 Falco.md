# CKS D3C2 Falco
###### tags: `CKS Day3`

## Falco Architecture
Falco can detect and alert on any behavior that involves making Linux system calls. Falco alerts are triggered based on specific system calls, arguments, and properties of the calling process. Falco operates at the user space and kernel space. **The system calls are interpreted by the Falco kernel module.** The syscalls are then analyzed using the libraries in the userspace. The events are then filtered using a rules engine where the Falco rules are configured. Suspicious events are then alerted to outputs that are configured as Syslog, files, Standard Output, and others.
![](https://i.imgur.com/699DmJy.png)


## Install Falco
参考链接:
[Falco 安装](https://falco.org/docs/getting-started/installation/)

1. Trust the falcosecurity GPG key, configure the apt repository, and update the package list:

```
curl -s https://falco.org/repo/falcosecurity-3672BA8F.asc | apt-key add -
echo "deb https://download.falco.org/packages/deb stable main" | tee -a /etc/apt/sources.list.d/falcosecurity.list
apt-get update -y
```
2. Install kernel headers:
```
apt-get -y install linux-headers-$(uname -r)
```

3. Install Falco:
```
apt-get install -y falco
```
Falco, the kernel module driver, and a default configuration are now installed. Falco is being ran as a systemd unit.

See running for information on how to manage, run, and debug with Falco.

Uninstall Falco:
```
apt-get remove falco
```


---
## Helm Install Falco as DaemonSet
```
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update
```
直接安装:
```
helm install falco falcosecurity/falco
```
卸载:
```
helm uninstall falco
```

默认Falco镜像仓库使用DockerHub, 如果遇到镜像拉取失败, 可以替换成AWS ECR仓库, 参数如下:
```
helm install falco --set image.registry=public.ecr.aws --set image.repository=b4t6c0y6/falco falcosecurity/falco
```
更多Helm安装参数和开关参考:
[This chart adds Falco to all nodes in your cluster using a DaemonSet](https://github.com/falcosecurity/charts/tree/master/falco#adding-falcosecurity-repository)