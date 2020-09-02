---
title: nginx限流方案 
---


# 代理层Nginx限流(降级)预案

作者：东方德胜

[TOC]

## 典型服务架构介绍
典型的互联网服务访问链路都是分层结构的，从流量入口，到应用层，到后端资源层；其中流量入口可能会有4层负载均衡、7层负载均衡，负载均衡也可能有多层；流量打到应用层之后，就要看具体的业务场景了，不同的业务可能会有不同的依赖请求，包括对第三方服务的或者对缓存、数据库、队列等资源的访问。

```
┌──────────────────┐                
│   LB - layer 4   │                
└─────────┬────────┘                
          │                         
          ▼                         
┌──────────────────┐                
│   LB - layer 7   │                
└──────────────────┘                
          │                         
          │                         
       ......                       
          │                         
          ▼                         
    ┌──────────┐        ┌──────────┐
    │   App    │───────│Resources │
    └──────────┘        └──────────┘
```

## 预案适用场景
此预案的应用场景并不是很多，在一般情况下我们不会启用这个预案。但是对Nginx入口(上图LB-layer 7处[^footnote])做限流的操作作为一项特殊场景下的预案还是有必要简单整理下的。
针对合适启动这个预案需要经过一系列的**人工判断**，并且具体是否要启用这个预案一般是需要经过业务方、运维、依赖方进行讨论确认的。
很多时候是否启动这个预案可能需要依赖一定的经验来判断，需要通过多项监控指标来综合考虑，没有某一个单一的监控指标可以指导启用这个方案。

[^footnote]: 之所以将限流操作放到7层代理来做，是因为7层上可以更方便的基于业务来做配置，会更灵活。针对下文中描述的场景，这个预案是一个弃车保帅的方案，是为了避免特定的业务对整体业务造成影响，或者被迫放弃部分业务流量。

下面简单描述几个适用此预案的场景：

1. 疑似被刷量，需要配合业务QPS、访问日志中的来源IP、访问接口统计来甄别；
2. 正常访问量增长，业务层代理、后端APP、后端资源等无法支撑，并且也没有可用的扩容资源、或者无法快速扩容；
3. 基于极端场景下资源池资源不足，需要舍弃部分非核心业务来保障核心业务的时候，可能会对非核心业务做缩容，此时可能需要配合Nginx入口层的限流策略，避免因为后端缩容导致(非核心)业务完全不可用；
4. 异常访问量，访问量大幅突增，后端无法支撑，并且无法快速定位、解决异常问题的时候；

## 监控指标
因为此预案的开启无法通过单一的监控指标来做判断，用于辅助判断是否启用此预案的监控指标列举如下：

1. 域名QPS历史趋势
2. 域名访问日志
3. 容器LB、APP层机器(Pod)负载、后端资源负载

## 操作手册


### 相关文档
1. http://nginx.org/en/docs/http/ngx_http_limit_req_module.html#limit_req
2. https://www.centos.bz/2017/03/using-nginx-limit_req-limit-user-request-rate/


### 操作方法
启用限流需要两个步骤：
1. 在http配置区段中声明一个limit_req_zone
2. 在需要做限流的http、server、location配置区段内部启用limit_req指令进行限速

### 配置语法


* limit_req_zone


```
Syntax:
limit_req_zone key zone=name:size rate=rate [sync];

Default:
—

Context:
http
```


* limit_req


```
Syntax:
limit_req zone=name [burst=number] [nodelay | delay=number];

Default:
—

Context:
http, server, location
```

### 配置样例


```
http {
     limit_req_zone $upstream_addr zone=thatsit:100m rate=4000r/s;

    server {
  server_name thatsit.com；
        location / {  
   limit_req zone=thatsit burst=300 nodelay;
        }
    }
}
```


###配置解释

* limit_req_zone

```
limit_req_zone $upstream_addr zone=thatsit:100m rate=4000r/s;`
```


> 声明一个大小为100M名称为thatsit的limit_req_zone(会申请一块共享内存来键值的状态)；
> 使用`$upstream_addr`变量来作为存储键值对用的键
> 限制到同一个upstream(`$upstream_addr`)的平均请求频率为每秒4000 requests；

* limit_req

```
limit_req zone=thatsit burst=300 nodelay;
```


> 在`location /`中启动请求限制，使用名为thatsit的共享内存空间来存储限流中用到的键值对
> 限制并发数300
> 请求超过限制之后不做延迟处理，直接响应错误，默认的错误状态码为503，这个状态码可以通过`limit_req_status`指令进行修改




## 注意事项
1. 配置的时候需要综合考虑请求的平均处理时间来配置请求并发数(burst)和频率(rates)；
2. 需要综合评估`nodelay`参数的影响，默认配置都是开启delay的，即所有超过限制频率的请求都会被延迟处理，在请求量高的情况下可能会超过Nginx `backlog`的限制；
3. 我们一般会把这个限制做在LB层，LB层一般都会包含多台机器，在做限制的时候需要做好计算(总的rates限制需要乘以LB服务器的数量)；
4. `limit_req_zone`参数支持配置多个变量作为key，可以根据实际需求合理配置；