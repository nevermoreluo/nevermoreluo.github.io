---
title: Strace
date: 2022-01-07 11:38:44
tags: [linux, cpp]
---


Strace是Linux环境下用于调试诊断应用程序调用systemcall的工具。

由于Strace只检测系统调用，因此Strace只是一个分析的侧面。  
例如：命令`perl -e 'while(1){}'`不会产生任何系统调用

系统调用包括以下几个方面file, process, network, signal, ipc, desc， 默认将检测所有系统调用即all<!--more-->


## 使用Strace统计运行期间系统调用占比
```
$ sudo strace -f -c main
strace: Process 30879 detached
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 64.79    2.184606       35236        62         3 futex
 28.13    0.948503         566      1675         1 clock_nanosleep
  2.85    0.096019           8     12105           getsockopt
  1.35    0.045417           3     16768           mprotect
  0.82    0.027561           7      4002      3633 recvfrom
  0.65    0.021963           3      7108           lseek
 ...
------ ----------- ----------- --------- --------- ----------------
100.00    3.371847                 55680      5957 total
```

## 使用Strace 记录程序运行期间的系统调用   
可以通过添加-tt记录调用时间戳，与程序日志对比可区分出程序各个阶段的系统调用  

 

```
$ sudo strace -f -tt -T -o /tmp/test/trace.log main

$ cat /tmp/test/trace.log
18721 21:02:31.936942 execve("/bin/bash", ["bash", "-c", "APOWO_SERVER_ROOT=/home/never/wo"...], 0x7ffef5c46408 /* 14 vars */) = 0 <0.000077>
18721 21:02:31.937082 brk(NULL)         = 0x563070e9c000 <0.000017>
18721 21:02:31.937133 arch_prctl(0x3001 /* ARCH_??? */, 0x7ffe20222180) = -1 EINVAL (Invalid argument) <0.000017>
18721 21:02:31.937183 access("/etc/ld.so.preload", R_OK) = -1 ENOENT (No such file or directory) <0.000019>


PID  调用时间 系统调用名称 返回 消耗时间  
18721 21:02:31.936942 execve("/bin/bash", ["bash", "-c", "APOWO_SERVER_ROOT=/home/never/wo"...], 0x7ffef5c46408 /* 14 vars */) = 0 <0.000077> 
```

## 参考
https://linux.die.net/man/1/strace