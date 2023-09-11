---
title: Linux service register script
date: 2023-09-11 10:53:39
categories:
  - Code
tags: [linux, shell]
---

## 前言
工作中存在一些功能需要注册为服务，保证开机自启以及自动重启业务。其实本质上就是注册一个service unit, 但是很多同学都没搞过。为了减少一些同学的使用成本，就给了个简单脚本按步骤执行。<!-- more --> 


## Show me the code

```bash
#!/bin/bash
set -e;

CURRENT_SCRIPT_DIR=$(dirname "$(readlink -f "$0")")
# 默认来说该注册脚本和程序一起放程序根目录那么CURRENT_SCRIPT_DIR就是项目根目录
PROJECT_DIR=$CURRENT_SCRIPT_DIR

# 设定服务名称 后续systemctl stop $SERVICE_NAME 用
SERVICE_NAME=

# 设定服务运行程序
# EXEC_FILE=$PROJECT_DIR/$SERVICE_NAME

# 运行程序 这里可以带参数
# 例如 START_CMD=$EXEC_FILE -f $PROJECT_DIR/config.json
# 可能会有启动参数 没有就直接填执行程序
START_CMD=$EXEC_FILE

HAS_START_CMD_ARGS=0

# 参数缺省时，用户交互手动输入
if [ -z "$SERVICE_NAME" ]; then
  read -p "Please set an service name: " SERVICE_NAME
  if [ -z "$SERVICE_NAME" ]; then
	echo "We can not register service with empty name, bye!!!";
	exit 1;
  fi
fi


if [ -z "$EXEC_FILE" ]; then
	read -p "Please set full path for project dir: " PROJECT_DIR
	if [ -z "$PROJECT_DIR" ]; then
		PROJECT_DIR=$CURRENT_SCRIPT_DIR
	fi
	read -p "Please set full path for exec file: "  EXEC_FILE
	
fi


if [[ $HAS_START_CMD_ARGS -ne 0 && -z $START_CMD ]]; then
    read -p "Please set start cmd: " START_CMD
else
    START_CMD=$EXEC_FILE
fi


SYSTEM_CONF=/usr/lib/systemd/system/$SERVICE_NAME.service


echo "SERVICE_NAME: $SERVICE_NAME"
echo "PROJECT_DIR: $PROJECT_DIR"
echo "EXEC_FILE: $EXEC_FILE"
echo "START_CMD: $START_CMD"

echo "Start install $SERVICE_NAME..."
cd $PROJECT_DIR
if [ -e $SYSTEM_CONF ]; then
  read -p "Warning!!! $SERVICE_NAME is already installed. skip install and setup $SERVICE_NAME? (y/n) " choice
      case "$choice" in
          y|Y )
              echo "Continuing..."
              ;;
          n|N )
              echo "Exiting..."
              exit 0
              ;;
          * )
              echo "Invalid choice. Exiting..."
              exit 1
              ;;
      esac
else
  echo "Install $SERVICE_NAME...";

  chmod +x $EXEC_FILE

  # 创建systemd服务文件
  tee $SYSTEM_CONF > /dev/null <<EOT
[Unit]
Description=$SERVICE_NAME Service
After=network.target
[Service]
Type=simple
Restart=always
RestartSec=10
ExecStart=$START_CMD
WorkingDirectory=$PROJECT_DIR
[Install]
WantedBy=multi-user.target
EOT

  # 重新加载systemd服务
  systemctl daemon-reload

  # 启动DeviceBus服务 可能要改配置 先不启动
  systemctl start $SERVICE_NAME
  # 设置开机自启动
  systemctl enable $SERVICE_NAME
  echo "Install $SERVICE_NAME end"
  echo "Start program with systemctl start $SERVICE_NAME"
fi
```


## Learn more
- [Bilibili Video](https://www.bilibili.com/video/BV1Tz4y1s7QW/?spm_id_from=333.337.search-card.all.click&vd_source=c791cc730e25c0810d7b175e322c8dbf)
- [Arch wiki](https://wiki.archlinuxcn.org/wiki/Systemd)
