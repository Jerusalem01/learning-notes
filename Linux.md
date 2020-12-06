## Linux 

#### 1. 简介

Linux 是一种自由和开放源码的类 UNIX 操作系统，使用 Linux 内核。 Linux 是一个领先的操作系统，世界上运算最快的 10 台超级电脑运行的都是 Linux 操作系统。

严格来讲，Linux 这个词本身只表示 Linux 内核，但在实际上人们已经习惯了用 Linux 来形容整个基于 Linux 内核，并且使用 GNU 工程各种工具和数据库的操作系统。

通常情况下，Linux 被打包成供桌上型电脑和服务器使用的 Linux 发行版本。

目前市面上较知名的发行版有：Ubuntu、RedHat、CentOS、Debian、Fedora、SuSE、OpenSUSE、TurboLinux、BluePoint、RedFlag、Xterm、SlackWare等。

关于操作系统的选型

```
https://www.bilibili.com/video/av27095823/
```

#### 2.重要目录结构

| 目录 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| bin  | 存放二进制可执行文件(ls,cat,mkdir等)                         |
| etc  | 存放系统配置文件                                             |
| usr  | 存放系统应用程序，比较重要的目录/usr/local 本地管理员软件安装目录 |
| var  | 用于存放运行时需要改变数据的文件                             |

#### 3. 常用操作及命令

- **操作文件目录**

| 命令   | 说明                               | 用法                       | 参数 | 说明                       |
| ------ | ---------------------------------- | -------------------------- | ---- | -------------------------- |
| ls -al | 显示文件和目录列表                 |                            |      |                            |
| ll     | 显示文件和目录列表                 |                            |      | 超级管理员                 |
| mkdir  | 创建目录                           | mkdir 目录名               |      |                            |
|        |                                    | mkdir -p 父级目录名 目录名 | -p   | 递归创建                   |
| cd     | 切换目录                           | cd 目录名                  |      |                            |
| touch  | 生成空文件                         | touch 文件名               |      |                            |
| echo   | 生成带内容文件                     | echo 内容 > 文件名         |      |                            |
|        |                                    | echo 内容 >> 文件名        |      | 追加内容                   |
| cat    | 显示文本文件内容                   | cat 文件名                 |      |                            |
| cp     | 复制文件或目录                     |                            |      |                            |
| rm     | 删除文件                           | rm 文件目录名称            |      |                            |
|        |                                    |                            | -f   | 强制删除文件或目录         |
|        |                                    |                            | -r   | 同时删除该目录下的所有文件 |
| mv     | 移动文件或目录                     | mv 文件名 目录名           |      |                            |
| find   | 在文件系统中查找指定的文件         |                            |      |                            |
| pwd    | 显示当前工作目录                   |                            |      |                            |
| more   | 分页显示文本文件内容               |                            |      |                            |
| head   | 显示文件开头内容                   |                            |      |                            |
| tail   | 显示文件结尾内容                   |                            |      |                            |
| grep   | 在指定的文本文件中查找指定的字符串 |                            |      |                            |
| tree   | 用于以树状图列出目录的内容         |                            |      |                            |
| ln     | 建立软链接                         |                            |      |                            |

- **压缩命令**

```
压缩文件夹：tar -zcvf 文件名
解压文件夹：tar -zxvf 文件名
```

- **系统管理命令**

| 命令     | 说明                                         |
| -------- | -------------------------------------------- |
| stat     | 显示指定文件的相关信息,比ls命令显示内容更多  |
| who      | 显示在线登录用户                             |
| hostname | 显示主机名称                                 |
| uname    | 显示系统信息                                 |
| top      | 显示当前系统中耗费资源最多的进程             |
| ps       | 显示瞬间的进程状态                           |
| du       | 显示指定的文件（目录）已使用的磁盘空间的总量 |
| df       | 显示文件系统磁盘空间的使用情况               |
| free     | 显示当前内存和交换空间的使用情况             |
|          | -h（换算）                                   |
| ifconfig | 显示网络接口信息                             |
| ping     | 测试网络的连通性                             |
| netstat  | 显示网络状态信息                             |
| clear    | 清屏                                         |
| kill     | 杀死一个进程                                 |

- **开关机命令**

```
# 重启
reboot
shutdown -r now

# 关机
shutdown -h now
```

- **Vim编辑器**

