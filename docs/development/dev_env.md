---
title: 开发环境配置
---

# 开发环境配置

本章节讲解锦鲤部署开发环境配置。

## 基础包安装

### Centos

```bash
yum install -y git wget
```

### Ubuntu

```bash
apt install -y git wget
```

## 安装golang

golang版本要求为1.14版本以上

CPU架构为x86_64架构

### 下载安装包

https://dl.google.com/go/go1.15.linux-amd64.tar.gz

### 解压安装包

```bash
tar -C /usr/local -xzf go1.14.3.linux-amd64.tar.gz
```

### 设置临时环境变量（当前可用）

```bash
export PATH=$PATH:/usr/local/go/bin
```

### 设置永久环境变量（重启可用）

```bash
echo 'export PATH=$PATH:/usr/local/go/bin' >> /etc/profile
```

### 验证安装

```bash
go version
```

## 配置国内golang源

```bash
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.io,direct
```

## 下载代码

```bash
git clone https://github.com/jinlicode/jinliconfig.git
```

## 测试编译

```bash
cd jinliconfig && go build
```
