---
title: docker安装
---

# docker安装

本文档主要是面向开发者，阐述锦鲤部署如何安装docker

前置条件：

1. 需要root权限执行。
2. 需要jinliconfig大于1.3版本

---

## 检查系统发行版本

### Ubuntu

```bash
lsb_release -a | grep Release
```

返回值：

    No LSB modules are available.
    Release:	20.04

### Centos

```bash
lsb_release -a |grep Release
```

返回值：

    Release: 7.4

---

## Ubuntu部署docker

前置条件：

1. Ubuntu 20.04

> step 1: 安装必要的一些系统工具

```bash
apt-get update
apt-get -y install apt-transport-https ca-certificates curl software-properties-common git curl wget unzip lrzsz
```

> step 2: 安装GPG证书

```bash
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```

> Step 3: 写入软件源信息

```bash
add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
```

> Step 4: 更新并安装 Docker-CE

```bash
apt-get -y update
apt-get -y install docker-ce=5:19.03.12* docker-compose=1.25.0-1
```

> Step 5: 开启Docker服务

```bash
systemctl start docker
```

> step 6: 设置开机启动

```bash
systemctl enable docker
```

> 设置docker源

```bash
mkdir -p /etc/docker
```

```golang
WriteFile("/etc/docker/daemon.json", `{"log-driver":"json-file","log-opts":{"max-size":"1m","max-file":"1"}}`)
```

> 重载docker

```bash
sudo systemctl daemon-reload && sudo systemctl restart docker
```

---

## Centos 安装docker

前置条件：

1. Centos7

> 安装源

```bash
yum install epel-release -y
```

> 关闭防火墙

```bash
systemctl stop firewalld.service && systemctl disable firewalld.service && setenforce 0 && sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
```

> step 1: 安装必要的一些系统工具

```bash
yum install -y yum-utils device-mapper-persistent-data lvm2 git curl wget unzip lrzsz
```

> Step 2: 添加软件源信息

```bash
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

> Step 3: 更新并安装 Docker-CE

```bash
yum makecache fast && yum -y install docker-ce-19.03.12 docker-compose-1.18.0
```

> Step 4: 开启Docker服务

```bash
systemctl start docker
```

> step 5: 设置开机启动

```bash
systemctl enable docker
```

> 设置docker源

```bash
mkdir -p /etc/docker
```

> 配置docker日志，golang代码

```golang
class.WriteFile("/etc/docker/daemon.json", `{"log-driver":"json-file","log-opts":{"max-size":"1m","max-file":"1"}}`)
```

> 重载docker

```bash
sudo systemctl daemon-reload && sudo systemctl restart docker
```