```
运行模式：
编辑模式：等待编辑命令输入
插入模式：编辑模式下，输入 i 进入插入模式，插入文本信息
命令模式：在编辑模式下，输入 : 进行命令模式

命令：
:q 直接退出vi
:wq 保存后退出vi ，并可以新建文件
:q! 强制退出
:w file 将当前内容保存成某个文件
:set number 在编辑文件显示行号
:set nonumber 在编辑文件不显示行号
:set paste 保留源格式粘贴文本

添加或删除多行注释
进入vi/vim编辑器，按CTRL+V进入可视化模式（VISUAL BLOCK）
移动光标上移或者下移，选中多行的开头
选择完毕后，按大写的的I键，此时下方会提示进入“insert”模式，输入你要插入的注释符，例如#
最后按ESC键，你就会发现多行代码已经被注释了
删除多行注释的方法，同样 Ctrl+v 进入列选择模式，移到光标把注释符选中，按下d，注释就被删除了
```

- **Root用户**

```
# 设置 Root账户密码
sudo passwd root

# 切换到 Root
su

# 设置允许远程登录 Root
vi /etc/ssh/sshd_config

修改：
LoginGraceTime 120
#PermitRootLogin without-password     //注释此行
PermitRootLogin yes                 //加入此行
StrictModes yes

# 重启服务
service ssh restart
```

- **账户管理**

```
# 增加用户
useradd 用户名
useradd -u (UID号)
useradd -p (口令)
useradd -g (分组)
useradd -s (SHELL)
useradd -d (用户目录)

# 修改用户
usermod -u (新UID)
usermod -d (用户目录)
usermod -g (组名)
usermod -s (SHELL)
usermod -p (新口令)
usermod -l (新登录名)
usermod -L (锁定用户账号密码)
usermod -U (解锁用户账号)
如：usermod -u 1024 -g group2 -G root lusifer
将 lusifer 用户 uid 修改为 1024，默认组改为系统中已经存在的 group2，并且加入到系统管理员组

# 删除用户
userdel 用户名 (删除用户账号)
userdel -r 删除账号时同时删除目录
如：userdel -r lusifer
删除用户名为 lusifer 的账户并同时删除 lusifer 的用户目录

# 组账户维护
groupadd 组账户名 (创建新组)
groupadd -g 指定组GID
groupmod -g 更改组的GID
groupmod -n 更改组账户名
groupdel 组账户名 (删除指定组账户）

# 口令维护
passwd 用户账户名 (设置用户口令)
passwd -l 用户账户名 (锁定用户账户)
passwd -u 用户账户名 (解锁用户账户)
passwd -d 用户账户名 (删除账户口令)
gpasswd -a 用户账户名 组账户名 (将指定用户添加到指定组)
gpasswd -d 用户账户名 组账户名 (将用户从指定组中删除)
gpasswd -A 用户账户名 组账户名 (将用户指定为组的管理员)

# 用户和组状态
su 用户名(切换用户账户)
id 用户名(显示用户的UID，GID)
whoami (显示当前用户名称)
groups (显示用户所属组)
```

- **文件权限管理**

```
# 查看文件
ls 显示文件名称
ls –al 显示文件或者目录及权限信息

ls -l 文件名 
显示信息包括：文件类型 (d 目录，- 普通文件，l 链接文件)，文件权限，文件的用户，文件的所属组，文件的大小，文件的创建时间，文件的名称

# 目录权限
-：普通文件
rw-：说明用户 lusifer 有读写权限，没有运行权限
r--：表示用户组 lusifer 只有读权限，没有写和运行的权限
r--：其他用户只有读权限，没有写权限和运行的权限
（3个字符为一组。r 只读，w 可写，x 可执行，- 表示无此权限）

# 更改操作权限

## chown（改变文件或者目录所有者）
chown [-R] 用户名称 文件或者目录
chown [-R] 用户名称 用户组名称 文件或目录
（-R：进行递归式的权限更改，将目录下的所有文件、子目录更新为指定用户组权限）

## chmod（改变访问权限）
chmod [who] [+ | - | =] [mode] 文件名

## who
表示操作对象可以是以下字母的一个或者组合
u：用户 user
g：用户组 group
o：表示其他用户
a：表示所有用户是系统默认的
## 操作符号
+：表示添加某个权限
-：表示取消某个权限
=：赋予给定的权限，取消文档以前的所有权限
## mode
表示可执行的权限，可以是 r、w、x

# 数字设定法

数字设定法中数字表示的含义
0 表示没有任何权限
1 表示有可执行权限 = x
2 表示有可写权限 = w
4 表示有可读权限 = r

r w x|r – x|r - x
4 2 1|4 - 1|4 - 1
user |group|others

若要 rwx 属性则 4+2+1=7
若要 rw- 属性则 4+2=6
若要 r-x 属性则 4+1=5
```

