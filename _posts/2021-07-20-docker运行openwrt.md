---
layout: post
tag: docker
title: docker运行openwrt
---

**信息来自下面这个网址，感谢原作者。**

https://hub.docker.com/r/sulinggg/openwrt-mini

### 1.打开网卡混杂模式

`sudo ip link set eth0 promisc on `

### 2.创建网络

`docker network create -d macvlan --subnet=192.168.123.0/24 --gateway=192.168.123.1 -o parent=eth0 macnet`

### 3.拉取镜像

`docker pull sulinggg/openwrt-mini`

### 4.创建并启动容器

`docker run --restart always --name openwrt -d --network macnet --privileged hub.docker.com/suling/openwrt:latest /sbin/init`

注：如果不需要随机启动，可以去掉 --restart always

### 5.进入容器并修改相关参数

`docker exec -it openwrt bash`

修改IP

`vim /etc/config/network`

IP和局域网内网IP网段保持一致

重启网络服务

`/etc/init.d/network restart`

之后可以通过web进入控制面板进行设置，需要关闭DHCP