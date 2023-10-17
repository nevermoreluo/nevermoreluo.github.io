---
title: Mysql主从同步配置
date: 2023-10-17 16:29:28
categories:
  - Tools
tags: [database]
---


对于环境隔离，读写分离的情况，有些情况下我们需要用到mysql的主从同步。以下介绍如何使用docker创建一个主从同步的数据库服务<!-- more -->

### 0. 环境假设
由于测试在本机创建两个数据库服务，因此以下ip都按<master_ip>,<slave_ip>代之用于区分，实际过程请按照实际情况填写

即   
主数据库为 <master_ip>:3308  
从数据库为 <slave_ip>:3309

### 1. 创建两个db，一个master监听3308，一个slave监听3309
```shell
# log-bin=mysql-bin以及server-id=1的设置可以通过my.cnf/my.ini配置修改，也可以通过docker启动mysql的命令行设置，注意修改my.cnf/my.ini配置后需要重启mysql

# 建立一个master db，监听3308
docker run --name mysql_master -e MYSQL_ROOT_PASSWORD=aaa123  -p 3308:3306 -d mysql:5.7 --log-bin=mysql-bin --server-id=1

# 建立一个slave db, 监听3309
docker run --name mysql_slave  -e MYSQL_ROOT_PASSWORD=aaa123  -p 3309:3306 -d mysql:5.7 --log-bin=mysql-bin --server-id=2
```

### 2. 登陆主库master db，设置专门为slave建立的用户并赋予权限

```sql
mysql -h <master_ip> -P 3308 -u root -p 

> CREATE USER 'backup'@'<slave_ip>' IDENTIFIED BY '<backup_password>';
> GRANT REPLICATION SLAVE ON *.* TO 'backup'@'<slave_ip>';
> FLUSH PRIVILEGES;
> SHOW MASTER STATUS;

File                Position
mysql-bin.000003	1737			
```
记录下最后`SHOW MASTER STATUS`返回结果中的File和Position  
注意 <backup_password>填自己设定的值即可，下面会用到


### 3. 登陆从库slave db，设置master的链接方式
```sql
mysql -h <slave_ip> -P 3309 -u root -p 
> STOP SLAVE;
> change master to MASTER_HOST='<master_ip>', MASTER_PORT=3308, MASTER_USER='backup',MASTER_PASSWORD='<backup_password>',MASTER_LOG_FILE='mysql-bin.000003',MASTER_LOG_POS=1737;
> START SLAVE;
> SHOW SLAVE STATUS;

```
这里注意一下<backup_password>需要填入第2步中设置的<backup_password>，MASTER_LOG_FILE以及MASTER_LOG_POS也是根据第2步中`SHOW MASTER STATUS`返回设置的  
设置成功时，`SHOW SLAVE STATUS;`会返回 Slave_IO_Running 和 Slave_SQL_Running都是Yes即可  


### 4. 回到主库，尝试创建数据表插入数据等操作，并确认从库会自动同步即可