- **软件包管理**

```
# 常用 APT 命令
## 安装软件包
apt-get install packagename
## 删除软件包
apt-get remove packagename
## 更新软件包列表
apt-get update
## 升级有可用更新的系统（慎用）
apt-get upgrade

# 其它 APT 命令
## 搜索
apt-cache search package
## 获取包信息
apt-cache show package
## 删除包及配置文件
apt-get remove package --purge
## 了解使用依赖
apt-cache depends package
## 查看被哪些包依赖
apt-cache rdepends package
## 安装相关的编译环境
apt-get build-dep package
## 下载源代码
apt-get source package
## 清理无用的包
apt-get clean && apt-get autoclean
## 检查是否有损坏的依赖
apt-get check
```

#### 4. LVM 磁盘扩容

- **LVM 的基本概念**

```
#物理卷 Physical volume (PV)   
可以在上面建立卷组的媒介，可以是硬盘分区，也可以是硬盘本身或者回环文件（loopback file）。物理卷包括一个特殊的 header，其余部分被切割为一块块物理区域。

#卷组 Volume group (VG)
将一组物理卷收集为一个管理单元。

#逻辑卷 Logical volume (LV)
虚拟分区，由物理区域（physical extents）组成。

#物理区域 Physical extent (PE)
硬盘可供指派给逻辑卷的最小单位（通常为 4MB）。
```

- **磁盘操作相关命令**

```
# 查看挂载点
df -h

# 显示当前的 logical volume
lvdisplay
备注： 注意这里目前有两个，一个是文件系统所在的 volume，另一个是 swap 分区使用的 volume，当然，我们需要扩容的是第一个

# 显示当前的 volume group
vgdisplay
备注： 注意 VG SIZE，这里应该是你当前的可用空间大小，待扩容完毕，这里显示的应该是最终的大小

# 显示当前的 physical volume
pvdisplay
```

- **开始 LVM 扩容**

```
# 查看 fdisk
fdisk -l

# 查看所有连接到电脑上的储存设备
fdisk -l |grep '/dev'
# 创建 sdb 分区

fdisk /dev/sdb
n	# 新建分区
l	# 选择逻辑分区，如果没有，则首先创建扩展分区（p），然后再添加逻辑分区（硬盘：最多四个分区 P-P-P-P 或 P-P-P-E）
回车
回车
回车
w	# 写入磁盘分区
# 格式化磁盘
mkfs -t ext4 /dev/sdb1
# 创建 PV
pvcreate /dev/sdb1
# 查看卷组
pvscan
# 扩容 VG
vgdisplay

vgextend ubuntu-vg /dev/sdb1
# 扩容 LV

# 增加指定大小
lvextend -L +30G /dev/ubuntu-vg/root
或
# 按百分比扩容
lvextend -l +100%FREE /dev/ubuntu-vg/root
# 刷新分区
resize2fs /dev/ubuntu-vg/root
# 删除 unknown device
pvscan
vgreduce --removemissing ubuntu-vg
注意：不要卸载扩容的磁盘，可能出现丢失数据或是系统无法启动
```

#### 5. 部署应用到生产环境

##### 5.1 安装 Java、maven

- **解压缩并移动到指定目录**

```
# 创建目录(递归创建)
mkdir -p /usr/local/java
mkdir -p /usr/local/maven

# 设置所有者（开启上传权限）
chown -R root:root /usr/local/java/
chown -R root:root /usr/local/maven/
或者
chmod 777 java
chmod 777 maven

# 上传压缩包到指定目录

# 解压缩
tar -zxvf jdk-8u231-linux-x64.tar.gz
tar -zxvf apache-maven-3.6.1-bin.tar.gz
```

- **配置环境变量**

1. 配置系统环境变量

```
vi /etc/environment

添加如下语句

PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games"

export JAVA_HOME=/usr/local/java/jdk1.8.0_231
export JRE_HOME=/usr/local/java/jdk1.8.0_231/jre
export MAVEN_HOME=/usr/local/maven/apache-maven-3.6.1
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$MAVEN_HOME/bin
```

1. 配置用户环境变量

