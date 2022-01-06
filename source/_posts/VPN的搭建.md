---
title: 关于Tinc VPN的搭建
date: 2022-01-06 19:39:14
tags: VPN

+description: "Welcome to Hexo! This is your very first post."
---

## 关于
Tinc 是一种相对小众的VPN方案。但是支持很多环境的安装，例如ubuntu，centos都可以用自带的包管理工具安装。<!-- more -->  
下面是对于Ubuntu的示例

## 准备两台（至少）机器
至少要有一台机器有稳定的公网IP

## 安装
在两台机器上都安装tinc
`apt update && apt install -y tinc`


## 配置

以下分别命名两台机器为pek，lax
下面我们建一个tinc名为test

网卡tinc0，内网网段设置为172.16.0.0/28

### 创建test目录
两边都执行
`mkdir /etc/tinc/test && mkdir /etc/tinc/test/hosts`

### 编辑tinc.conf即tinc的配置文件

`vim /etc/tinc/test/tinc.conf`
分两部分

pek写入
```
Interface = tinc0
Name = pek
Mode = switch
ConnectTo = lax
# In newer versions (>= 1.1) you can use AutoConnect instead
# AutoConnect = yes
```

lax写入
```
Interface = tinc0
Name = lax
Mode = switch
# In newer versions (>= 1.1) you can use AutoConnect instead
# AutoConnect = yes
```

### 编辑tinc-up，即当启用test时需要tinc执行的脚本

`vim /etc/tinc/test/tinc-up`
pek写入

```bash
#!/bin/sh

# set the interface up
ip link set dev $INTERFACE up

# add transfer networks
ip addr add 172.16.0.1/28 dev $INTERFACE scope link

# add routes
# ip route add 172.16.0.0/28 dev $INTERFACE src 172.16.0.2

```

lax写入

```
#!/bin/sh

# set the interface up
ip link set dev $INTERFACE up

# add transfer networks
ip addr add 172.16.0.2/28 dev $INTERFACE scope link

# add routes
# ip route add 172.16.0.0/28 dev $INTERFACE src 172.16.0.1

```

### 编辑tinc-down，即当关闭test时需要tinc执行的脚本

`vim /etc/tinc/test/tinc-down`
两边都键入

```
#!/bin/sh

ip link set $INTERFACE down

```


### 给予tinc-up和tinc-down脚本添加执行权限
`chmod +x tinc-*`

### 创建tinc之间交互需要的秘钥，
`cd /etc/tinc/test && tincd -K4096 -n test`

### 交换lax和pek的秘钥
将lax上的/etc/tinc/test/hosts/lax的内容写入pek的/etc/tinc/test/hosts/lax文件内 并在第一行加入`Address=<lax的外网IP>`
将pek上的/etc/tinc/test/hosts/pek的内容写入pek的/etc/tinc/test/hosts/pek文件内 并在第一行加入 `Address=<pek的外网IP>`
简单的来说就是交换两台机器彼此的秘钥，并写入IP地址

### 将test写入tinc的nets.boot中,并重启tinc
`echo 'test' >### /etc/tinc/nets.boot && service tinc restart`

之后就可以用ifconfig来查看是否有新加的tinc0网卡，以及IP地址。可以用ping来校验双方是否联通

## 参考
[Tinc doc](https://www.tinc-vpn.org/documentation/tinc.pdf)
[DN42](https://dn42.eu/howto/tinc)

