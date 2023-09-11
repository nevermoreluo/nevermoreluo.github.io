---
title: NSIS打包C# 程序
date: 2022-01-06 20:13:01
categories:
  - Code
tags: [C#, NSIS]
---

NSIS是一个Nullsoft脚本安装系统（英语：Nullsoft Scriptable Install System，缩写：NSIS）为一个开放源代码脚本驱动的封装安装档用工具。可以用其脚本语言自定安装的流程，同时支持多种语系的安装接口。<!-- more -->  


## 安装NSIS
有些人建议安装Unicode NSIS，NSIS3.0.1版本已经兼容了unicode，除非是老版本的，可以考虑安装Unicode NSIS。
[NSIS下载地址](https://nsis.sourceforge.io/Download)

下载安装完成后，可以在桌面看到一个NSIS的图标，先别急着打开它，现在我们还用不上它。

### 安装插件
安装一些常用插件（以下插件均为按需安装，如果不需要也可以选择不安装）

AccessControl.dll : 它提供了一组与Windows NT访问控制列表（ACL）管理相关的功能，相当于设定目录权限的功能。 [下载地址](https://nsis.sourceforge.io/AccessControl_plug-in)
KillProcDLL.dll：关闭指定名称的exe,用于卸载时关闭进程。[下载地址](https://nsis.sourceforge.io/KillProcDLL_plug-in)
liteFirewall.dll： 关闭防火墙[下载地址](https://leons.im/portfolio/nsis-plugin-litefirewall/)
UAC.dll，UAC.nsh [下载地址](https://nsis.sourceforge.io/UAC_plug-in)
以上插件下载解压后，将dll文件复制到C:\Program Files (x86)\NSIS\Plugins内，nsh文件复制到C:\Program Files (x86)\NSIS\Include内。
### 添加NSIS配置
下载一个[Exmaple这个例子源自](http://seesawworld.blogspot.sg/2016/02/1-nsis.html)，在此鸣谢原作者。
这里主要逻辑在Exmaple.nsi内，其他都是一些资源文件，简单修改一下配置：

```
; !define 用于声明一个常量
!define PRODUCT_NAME "YuanJin" 产品名称
!define PRODUCT_VERSION "1.0" 版本号
!define PRODUCT_PUBLISHER "YuanJin" 发布者
!define PRODUCT_WEB_SITE "http://www.yuanjin.io/" 产品官网，这将在安装完成后显示在开始程序栏内的快捷键

!define PRODUCT_STARTUP_APP "YuanJin.exe" 注意更改这个，这个就是你需要打包的exe文件，程序安装完成时将自动执行

; 主要讲一下依赖，个别程序会需要第三方依赖，我是将第三方依赖加入到DIR_TO_INSTALL中，如下配置
Section "ExampleSection01" SEC01
  CreateDirectory "$SMPROGRAMS\${PRODUCT_NAME}" 
  SetOutPath $INSTDIR
  SetOverwrite try
  File "..\Example\DIR_TO_INSTALL\JsonFx.dll"
  File "..\Example\DIR_TO_INSTALL\JsonFx.xml"
  File "..\Example\DIR_TO_INSTALL\Newtonsoft.Json.dll"
  File "..\Example\DIR_TO_INSTALL\Newtonsoft.Json.xml"
  File "..\Example\DIR_TO_INSTALL\YuanJin.exe.config"
  File "..\Example\DIR_TO_INSTALL\YuanJin.pdb"
  File "favico.ico"
  CreateShortCut "$SMPROGRAMS\${PRODUCT_NAME}\$(ShortCutUninstall)"  "$INSTDIR\uninst2.exe" "" "$INSTDIR\${APPLICATION_ICON}" 0
SectionEnd
```

## 打包程序

现在我们回到桌面，打开NSIS程序，你会看到一个很精致小巧的界面，没错不要怀疑：），雾~
点击Complier NSI scripts —-> load script —->找到刚刚编辑的Exmaple.nsi双击，如果一切正常你将会看到Test Installer可以被点击了。你会在Exmaple/output中看到打包好的exe了。

[NSIS中文文档](http://omega.idv.tw/nsis/Contents.html)
