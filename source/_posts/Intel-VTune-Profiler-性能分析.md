---
title: Intel VTune Profiler 性能分析
date: 2023-09-14 10:53:38
categories:
  - Tools
tags: [profile]
---


### Intel VTune Profiler
Intel VTune Profiler 以下简称Vtune,是intel推出的一款用于应用性能分析的一款软件工具集，是的区别于普通的分析软件，它其实一系列分析工具的集合。支持windows，linux，macos等系统，以及c/c++, Java, go, .Net, python等多种语言。<!--more--> 
其内部还有多种分析维度
![VTune](/images/vtune_utils.png)

#### 安装 
按照[官方文档](https://www.intel.com/content/www/us/en/developer/tools/oneapi/vtune-profiler-download.html)安装即可   
以下简述 ubuntu安装过程  
下载对应的离线安装shell脚本，（当然也可以按apt安装），
`wget https://registrationcenter-download.intel.com/akdlm/irc_nas/18267/l_oneapi_vtune_p_2021.8.0.533_offline.sh` 此处部分同学有科学上网软件的话，可能浏览器下载比wget快。

下载完成后执行`sudo bash l_oneapi_vtune_p_2021.8.0.533_offline.sh`即开始安装  

安装完成后,还需要根据[文档](https://www.intel.com/content/www/us/en/develop/documentation/get-started-with-vtune/top/linux-os.html)执行`source /opt/intel/oneapi/vtune/latest/env/vars.sh`,即可通过vtune-gui打开vtune了， 最好将该命令写入 ~/.bashrc内



#### Hotspots
是用于分析cpu占用时间最多的调用，可以通过各种图形化gui的分析过滤方式查看cpu比较高的时刻函数占用情况。  

依旧是经典的旁路采样分析模式的采样数据， 配置完应用程序以及环境变量后，当你按下执行后，程序会开始启动，按下stop则开始分析采样数据生成各种图表。

其中的Bottom-up, Top-down Tree, Flame Graph 图表,其实都是同一组数据在不同纬度以及不同图表展示下的结果，因此可以统一对下方条件过滤处，进行时间，线程，动态库等条件过滤达到自己想查看的某些时刻或者某些部分的cpu占用比，这个很实用。

由于图表会涉及很多业务数据 因此只将外部启动的主程序图表做一个展示，

![image](/images/vtune_ex1.png)  
外部启动程序其实会根据配置载入不同的动态库实现不同功能因此主要都是系统库dlopen的占用,暂时不准备对这部分做优化，而且此处占用也只会在服务器起来的时候占用一次，因此本次暂时不优化该部分。

我们甚至可以通过双击对应想要查看的函数，获得函数每行语句的cpu时间占比。  
![image](/images/vtune_ex1_code.png)
