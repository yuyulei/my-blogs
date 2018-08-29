---
title: "Docker Network"
date: 2018-08-28T22:28:55+08:00
archives: "2018"
tags: ["docker", "network"]
author: Yu Yulei
---

理解 docker 支持的 network drivers, 对常见的 host、bridge 模式进一步的了解和实践。

<!--more-->

# Docker Network

理解 docker 支持的 network drivers, 对常见的 host、bridge 模式进一步的了解和实践。

## 讨论范围

不涉及操作系统网络细节，比如：

* docker 是如何在 linux 系统中修改 iptables 规则的
* docker 是如何在 windows 系统中操作路由规则的
* docker 是如何对数据包封装、加密等

## Network Drivers

包括：`bridge`, `host`, `overlay`, `macvlan`, `none`

### bridge

docker 默认选择的网络驱动。默认情况下，docker 启动的时候会自动创建 "default bridge driver" 为 `docker0`（或者就叫 `docker`）。

#### use bridge

**summary:** 多个 containers 在一台 Docker host 上相互交流时。


**区别(user-defined bridges & the default bridge):**

1. `user-defined bridges`:
    * 更好的隔离性和可操作性: 处于同一个 `user-defined bridges` 中的 containers。所有端口都是暴露的, 不需要 `-p` 或者 `--publish` 暴露端口
    * 提供自动的 DNS 解决方案: `containers` 可以通过 ip, container name 相互连接访问, 不需要 `--link`(实质上是修改 container 内部 `/etc/hosts`)
    * 随时可对 `user-defined bridges` 关停
    * 自定义配置网络参数, 比如 MTU等

2. `the default bridge`:
    * 共享环境变量


### host

host 模式会复用 docker host（宿主机）的网络，不再与宿主机存在网络隔离性。

#### use host

**summary:** 不独立宿主机的 Network


### overlay

overlay 常用语连接 多个 `docker daemon` 组成的网络, 比如说要实现 `docker swarm` 模式下的某个 `container` 与单独的 `docker standalone` 模式下的一个 `container` 交流。

### macvlan

macvlan 允许将一个 `MAC` 地址赋予一个 `container`, 使其以一个物理设备形式出现在网络中

### none

通常适用于已存在自定义网络的环境下，比如早期的 `kirk`。


## Network tutorial

### `bridge`

官方文档：https://docs.docker.com/network/network-tutorial-standalone/#use-user-defined-bridge-networks

掌握几点：

1. 容器启动的时候会注册到对应的 `bridge`, 无论是默认的还是自定义的。
2. 不同的 `bridge` 之间的 container 是无法相互访问的
3. 允许一个容器同时属于不同的 `bridge`

### `host`

官方文档：https://docs.docker.com/network/network-tutorial-host/#other-networking-tutorials

以 `host` 模式启动

```
docker run --rm -d --network host --name my_nginx nginx

...

sudo netstat -tulpn | grep :80
```

可以看到宿主机的 80 端口被占用了。

## Use a proxy server

官方文档：https://docs.docker.com/network/proxy/#configure-the-docker-client

高版本的 docker 通过修改 `~/.docker/config.json`, 类似如下：

```
{
 "proxies":
 {
   "default":
   {
     "httpProxy": "http://127.0.0.1:3001",
     "noProxy": "*.test.example.com,.example2.com"
   }
 }
}
```

效果是：当你创建或者启动一个容器时，上述内容会以环境变量的形式加到容器里面。

低版本的 docker 设置环境变量

|Variable|Dockerfile example|`docker run` Example|
|---|---|---|
|HTTP_PROXY|ENV HTTP_PROXY "http://127.0.0.1:3001"|--env HTTP_PROXY="http://127.0.0.1:3001"|
|HTTPS_PROXY|ENV HTTPS_PROXY "https://127.0.0.1:3001" |--env HTTPS_PROXY="https://127.0.0.1:3001"|
|FTP_PROXY|ENV FTP_PROXY "ftp://127.0.0.1:3001"|--env FTP_PROXY="ftp://127.0.0.1:3001"|
|NO_PROXY|ENV NO_PROXY "*.test.example.com,.example2.com"|--env NO_PROXY="*.test.example.com,.example2|
