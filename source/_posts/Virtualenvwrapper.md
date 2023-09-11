---
title: Virtualenvwrapper
date: 2022-01-06 19:31:46
categories:
  - Code
tags: python
---


通过命令`pip install virtualenvwrapper`安装<!-- more -->  

```bash
# 将下面代码写入SHELL的环境变量，通过相关搜索得到建议ubuntu用户将其添加如~/.bash_profile文件内，deepin下好像没有，因此我将其添加进了~/.bashrc
export WORKON_HOME=~/Envs
source /usr/local/bin/virtualenvwrapper.sh
```

添加好环境变量之后执行如下语句
```bash
mkdir -p $WORKON_HOME #为虚拟环境建立一个文件夹，仅第一次需要
echo 'cd $VIRTUAL_ENV'>>$WORKON_HOME/postactivate #设置进入虚拟环境时的默认目录
mkvirtualenv $envname #创建一个虚拟环境，和virtual一样可以指定参数例如-p
workon $envname #实际上当你新建的时候，你就会进入这个虚拟环境，这条命令用于你下一次再进入或切换到envname环境下
deactivate #退出虚拟环境
rmvirtualenv $envname #删除虚拟环境
```
