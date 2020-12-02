## 集群

### 1. 目标 

- **高可用：**当一台服务器停止服务后，对于业务及用户毫无影响。 停止服务的原因可能由于网卡、路由器、机房、CPU负载过高、内存溢出、自然灾害等不可预期的原因导致，在很多时候也称单点问题。 
- **突破数据量限制：**一台服务器不能储存大量数据，需要多台分担，每个存储一部分，共同存储完整个集群数据。最好能做到互相备份，即使单节点故障，也能在其他节点找到数据。
- **数据备份容灾：**单点故障后，存储的数据仍然可以在别的地方拉起。
- **压力分担：**由于多个服务器都能完成各自一部分工作，所以尽量的避免了单点压力的存在。

### 2. 基础形式

### ![](E:\程序人生\个人学习笔记\学习笔记图床\QQ图片20201201170615.png)4. MySql集群

#### 4.1 主流架构

##### ![](E:\程序人生\个人学习笔记\学习笔记图床\QQ图片20201201172427.png)

- **Mysql-MMM**
- **MHA**
- **InnoDB Cluster**

总结：**主写从读，主从同步**

#### 4.2 部署

- **主节点**

```
docker run -p 3307:3306 --name mysql-master \
-v /usr/local/docker/mysql/master/log:/var/log/mysql \
-v /usr/local/docker/mysql/master/data:/var/lib/mysql \
-v /usr/local/docker/mysql/master/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql:5.7
```

- **从节点**

```
docker run -p 3317:3306 --name mysql-slaver-01 \
-v /usr/local/docker/mysql/slaver/log:/var/log/mysql \
-v /usr/local/docker/mysql/slaver/data:/var/lib/mysql \
-v /usr/local/docker/mysql/slaver/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql:5.7
```

- **常规配置**

```
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
init_connect="SET collation_connection = utf8_unicode_ci"
init_connect="SET NAMES utf8"
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
```

- **主从配置**

```
# 主节点添加如下配置
server-id=1    
log-bin=mysql-bin
read-only=0
binlog-do-db=tesco-admin
binlog-do-db=tesco-coupon
binlog-do-db=tesco-goods
binlog-do-db=tesco-mqmsg
binlog-do-db=tesco-order
binlog-do-db=tesco-seata
binlog-do-db=tesco-user
binlog-do-db=tesco-ware
      
replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema

# 从节点添加如下配置
server-id=2   
log-bin=mysql-bin
read-only=1
binlog-do-db=tesco-admin
binlog-do-db=tesco-coupon
binlog-do-db=tesco-goods
binlog-do-db=tesco-mqmsg
binlog-do-db=tesco-order
binlog-do-db=tesco-seata
binlog-do-db=tesco-user
binlog-do-db=tesco-ware
      
replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema
```

- 授权配置

  在Master节点的连接，命令行界面执行

```
GRANT  REPLICATION  SLAVE  ON  *.*  TO  'backup'@'%' IDENTIFIED  BY '123456';   #添加用来同步的用户

SHOW MASTER STATUS;			#查看状态
```

​	在Master节点的连接，命令行界面执行

```
change master to master_host='192.168.75.161',master_user='backup',master_password='123456',master_log_file='mysql-bin.000002',master_log_pos=0,master_port=3307;			#告诉从节点连接哪个主节点
```

- 命令

```
#开启从库
start slave;

#关闭从库
stop slave;

#查看从库状态
show slave status;
```

#### 4.3 ShardingSphere - 分库分表













