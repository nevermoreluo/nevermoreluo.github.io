---
title: Centos7升级内核
date: 2023-11-16 13:39:24
categories:
  - Tools
tags: [linux]
---

## 前言
部分编译工具老的内核存在bug，需要升级内核，简单记录下<!-- more -->

### 1. 下载内核rpm历史版本  
查找 kernel rpm[历史版本](http://mirrors.coreix.net/elrepo-archive-archive/kernel/el7/x86_64/RPMS/)   
找到合适的历史版本下载到需要升级的机器上，这里以5.4为例，其他版本可以在上面的链接里面找  
```
wget http://mirrors.coreix.net/elrepo-archive-archive/kernel/el7/x86_64/RPMS/kernel-lt-devel-5.4.203-1.el7.elrepo.x86_64.rpm
wget http://mirrors.coreix.net/elrepo-archive-archive/kernel/el7/x86_64/RPMS/kernel-lt-headers-5.4.203-1.el7.elrepo.x86_64.rpm
wget http://mirrors.coreix.net/elrepo-archive-archive/kernel/el7/x86_64/RPMS/kernel-lt-5.4.203-1.el7.elrepo.x86_64.rpm
```


### 2. 安装内核  
使用rpm安装下载好的内核  
```
rpm -ivh kernel-lt-5.4.203-1.el7.elrepo.x86_64.rpm
rpm -ivh kernel-lt-devel-5.4.203-1.el7.elrepo.x86_64.rpm
rpm -ivh kernel-lt-headers-5.4.203-1.el7.elrepo.x86_64.rpm
```
或者如果确认目录下只有这个版本的源的话直接一把梭哈也行`rpm -Uvh kernel-*.rpm`  


### 3. 查看内核顺序

`awk -F\' '$1=="menuentry " {print i++ " : " $2}' /boot/grub2/grub.cfg`

执行后会有以下类似的结果  
```
[root@localhost ~]# awk -F\' '$1=="menuentry " {print i++ " : " $2}' /boot/grub2/grub.cfg
0 : CentOS Linux (5.4.203-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux (3.10.0-1160.119.1.el7.x86_64) 7 (Core)
2 : CentOS Linux (3.10.0-1160.el7.x86_64) 7 (Core)
3 : CentOS Linux (0-rescue-4f420f77aafd44d9abf76817faaf25d4) 7 (Core)
```
当中的0，1，2，3就是开机时可选的内核  

### 4. 设置刚安装的内核为启动内核  
根据上一步的最前面的枚举设置内核，比如我们装的是5.4，对应的索引是0，就执行`grub2-set-default 0`  
执行后需要`reboot`重启生效  

### 5. 查看内核是否已经升级  

```
[root@localhost ~]# uname -a
Linux localhost.localdomain 5.4.203-1.el7.elrepo.x86_64 #1 SMP Fri Jul 1 09:00:33 EDT 2022 x86_64 x86_64 x86_64 GNU/Linux
```