```
vi /etc/profile

添加如下语句

if [ "$PS1" ]; then
  if [ "$BASH" ] && [ "$BASH" != "/bin/sh" ]; then
    # The file bash.bashrc already sets the default PS1.
    # PS1='\h:\w\$ '
    if [ -f /etc/bash.bashrc ]; then
      . /etc/bash.bashrc
    fi
  else
    if [ "`id -u`" -eq 0 ]; then
      PS1='# '
    else
      PS1='$ '
    fi
  fi
fi

export JAVA_HOME=/usr/local/java/jdk1.8.0_231
export JRE_HOME=/usr/local/java/jdk1.8.0_231/jre
export MAVEN_HOME=/usr/local/maven/apache-maven-3.6.1
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH:$HOME/bin:$MAVEN_HOME/bin

if [ -d /etc/profile.d ]; then
  for i in /etc/profile.d/*.sh; do
    if [ -r $i ]; then
      . $i
    fi
  done
  unset i
fi

# 使用户环境变量生效
source /etc/profile
```

1. 环境配置的永久化

```
vi /etc/bash.bashrc

添加如下内容：

export JAVA_HOME=/usr/local/java/jdk1.8.0_231
export JRE_HOME=/usr/local/java/jdk1.8.0_231/jre
export MAVEN_HOME=/usr/local/maven/apache-maven-3.6.1
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH:$HOME/bin:$MAVEN_HOME/bin

# 使配置生效
source /etc/bash.bashrc
```

- **验证**

```
java -version
mvn -v

# 重启
reboot

java -version
mvn -v
```

- **为其他用户更新用户环境变量**

```
su 用户名

source /etc/profile
```

##### 5.2 安装 Tomcat

- **解压缩并移动到指定目录**

```
# 解压缩
tar -zxvf apache-tomcat-8.5.23.tar.gz

# 变更目录名
mv apache-tomcat-8.5.23 tomcat

# 移动目录
mv tomcat/ /usr/local/
```

- **常用命令**

```
# 启动
/usr/local/tomcat/bin/startup.sh

# 停止
/usr/local/tomcat/bin/shutdown.sh

# 目录内执行脚本
./startup.sh
```

##### 5.3 安装 MySQL

- **安装**

```
# 更新数据源
apt-get update

# 安装 MySQL
apt-get install mysql-server
（系统将提示您在安装过程中创建 root 密码。选择一个安全的密码，并确保你记住它，因为你以后需要它）
```

- **配置**

```
因为是全新安装，您需要运行附带的安全脚本。这会更改一些不太安全的默认选项，例如远程 root 登录和示例用户。在旧版本的 MySQL 上，您需要手动初始化数据目录，但 Mysql 5.7 已经自动完成了。
# 运行安全脚本
mysql_secure_installation
(这将提示您输入您在之前步骤中创建的 root 密码。您可以按 Y，然后 ENTER 接受所有后续问题的默认值，但是要询问您是否要更改 root 密码。您只需在之前步骤中进行设置即可，因此无需现在更改)
```

- **验证**

```
systemctl status mysql.service

# 查看 MySQL 版本
mysqladmin -p -u root version
```

- **配置远程访问**

1. 修改配置文件

```
vi /etc/mysql/mysql.conf.d/mysqld.cnf

修改

bind-address = 127.0.0.1           //注释此行
```

1. 重启`MySQL`

```
service mysql restart
```

1. 登录`MySQL`

```
mysql -u root -p
```

1. 授权`root`用户允许所有人连接

```
grant all privileges on *.* to 'root'@'%' identified by 'mysql root 账户密码';
```

- **因弱口令无法成功授权解决步骤**

```
# 查看密码安全级别
select @@validate_password_policy;

# 设置密码安全级别
set global validate_password_policy=0;

# 查看密码长度限制
select @@validate_password_length;

# 设置密码长度限制
set global validate_password_length=1;
```

- **常用命令**

```
#启动
service mysql start

#停止
service mysql stop

#重启
service mysql restart
```

- **其它配置**

```
vi /etc/mysql/mysql.conf.d/mysqld.cnf

# 配置默认字符集
在 [mysqld] 节点上增加如下配置
[client]
default-character-set=utf8

在 [mysqld] 节点底部增加如下配置
default-storage-engine=INNODB
character-set-server=utf8
collation-server=utf8_general_ci

#配置忽略数据库大小写敏感
在 [mysqld] 节点底部增加如下配置
lower-case-table-names = 1   
```

- **部署**

```
·····
```