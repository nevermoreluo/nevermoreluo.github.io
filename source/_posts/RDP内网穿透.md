---
title: RDP内网穿透
date: 2023-09-11 15:58:59
categories:
  - Tools
tags: [windows]
---

用于有公网服务器的内网穿透搭建

https://github.com/fatedier/frp 是一款内网穿透服务的开源服务，跨平台服务端和客户端<!-- more --> 

1. frp服务器

1.1 下载服务器

在该界面下载最新的压缩包 , (压缩包内包含了客户端和服务端，因此等会windows上也要下载一次做客户端)

‣

1.2 配置frp服务器

将压缩包内容解压到自定义目录内，例如我的应用都会装到`/opt`目录下

以下假设解压到/opt/frp内

服务端配置文件/opt/frp/frps.ini

```bash
# frps.ini
[common]
# 服务器绑定的端口 这个端口需要外网可以访问
bind_port = 7000

# 用来做服务器校验的， 服务器和客户端设置为一致即可，相当于密码建议修改
token = aabbccdd
```

1.3 尝试启动服务

/opt/frp/frps -c /opt/frp/frps.ini

正常启动就ok

可以按ctrl+c退出

1.4 启动服务器

可以直接使用上述测试方式启动`/opt/frp/frps -c /opt/frp/frps.ini`

但是建议改为[systemctl启动](https://gofrp.org/docs/setup/systemd/) 这样就能达到开机自启的效果了 

当然supervisor，pm2等这些也都可以

1. frp客户端

1.1 下载客户端

在该界面下载最新的压缩包 , (压缩包内包含了客户端和服务端)

‣

2.2 配置客户端

将压缩包解压到自己建的安装目录内

以下假设为 F:\app\frp

修改F:\app\frp\frpc.ini

```bash
# frpc.ini
[common]
# 服务器地址
server_addr = x.x.x.x
# 服务器绑定的端口 bind_port 
server_port = 7000
# 需要和服务器的token配置一致
token = aabbccdd

[rdp]
# rdp服务协议
type = tcp
# 本地的回环地址
local_ip = 127.0.0.1
# rdp端口 默认是3389
local_port = 3389
# rdp服务在服务器暴露的端口
remote_port = 6000
```

2.3 尝试启动

打开powershell（按win键入powershell搜索，点击powershell）

输入如下命令

```vbnet
F:\app\frp\frpc.exe -c F:\app\frp\frpc.ini
```

如果正常启动（如果你无法判断什么是正常启动，直接通过x.x.x.x:6000测试连接rdp服务测试），则可以进入下一步

2.4 设置开机自启动

新建文件F:\app\frp\start_frpc.vbs，键入如下命令

```vbnet
'start_frpc.vbs
CreateObject("WScript.Shell").Run """F:\app\frp\frpc.exe""" & "-c" & """F:\app\frp\frpc.ini""",0
```

建议通过任务计划程序实现开机自启

常规  不管用户是否登录都要运行 勾选不存储密码

触发器 改成登陆时

操作 启动程序 程序或脚本填 F:\app\frp\start_frpc.vbs或者通过浏览找到该文件

条件 电源内选项 勾选网络 只有在以下网络连接可用才启动