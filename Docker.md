# Docker

### 为什么要使用 Docker ？

- 更高效的利用系统资源

- 更快速的启动时间

- 一致的运行环境

- 持续交付和部署

- 更轻松的迁移

- 更轻松的维护和扩展

- 对比传统虚拟机总结

| 特性       | 容器               | 虚拟机     |      |
| ---------- | ------------------ | ---------- | ---- |
| 启动       | 秒级               | 分钟级     |      |
| 硬盘使用   | 一般为MB           | 一般为GB   |      |
| 性能       | 接近原生           | 弱于       |      |
| 系统支持量 | 单机支持上千个容器 | 一般几十个 |      |

### 1. 基本概念
#### 1.1 基本概念
- 镜像（Image）
- 容器（Container）
- 仓库（Repository）
#### 1.2 引擎
- 一种服务器，一种称为守护进程并且长时间运行的程序。
- `REST API`用于指定程序可以用来与守护进程通信的接口，并指示它做什么。
- 一个有命令行界面 (CLI) 工具的客户端。
#### 1.3 系统架构

Docker 使用客户端-服务器 (C/S) 架构模式，使用远程 API 来管理和创建 Docker 容器。

容器与镜像的关系类似于面向对象编程中的对象与类。

| 标题            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| 镜像(Images)    | Docker 镜像是用于创建 Docker 容器的模板。                    |
| 容器(Container) | 容器是独立运行的一个或一组应用。                             |
| 客户端(Client)  | Docker 客户端通过命令行或者其他工具使用 Docker API (https://docs.docker.com/reference/api/docker_remote_api) 与 Docker 的守护进程通信。 |
| 主机(Host)      | 一个物理或者虚拟的机器用于执行 Docker 守护进程和容器。       |
| 仓库(Registry)  | Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。Docker Hub(https://hub.docker.com) 提供了庞大的镜像集合供使用。 |
| Docker Machine  | Docker Machine是一个简化 Docker 安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装Docker，比如VirtualBox、 Digital Ocean、Microsoft Azure。 |
#### 1.4 镜像
```
分层存储
```
#### 1.5 容器
#### 1.6 仓库
```
公有 Docker Registry
私有 Docker Registry
```
### 2. 安装
```
# 更新软件源  
sudo apt-get update 

# 安装所需依赖   
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common

# 安装 GPG 证书   
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add - 

# 新增软件源信息  
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"

# 再次更新软件源   
sudo apt-get -y update 

# 安装 Docker CE 版   
sudo apt-get -y install docker-ce

# 验证   
docker version   

# 配置加速器 
vi /etc/docker/daemon.json  
粘贴（采用paste方法）   
{     
  "registry-mirrors": [   
    "https://wzwr26i4.mirror.aliyuncs.com"   
    ]   
}

# 重启
systemctl restart docker

# 验证   
docker info
```
```
本文以Ubuntu为例，更多安装示例请参考
https://www.funtl.com/zh/docs-docker/%E5%AE%89%E8%A3%85-Docker.html
```
### 3. 镜像
#### 3.1 常用命令
```
# 获取镜像
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]

Docker 镜像仓库地址：地址的格式一般是 <域名/IP>[:端口号]。
默认地址是 Docker Hub。

# 运行
docker run -it --rm image-name bash

参数说明：
-it：这是两个参数
一个是 -i：交互式操作
一个是 -t 终端
--rm：容器退出后随之将其删除
bash：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 bash
exit 退出容器

# 列出镜像
docker images

# 镜像体积
docker system df

# 虚悬镜像
<none>               <none>              00285df0df87        5 days ago          342 MB

# 显示虚悬镜像：
docker images -f dangling=true

# 删除所有虚悬镜像
docker rmi $(docker images -q -f dangling=true)
docker prune

# 显示所有镜像(包括中间层镜像)
docker images -a

# 删除所有镜像
docker rmi $(docker images -q)

# 强制删除所有镜像
docker rmi -r $(docker images -q)

# 列出指定镜像
docker images ubuntu
docker images ubuntu:16.04
docker images -f since=mongo:3.2
docker images -f label=com.example.version=0.1

# 以特定格式显示镜像
docker images -q
docker images --format "{{.ID}}: {{.Repository}}"
docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"

# 删除本地镜像
docker image rm [选项] <镜像1> [<镜像2> ...]

其中，<镜像> 可以是 镜像短 ID、镜像长 ID、镜像名 或者 镜像摘要。
```
#### 3.2 Dockerfile
##### 3.2.1 构建镜像
```
# 在 Dockerfile文件所在目录执行
docker build -t image-name .

# 从 Docker文件构建 Docker映像
docker build -t image-name docker-file-location

参数说明：
-t：给镜像命名
. ：1.在当前目录找到Dockerfile        
    2.指定上下文目录并打包
```

**注意：**  

**镜像构建上下文（Context）** （**重要概念**） 

docker build 命令最后有一个 `.`  **==目的是指定上下文路径==**
```
Docker 在运行时分为 Docker 引擎（服务端守护进程）和客户端工具。Docker 的引擎提供了一组 REST API，被称为 Docker Remote API。而如 docker 命令这样的客户端工具，则是通过这组 API 与 Docker 引擎交互，从而完成各种功能。因此，虽然表面上我们好像是在本机执行各种 docker 功能，但实际上，一切都是使用的远程调用形式在服务端（Docker 引擎）完成。

当我们进行镜像构建的时候，并非所有定制都会通过 RUN 指令完成，经常会需要将一些本地文件复制进镜像，比如通过 COPY 指令、ADD 指令等。而 docker build 命令构建镜像，其实并非在本地构建，而是在服务端，也就是 Docker 引擎中构建的。

那么在这种客户端/服务端的架构中，如何才能让服务端获得本地文件呢？

这就引入了上下文的概念。当构建的时候，用户会指定构建镜像上下文的路径，docker build 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。

如果在 Dockerfile 中这么写：

COPY ./package.json /app/

这并不是要复制执行 docker build 命令所在的目录下的 package.json，也不是复制 Dockerfile 所在目录下的 package.json，而是复制 上下文（context） 目录下的 package.json。

因此，COPY 这类指令中的源文件的路径都是相对路径。这也是初学者经常会问的为什么 COPY ../package.json /app 或者 COPY /opt/xxxx /app 无法工作的原因，因为这些路径已经超出了上下文的范围，Docker 引擎无法获得这些位置的文件。如果真的需要那些文件，应该将它们复制到上下文目录中去。

一般来说，应该会将 Dockerfile 置于一个空目录下，或者项目根目录下。如果该目录下没有所需文件，那么应该把所需文件复制一份过来。如果目录下有些东西确实不希望构建时传给 Docker 引擎，那么可以用 .gitignore 一样的语法写一个 .dockerignore，用于剔除不需要作为上下文传递给 Docker 引擎的。

那么为什么会有人误以为 . 是指定 Dockerfile 所在目录呢？这是因为在默认情况下，如果不额外指定 Dockerfile 的话，会将上下文目录下的名为 Dockerfile 的文件作为 Dockerfile。

这只是默认行为，实际上 Dockerfile 的文件名并不要求必须为 Dockerfile，而且并不要求必须位于上下文目录中，比如可以用 -f ../Dockerfile.php 参数指定某个文件作为 Dockerfile。
```
- **其它docker build的用法** 

1. 直接用Git repo进行构建
```
docker build https://github.com/twang2218/gitlab-ce-zh.git#:8.14
```
2. 用给定的tar压缩包构建   
```
docker build http://server/context.tar.gz
```
3. 从标准输入中读取Dockerfile进行构建   
```
docker build - < Dockerfile
或
cat Dockerfile | docker build -

如果标准输入传入的是文本文件，则将其视为 Dockerfile，并开始构建。

这种形式没有上下文，因此不可以将本地文件 COPY 进镜像。
```
4. 从标准输入中读取上下文压缩包进行构建  
```
docker build - < context.tar.gz

如果标准输入的文件格式是gzip、bzip2以及xz，会直接将其展开，将里面视为上下文，并开始构建。
```
##### 3.2.2 指令详解
- **FROM 指定基础镜像**  

定制镜像，是以一个镜像为基础，在其上进行定制。基础镜像是必须指定的。因此一个 Dockerfile 中 FROM 是必备的指令，并且必须是第一条指令。

 Docker 存在一个特殊的镜像，名为 scratch ，它表示一个空白的镜像。以 scratch 为基础镜像，意味着不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。

- **RUN 执行命令**  

```
RUN <命令>   # shell格式

RUN ["可执行文件","参数1","参数2"]   # exec 格式
```
==错误写法：==
```
FROM debian:jessie
RUN apt-get update
RUN mkdir -p /usr/src/redis
RUN make -C /usr/src/redis
```
**说明：**  

1. Dockerfile 中每一个指令都会建立一层，上面的这种写法，产生非常臃肿、非常多层的镜像，不仅增加了构建部署的时间，还很容易出错。  
2. 很多运行时不需要的东西，都被装进了镜像里，比如编译环境、更新的软件包等等。  

正确写法：
```
FROM debian:jessie
RUN apt-get update \
    && mkdir -p /usr/src/redis \
    && make -C /usr/src/redis \
```
**说明：**  

1. Dockerfile 支持`Shell`类的行尾添加`\`进行换行，行首添加`#`进行注释。  
2. 一组命令的最后添加清理工作的命令，删除为了编译构建所需要的软件，清理所有下载、展开的文件，清理 apt 缓存文件。  
3. 镜像是多层存储，每一层的东西并不会在下一层被删除，会一直跟随着镜像。因此镜像构建时，一定要确保每一层只添加真正需要添加的东西，任何无关的东西都应该清理掉。

- **COPY - 复制文件**  

```
COPY <源路径>... <目标路径>
COPY ["<源路径1>",... "<目标路径>"]
```
```
COPY 指令将从构建上下文目录中<源路径>的文件/目录复制到新的一层的镜像内的<目标路径>位置。

<源路径>可以是多个，甚至可以是通配符，其通配符规则要满足 Go 的 filepath.Match 规则

<目标路径>可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 WORKDIR 指令来指定）。

目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。
```
- **ADD - 高级复制文件**  

ADD 指令和 COPY 的格式和性质基本一致。

在 COPY 基础上增加了一些功能：
```
<源路径> 可以是一个URL，Docker引擎会去下载这个链接的文件放到<目标路径>去。

下载后的文件权限自动设置为600，如果不是理想的权限，需要增加额外的一层RUN进行权限调整。

如果下载的是个压缩包，还需要额外的一层RUN指令进行解压缩。

所以不如直接使用RUN指令，然后使用 wget或者curl工具下载，处理权限、解压缩、清理无用文件。

因此，这个功能其实并不实用，不推荐使用。
```
```
如果<源路径>为一个tar压缩文件，压缩格式为gzip、bzip2、x，ADD指令会自动解压缩这个压缩文件到<目标路径>去。

在某些情况下，这个自动解压缩的功能非常有用。

但当我们是希望复制个压缩文件进去，而不解压缩，这时就不可以使用ADD命令了。
```
```
另外需要注意的是,ADD指令会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。因此在COPY和ADD指令中选择的时候，可以遵循这样的原则：所有的文件复制均使用COPY指令，仅在需要自动解压缩的场合使用ADD。
```
- **CMD - 容器启动命令**

CMD 指令用于指定默认的容器主进程的启动命令。

```
CMD <命令>   # shell格式

CMD ["可执行文件", "参数1", "参数2"...]   # exec格式（推荐）

CMD ["参数1", "参数2"...]   # 参数列表格式（在指定了 ENTRYPOINT指令后，用 CMD指定具体的参数）
```
```
在运行时可以指定新的命令来替代镜像设置中的这个默认命令。

ubuntu 镜像默认的 CMD 是 /bin/bash，如果我们直接 docker run -it ubuntu 的话，会直接进入 bash。

如 docker run -it ubuntu cat /etc/os-release。这就是用 cat /etc/os-release 命令替换默认的 /bin/bash 命令，输出系统版本信息。

在指令格式上，一般推荐使用 exec 格式，这类格式在解析时会被解析为 JSON 数组，因此一定要使用双引号 ，而不要使用单引号。

如果使用 shell 格式的话，实际的命令会被包装为 sh -c 的参数的形式进行执行。比如：

CMD echo $HOME

在实际执行中，会将其变更为：

CMD [ "sh", "-c", "echo $HOME" ]
```
**关于容器中应用在前台执行和后台执行的问题**
```
Docker 不是虚拟机，容器中的应用都应该以前台执行，而不是像虚拟机、物理机里面那样，用 upstart/systemd 去启动后台服务，容器内没有后台服务的概念。

一些初学者将 CMD 写为：

CMD service nginx start

然后发现容器执行后就立即退出了。甚至在容器内去使用 systemctl 命令结果却发现根本执行不了。

对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出。

而使用 service nginx start 命令，则是希望 upstart 来以后台守护进程形式启动 nginx 服务。

而 CMD service nginx start 会被理解为 

CMD [ "sh", "-c", "service nginx start"]

因此主进程实际上是 sh。那么当 service nginx start 命令结束后，sh 也就结束了，sh 作为主进程退出了，自然就会令容器退出。

正确的做法是直接执行 nginx 可执行文件，并且要求以前台形式运行。比如：

CMD ["nginx", "-g", "daemon off;"]
```
- **ENTRYPOINT - 入口点**
```

```
- **ENV**
```
设置环境变量
```
```
格式有两种：

ENV <key> <value>

ENV <key1>=<value1> <key2>=<value2>...
```
- **VOLUME**
```
格式为：

VOLUME ["<路径1>", "<路径2>"...]

VOLUME <路径>
```
- **EXPOSE**
```
声明运行时容器提供服务端口
```
```
格式为 

EXPOSE <端口1> [<端口2>...]
```
- **WORKDIR**
```
格式为 

WORKDIR <工作目录路径>
```

```
详细解释查阅千锋教育李卫民博客

funtl.com
```
##### 3.3.3 多阶段构建
```

```
### 4. 容器
#### 4.1 常用命令
```
# 启动
docker run -p -d --name container-name image-name

参数说明：
-p：分配端口
-d：守护态运行
--name：给容器命名
```
```
# 交互式启动
docker run -t -i image-name /bin/bash

参数说明：
-t：让Docker分配一个伪终端并绑定到容器的标准输入上
-i：让容器的标准输入保持打开

交互模式下，用户可以通过所创建的终端来输入命令
root@af8bae53bdd3:/# pwd
/
root@af8bae53bdd3:/# ls
bin boot dev etc home lib lib64 media mnt opt proc root run sbin srv sys tmp usr var
```

利用`docker run`来创建容器时，`Docker`在后台运行的标准操作包括：

1. 检查本地是否存在指定的镜像，不存在就从公有仓库下载
2. 利用镜像创建并启动一个容器
分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
3. 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
4. 从地址池配置一个 ip 地址给容器
5. 执行用户指定的应用程序
6. 执行完毕后容器被终止

```
# 交互式进入Docker容器
docker exec -it container-id /bin/bash
```
```
# 启动已终止容器
docker start 
```

容器的核心为所执行的应用程序，所需要的资源都是应用程序运行所必需的。除此之外，并没有其它的资源。可以在伪终端中利用 ps 或 top 来查看进程信息。
```
# Docker 守护态运行
docker run -d image-name
```
```
# 查看容器
docker ps
```
```
# 获取容器的输出信息
docker container logs[container ID or NAMES]
```
```
#查看所有容器
docker ps -a
```
```
# 删除所有处于终止状态的容器
docker rm $(docker ps -a -q)
docker prune
```
```
# 强制删除容器
docker rm -f 
```
```
# 终止容器
docker stop
```
当 Docker 容器中指定的应用终结时，容器也自动终止。
```
# 终止并启动
docker restart
```
```
# 导出容器
docker export 7691a814370e > ubuntu.tar
这样将导出容器快照到本地文件

# 导入容器快照
从容器快照文件中再导入为镜像
cat ubuntu.tar | docker import - image-name

此外，也可以通过指定 URL 或者某个目录来导入
docker import http://example.com/exampleimage.tgz example/imagerepo

注：用户既可以使用 docker load 来导入镜像存储文件到本地镜像库，也可以使用 docker import 来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。
```
### 5. 仓库
#### 5.1 Docker Hub
```
# 注册
https://cloud.docker.com 免费注册 Docker 账号。

# 命令行界面登录与退出
docker login 
交互式的输入用户名及密码
docker logout 

# 查找镜像
docker search 

根据是否是官方提供，可将镜像资源分为两类。
一种是基础镜像或根镜像。由 Docker 公司创建、验证、支持、提供。这样的镜像往往使用单个单词作为名字。
还有一种类型，由 Docker 的用户创建并维护的，往往带有用户名称前缀。、

#下载镜像
docker pull 

# 推送镜像
docker tag ubuntu:17.10 username/ubuntu:17.10
docker push username/ubuntu:17.10

# 自动创建
步骤：
1. 创建并登录 Docker Hub，以及目标网站；、
2. 在目标网站中连接帐户到 Docker Hub；
3. 在 Docker Hub 中 配置一个自动创建；
4. 选取一个目标网站中的项目（需要含 Dockerfile）和分支；
5. 指定 Dockerfile 的位置，并提交创建。
```
#### 5.2 私有仓库
```
请参考文档 
Registry-镜像管理平台
https://www.funtl.com/zh/docs-docker/Docker-%E7%A7%81%E6%9C%89%E4%BB%93%E5%BA%93.html#%E5%AE%89%E8%A3%85%E8%BF%90%E8%A1%8C-docker-registry
```
### 6. 数据管理
#### 6.1 数据卷
##### 6.1.1 参数的选择
```
Docker 新用户应该选择 --mount 参数
推荐使用 --mount 参数
```
##### 6.1.2 常用操作及命令
```
# 创建数据卷
docker volume create [volume_name]

# 查看所有数据卷
docker volume ls

# 查看指定数据卷的信息
docker volume inspect [volume_name]

#删除数据卷
docker volume rm [volume_name]

#删除所有未关联的数据卷
docker volume rm $(docker volume ls -qf dangling=true)

# 启动一个挂载数据卷的容器
docker run 
-p 宿主机端口：容器端口
--name 容器名
-d 守护态运行
-v 宿主机目录：容器目录  （挂载数据卷）
镜像名称

（一次 docker run 可以挂载多个数据卷）
```

**附：**

1. 数据卷是被设计用来持久化数据的，它的生命周期独立于容器。

2. `Docker`不会在容器被删除后自动删除数据卷，并且也不存在垃圾回收机制来处理没有任何容器引用的数据卷。

3. 如果需要在删除容器的同时移除数据卷。可以在删除容器的时候使用命令`docker rm -v`

4. 无主的数据卷可能会占据很多空间，要清理请使用以下命令`docker volume prune`

#### 6.2 监听主机目录
- **参数的选择**
```
Docker 新用户应该选择 --mount 参数
推荐使用 --mount 参数
```
- **挂载一个主机目录作为数据卷**
```
docker run -d -P \
    --name web \
    # -v /src/webapp:/opt/webapp \
    --mount type=bind,source=/src/webapp,target=/opt/webapp \
    training/webapp \
    python app.py
```

Docker 挂载主机目录的默认权限是读写，用户也可以通过增加`readonly`指定为只读。
```
$ docker run -d -P \
    --name web \
    # -v /src/webapp:/opt/webapp:ro \
    --mount type=bind,source=/src/webapp,target=/opt/webapp,readonly \
    training/webapp \
    python app.py
],
```
- **挂载一个本地主机文件作为数据卷**
```
docker run --rm -it \
   # -v $HOME/.bash_history:/root/.bash_history \
   --mount type=bind,source=$HOME/.bash_history,target=/root/.bash_history \
   ubuntu:17.10 \
   bash
```
```
root@2affd44b4667:/# history
1  ls
2  diskutil list   
```
这样可以记录在容器输入过的命令。

### 7. Docker Compose
#### 7.1 简介
Docker 官方编排项目之一，负责快速的部署分布式应用，实现对 Docker 容器集群的快速编排。

Compose 有两个重要的概念：

- 服务 (service)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
- 项目 (project)：由一组关联的应用容器组成的一个完整业务单元，在`docker-compose.yml`文件中定义。

Compose 的默认管理对象是项目，通过子命令对项目中的一组容器进行便捷地生命周期管理。

#### 7.2 安装与卸载
- **二进制包**  
```
sudo curl -L https://github.com/docker/compose/releases/download/1.17.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```
```
sudo chmod +x /usr/local/bin/docker-compose
（添加可执行权限）
```
- **PIP安装**  
```
注：x86_64 架构的 Linux 建议按照上边的方法下载二进制包进行安装，如果您计算机的架构是 ARM (例如，树莓派)，再使用 pip 安装。
```
```
sudo pip install -U docker-compose
```
- **bash 补全命令**  
```
curl -L https://raw.githubusercontent.com/docker/compose/1.8.0/contrib/completion/bash/docker-compose
```
- **查看**
```
docker-compose version
```
- **容器中执行**  
```
curl -L https://github.com/docker/compose/releases/download/1.8.0/run.sh > /usr/local/bin/docker-compose
```
```
chmod +x /usr/local/bin/docker-compose
```
- **卸载**  

如果是二进制包方式安装的，删除二进制文件即可。
```
sudo rm /usr/local/bin/docker-compose
```
如果是通过 pip 安装的，则执行如下命令即可删除。
```
sudo pip uninstall docker-compose
```
#### 7.3 命令说明   
- **build**  
```
docker-compose build [options] [SERVICE...]
```
构建（重新构建）项目中的服务容器。

可以随时在项目目录下运行 docker-compose build 来重新构建服务。

选项包括：
```
--force-rm 删除构建过程中的临时容器。  
--no-cache 构建镜像过程中不使用cache（这将加长构建过程）  
--pull 始终尝试通过 pull 来获取更新版本的镜像。
```
- **config**  
```
验证 Compose 文件格式是否正确
```
- **down**
```
停止 up 命令所启动的容器，并移除网络
```
- **exec**
```
进入指定的容器。
```
- **help**
```
获得一个命令的帮助。
```
- **images**
```
列出 Compose 文件中包含的镜像。
```
- **kill**
```
docker-compose kill [options] [SERVICE...]
```
通过发送 SIGKILL 信号来强制停止服务容器。

支持通过 -s 参数来指定发送的信号，例如
```
docker-compose kill -s SIGINT
```
- **logs**
```
docker-compose logs [options] [SERVICE...]
```
查看服务容器的输出。默认情况下，docker-compose 将对不同的服务输出使用不同的颜色来区分。可以通过 --no-color 来关闭颜色。
- **pause**
```
docker-compose pause [SERVICE...]
```
暂停一个服务容器。
- **port**
```
docker-compose port [options] SERVICE PRIVATE_PORT
```

打印某个容器端口所映射的公共端口。

选项：
```
--protocol=proto 指定端口协议，tcp（默认值）或者 udp。
--index=index 如果同一服务存在多个容器，指定命令对象容器的序号（默认为 1）
```
- **ps**
```
docker-compose ps [options] [SERVICE...]。
```
列出项目中目前的所有容器。

选项：
```
-q 只打印容器的 ID 信息
```
- **pull**
```
docker-compose pull [options] [SERVICE...]。
```
拉取服务依赖的镜像。

选项：
```
--ignore-pull-failures 忽略拉取镜像过程中的错误。
```
- **push**
```
推送服务依赖的镜像到 Docker 镜像仓库。
```
- **restart**
```
docker-compose restart [options] [SERVICE...]。
```
重启项目中的服务。

选项：
```
-t, --timeout TIMEOUT  指定重启前停止容器的超时（默认为 10 秒）
```
- **rm**
```
docker-compose rm [options] [SERVICE...]
```
删除所有（停止状态的）服务容器。推荐先执行 docker-compose stop 命令来停止容器。

选项：
```
-f, --force 强制直接删除，包括非停止状态的容器。
-v 删除容器所挂载的数据卷。
```
- **run**
```
docker-compose run [options] [-p PORT...] [-e KEY=VAL...] SERVICE [COMMAND] [ARGS...]
```
在指定服务上执行一个命令。

例如：
```
$ docker-compose run ubuntu ping docker.com
```
将会启动一个 ubuntu 服务容器，并执行 ping docker.com 命令。

默认情况下，如果存在关联，则所有关联的服务将会自动被启动，除非这些服务已经在运行中。

该命令类似启动容器后运行指定的命令，相关卷、链接等等都将会按照配置自动创建。

两个不同点：

给定命令将会覆盖原有的自动运行命令；
不会自动创建端口，以避免冲突。
如果不希望自动启动关联的容器，可以使用 --no-deps 选项，例如
```
$ docker-compose run --no-deps web python manage.py shell
```
将不会启动 web 容器所关联的其它容器。

选项：
```
-d 后台运行容器。
--name NAME 为容器指定一个名字。
--entrypoint CMD 覆盖默认的容器启动指令。
-e KEY=VAL 设置环境变量值，可多次使用选项来设置多个环境变量。
-u, --user="" 指定运行容器的用户名或者 uid。
--no-deps 不自动启动关联的服务容器。
--rm 运行命令后自动删除容器，d 模式下将忽略。
-p, --publish=[] 映射容器端口到本地主机。
--service-ports 配置服务端口并映射到本地主机。
-T 不分配伪 tty，意味着依赖 tty 的指令将无法运行。
```
- **scale**
```
docker-compose scale [options] [SERVICE=NUM...]。
```
设置指定服务运行的容器个数。

通过 service=num 的参数来设置数量。例如：
```
$ docker-compose scale web=3 db=2
```
将启动 3 个容器运行 web 服务，2 个容器运行 db 服务。

一般的，当指定数目多于该服务当前实际运行容器，将新创建并启动容器；反之，将停止容器。

选项：
```
-t, --timeout TIMEOUT 停止容器时候的超时（默认为 10 秒）
```
- **start**
```
docker-compose start [SERVICE...]。
```
启动已经存在的服务容器。

- **stop**
```
docker-compose stop [options] [SERVICE...]。
```
停止处于运行状态的容器。

选项：
```
-t, --timeout TIMEOUT 停止容器时候的超时（默认为 10 秒）
```
- **top**
```
查看各个服务容器内运行的进程。
```
- **unpause**
```
docker-compose unpause [SERVICE...]。
```
恢复处于暂停状态中的服务。
- **up**
```
docker-compose up [options] [SERVICE...]。
```
该命令十分强大，它将尝试自动完成包括构建镜像，（重新）创建服务，启动服务，并关联服务相关容器的一系列操作。

链接的服务都将会被自动启动，除非已经处于运行状态。

**可以说，大部分时候都可以直接通过该命令来启动一个项目。**

默认情况，docker-compose up 启动的容器都在前台，控制台将会同时打印所有容器的输出信息，可以很方便进行调试。
```
Ctrl-C 

停止命令时，所有容器将会停止。
```
```
docker-compose up -d

将会在后台启动并运行所有的容器。一般推荐生产环境下使用该选项。
```

默认情况，如果服务容器已经存在，`docker-compose up` 将会尝试停止容器，然后重新创建（保持使用 volumes-from 挂载的卷），以保证新启动的服务匹配 docker-compose.yml文件的最新内容。   

如果用户不希望容器被停止并重新创建，可以使用 
```
docker-compose up --no-recreate
```
这样将只会启动处于停止状态的容器，而忽略已经运行的服务。

如果用户只想重新部署某个服务，可以使用
```
docker-compose up --no-deps -d <SERVICE_NAME>
```
重新创建服务并后台停止旧服务，启动新服务，并不会影响到其所依赖的服务。

选项：
```
-d 在后台运行服务容器。
--no-color 不使用颜色来区分不同的服务的控制台输出。
--no-deps 不启动服务所链接的容器。
--force-recreate 强制重新创建容器，不能与 --no-recreate 同时使用。
--no-recreate 如果容器已经存在了，则不重新创建，不能与 --force-recreate 同时使用。
--no-build 不自动构建缺失的服务镜像。
-t, --timeout TIMEOUT 停止容器时候的超时（默认为 10 秒）
```
- **version**
```
docker-compose version。
```
打印版本信息。

#### 7.4 模板文件（Compose的核心）

默认的模板文件名称为  

`docker-compose.yml`

- **build**   

指定Dockerfile所在文件夹的路径（可以是绝对路径，或者相对 docker-compose.yml 文件的路径）。Compose 将会利用它自动构建这个镜像，然后使用这个镜像。
```
version: '3'
services:

  webapp:
    build: ./dir
```
**可以：**  

使用 context 指令指定 Dockerfile 所在文件夹的路径。

使用 dockerfile 指令指定 Dockerfile 文件名。

使用 arg 指令指定构建镜像时的变量。
```
version: '3'
services:

  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
```
使用 cache_from 指定构建镜像的缓存
```
build:
  context: .
  cache_from:
    - alpine:latest
    - corp/web_app:3.14
```
- **cap_add, cap_drop**   

指定容器的内核能力（capacity）分配。

例如：

让容器拥有所有能力可以指定为：
```
cap_add:
  - ALL
```
去掉 NET_ADMIN 能力可以指定为：
```
cap_drop:
  - NET_ADMIN
```
- **command**  

覆盖容器启动后默认执行的命令。
```
command: echo "hello world"
```
- **configs**   

仅用于 Swarm mode

- **cgroup_parent**  

指定父 cgroup 组，意味着将继承该组的资源限制。

例如，创建了一个 cgroup 组名称为 cgroups_1
```
cgroup_parent: cgroups_1
```
- **container_name**  

指定容器名称。

默认将会使用 `项目名称_服务名称_序号` 这样的格式。
```
container_name: docker-web-container
```
**注意**: 指定容器名称后，该服务将无法进行扩展（scale），因为 Docker 不允许多个容器具有相同的名称。

- **deploy**

仅用于 Swarm mode

- **devices**  

指定设备映射关系。
```
devices:
  - "/dev/ttyUSB1:/dev/ttyUSB0"
```
- **depends_on**  

解决容器的依赖、启动先后的问题。

以下例子中会先启动 redis db 再启动 web
```
version: '3'

services:
  web:
    build: .
    depends_on:
      - db
      - redis

  redis:
    image: redis

  db:
    image: postgres
```
注意：web 服务不会等待 redis db 「完全启动」之后才启动。

- **dns**  

自定义 DNS 服务器（可以是一个值，也可以是一个列表）
```
dns: 8.8.8.8
```
```
dns:
  - 8.8.8.8
  - 114.114.114.114
```
- **dns_search**  

配置 DNS 搜索域（可以是一个值，也可以是一个列表）
```
dns_search: example.com
```
```
dns_search:
  - domain1.example.com
  - domain2.example.com
```
- **tmpfs**  

挂载一个 tmpfs 文件系统到容器。
```
tmpfs: /run
tmpfs:
  - /run
  - /tmp
```
- **env_file**  

从文件中获取环境变量，可以为单独的文件路径或列表。

如果通过  
```
docker-compose -f FILE
```
指定 Compose 模板文件，则`env_file`中变量的路径会基于模板文件路径。

如果有变量名称与 environment 指令冲突，则按照惯例，以后者为准。
```
env_file: .env
```
```
env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/secrets.env
```
环境变量文件中每一行必须符合格式，支持 # 开头的注释行。
```
# common.env: Set development environment
PROG_ENV=development
```
- **environment**  

设置环境变量。

可以使用数组或字典两种格式。

只给定名称的变量会自动获取运行 Compose 主机上对应变量的值，可以用来防止泄露不必要的数据。
```
environment:
  RACK_ENV: development
  SESSION_SECRET:
```
```
environment:
  - RACK_ENV=development
  - SESSION_SECRET
```
如果变量名称或者值中用到表达布尔含义的词汇，最好放到引号里，避免YAML自动解析某些内容为对应的布尔语义。

这些特定词汇，包括
```
y|Y|yes|Yes|YES|n|N|no|No|NO|true|True|TRUE|false|False|FALSE|on|On|ON|off|Off|OFF
```
- **expose**  

暴露端口，但不映射到宿主机，只被连接的服务访问。

仅可以指定内部端口为参数
```
expose:
 - "3000"
 - "8000"
```
- **external_links**  

==注意==：不建议使用该指令。

链接到 docker-compose.yml 外部的容器，甚至并非 Compose 管理的外部容器。
```
external_links:
 - redis_1
 - project_db_1:mysql
 - project_db_1:postgresql
```
- **extra_hosts**  

类似 Docker 中的`--add-host`参数，指定额外的 host 名称映射信息。
```
extra_hosts:
 - "googledns:8.8.8.8"
 - "dockerhub:52.1.157.61"
```
会在启动后的服务容器中`/etc/hosts`文件中添加如下内容：
```
8.8.8.8 googledns
52.1.157.61 dockerhub
```
- **healthcheck**  

通过命令检查容器是否健康运行。
```
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
```
- **image**  

指定为镜像名称或镜像ID。如果镜像在本地不存在，Compose 将会尝试拉取这个镜像。
```
image: ubuntu
image: orchardup/postgresql
image: a4bc65fd
```
- **labels**  

为容器添加Docker元数据（metadata）信息。

例如可以为容器添加辅助说明信息。
```
labels:
  com.startupteam.description: "webapp for a startup team"
  com.startupteam.department: "devops department"
  com.startupteam.release: "rc3 for v1.0"
```

- **logging**  

配置日志选项。
```
logging:
  driver: syslog
  options:
    syslog-address: "tcp://192.168.0.42:123"
```
目前支持三种日志驱动类型。
```
driver: "json-file"
driver: "syslog"
driver: "none"
```
options 配置日志驱动的相关参数。
```
options:
  max-size: "200k"
  max-file: "10"
```
- **network_mode**

设置网络模式。 
```
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```
- **networks**

配置容器连接的网络。
```
version: "3"
services:

  some-service:
    networks:
     - some-network
     - other-network

networks:
  some-network:
  other-network:
```
- **pid**  

跟主机系统共享进程命名空间。打开该选项的容器之间，以及容器和宿主机系统之间可以通过进程ID来相互访问和操作。
```
pid: "host"
```
- **ports**

暴露端口信息。

使用 `宿主端口：容器端口` `(HOST:CONTAINER) `格式

或者仅仅指定容器的端口（宿主将会随机选择端口）都可以。
```
ports:
 - "3000"
 - "8000:8000"
 - "49100:22"
 - "127.0.0.1:8001:8001"
```
注意：当使用`HOST:CONTAINER`格式来映射端口时，如果你使用的容器端口小于60并且没放到引号里，可能会得到错误结果，因为YAML会自动解析`xx:yy`这种数字格式为60进制。为避免出现这种问题，**建议数字串都采用引号包括起来的字符串格式。**

- **secrets**

存储敏感数据，例如 mysql 服务密码。
```
version: "3.1"
services:

mysql:
  image: mysql
  environment:
    MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_root_password
  secrets:
    - db_root_password
    - my_other_secret

secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```
- **security_opt**

指定容器模板标签（label）机制的默认属性（用户、角色、类型、级别等）。
```
security_opt:
    - label:user:USER
    - label:role:ROLE
```
- **stop_signal**

设置另一个信号来停止容器。

在默认情况下使用的是 SIGTERM 停止容器。
```
stop_signal: SIGUSR1
```
- **sysctls**

配置容器内核参数。
```
sysctls:
  net.core.somaxconn: 1024
  net.ipv4.tcp_syncookies: 0

sysctls:
  - net.core.somaxconn=1024
  - net.ipv4.tcp_syncookies=0
```
- **ulimits**

指定容器的 ulimits 限制值。

例如，指定  
最大进程数 65535  
文件句柄数 20000（软限制，应用可以随时修改，不能超过硬限制）   
文件句柄数 40000（系统硬限制，只能 root 用户提高）
```
  ulimits:
    nproc: 65535
    nofile:
      soft: 20000
      hard: 40000
```
- **volumes**

数据卷所挂载路径设置（支持相对路径）。

可以设置宿主机路径`（HOST:CONTAINER）` 

或加上访问模式`（HOST:CONTAINER:ro）`
```
volumes:
 - /var/lib/mysql
 - cache/:/tmp/cache
 - ~/configs:/etc/configs/:ro
```
- **其它指令**

此外  
`domainname` `entrypoint` `hostname` `ipc` `mac_address` `privileged` `read_only` `shm_size` `restart` `stdin_open` `tty, user` `working_dir` 等指令，基本跟 docker run 中对应参数的功能一致。

指定服务容器启动后执行的入口文件：
```
entrypoint: /code/entrypoint.sh
```
指定容器中运行应用的用户名：
```
user: nginx
```
指定容器中工作目录：
```
working_dir: /code
```
指定容器中搜索域名、主机名、mac 地址：
```
domainname: your_website.com
hostname: test
mac_address: 08-00-27-00-0C-0A
```
允许容器中运行一些特权命令：
```
privileged: true
```
指定容器退出后的重启策略为始终重启。该命令对保持服务始终运行十分有效，在生产环境中推荐配置为`always`或者`unless-stopped`
```
restart: always
```
以只读模式挂载容器的root文件系统，意味着不能对容器内容进行修改：
```
read_only: true
```
打开标准输入，可以接受外部输入：
```
stdin_open: true
```
模拟一个伪终端：
```
tty: true
```
- **读取变量**

Compose 模板文件支持动态读取主机的系统环境变量和当前目录下的 .env 文件中的变量。

例如，下面的 Compose 文件将从运行它的环境中读取变量`${MONGO_VERSION}`的值，并写入执行的指令中。
```
version: "3"
services:

db:
  image: "mongo:${MONGO_VERSION}"
```
如果执行`MONGO_VERSION=3.2 docker-compose up`则会启动一个`mongo:3.2`镜像的容器；

如果执行`MONGO_VERSION=2.8 docker-compose up`则会启动一个 mongo:2.8 镜像的容器。

若当前目录存在 .env 文件，执行 docker-compose 命令时将从该文件中读取变量。

在当前目录新建 .env 文件并写入以下内容。
```
# 支持 # 号注释
MONGO_VERSION=3.6
```
执行 docker-compose up 则会启动一个 mongo:3.6 镜像的容器。

#### 7.5 注意事项

**需要配置主机名，主机IP，DNS，修改cloud.cfg文件**

详细步骤请参考文档`容器集群的部署`中`创建镜像虚拟机`的相关内容。



#### 7.6 常用命令
```
# 前台运行
docker-compose up

# 后台运行
docker-compose up -d

# 启动
docker-compose start

# 停止
docker-compose stop

# 停止并移除容器
docker-compose down
```

### 8. 网络配置
```
待完善
```

### 9. 安全
```
待完善
```

### 10. 底层实现
```

```

### 11. 附录
#### 11.1 命令查询
- **客户端命令选项**
```
--config=""：指定客户端配置文件，默认为 `/.docker`；
-D=true|false：是否使用 debug 模式。默认不开启；
-H, --host=[]：指定命令对应 Docker 守护进程的监听接口，可以为 unix 套接字（unix:///path/to/socket），文件句柄（fd://socketfd）或 tcp 套接字（tcp://[host[:port]]），默认为 unix:///var/run/docker.sock；
-l, --log-level="debug|info|warn|error|fatal"：指定日志输出级别；
--tls=true|false：是否对 Docker 守护进程启用 TLS 安全机制，默认为否；
--tlscacert= /.docker/ca.pem：TLS CA 签名的可信证书文件路径；
--tlscert= /.docker/cert.pem：TLS 可信证书文件路径；
--tlscert= /.docker/key.pem：TLS 密钥文件路径；
--tlsverify=true|false：启用 TLS 校验，默认为否。
```
- **dockerd 命令选项**
```
--api-cors-header=""：CORS 头部域，默认不允许 CORS，要允许任意的跨域访问，可以指定为 “*”；
--authorization-plugin=""：载入认证的插件；
-b=""：将容器挂载到一个已存在的网桥上。指定为 'none' 时则禁用容器的网络，与 --bip 选项互斥；
--bip=""：让动态创建的 docker0 网桥采用给定的 CIDR 地址; 与 -b 选项互斥；
--cgroup-parent=""：指定 cgroup 的父组，默认 fs cgroup 驱动为 `/docker`，systemd cgroup 驱动为 `system.slice`；
--cluster-store=""：构成集群（如 Swarm）时，集群键值数据库服务地址；
--cluster-advertise=""：构成集群时，自身的被访问地址，可以为 `host:port` 或 `interface:port`；
--cluster-store-opt=""：构成集群时，键值数据库的配置选项；
--config-file="/etc/docker/daemon.json"：daemon 配置文件路径；
--containerd=""：containerd 文件的路径；
-D, --debug=true|false：是否使用 Debug 模式。缺省为 false；
--default-gateway=""：容器的 IPv4 网关地址，必须在网桥的子网段内；
--default-gateway-v6=""：容器的 IPv6 网关地址；
--default-ulimit=[]：默认的 ulimit 值；
--disable-legacy-registry=true|false：是否允许访问旧版本的镜像仓库服务器；
--dns=""：指定容器使用的 DNS 服务器地址；
--dns-opt=""：DNS 选项；
--dns-search=[]：DNS 搜索域；
--exec-opt=[]：运行时的执行选项；
--exec-root=""：容器执行状态文件的根路径，默认为 `/var/run/docker`；
--fixed-cidr=""：限定分配 IPv4 地址范围；
--fixed-cidr-v6=""：限定分配 IPv6 地址范围；
-G, --group=""：分配给 unix 套接字的组，默认为 `docker`；
-g, --graph=""：Docker 运行时的根路径，默认为 `/var/lib/docker`；
-H, --host=[]：指定命令对应 Docker daemon 的监听接口，可以为 unix 套接字（unix:///path/to/socket），文件句柄（fd://socketfd）或 tcp 套接字（tcp://[host[:port]]），默认为 unix:///var/run/docker.sock；
--icc=true|false：是否启用容器间以及跟 daemon 所在主机的通信。默认为 true。
--insecure-registry=[]：允许访问给定的非安全仓库服务；
--ip=""：绑定容器端口时候的默认 IP 地址。缺省为 0.0.0.0；
--ip-forward=true|false：是否检查启动在 Docker 主机上的启用 IP 转发服务，默认开启。注意关闭该选项将不对系统转发能力进行任何检查修改；
--ip-masq=true|false：是否进行地址伪装，用于容器访问外部网络，默认开启；
--iptables=true|false：是否允许 Docker 添加 iptables 规则。缺省为 true；
--ipv6=true|false：是否启用 IPv6 支持，默认关闭；
-l, --log-level="debug|info|warn|error|fatal"：指定日志输出级别；
--label="[]"：添加指定的键值对标注；
--log-driver="json-file|syslog|journald|gelf|fluentd|awslogs|splunk|etwlogs|gcplogs|none"：指定日志后端驱动，默认为 json-file；
--log-opt=[]：日志后端的选项；
--mtu=VALUE：指定容器网络的 mtu；
-p=""：指定 daemon 的 PID 文件路径。缺省为 `/var/run/docker.pid`；
--raw-logs：输出原始，未加色彩的日志信息；
--registry-mirror=<scheme>://<host>：指定 `docker pull` 时使用的注册服务器镜像地址；
-s, --storage-driver=""：指定使用给定的存储后端；
--selinux-enabled=true|false：是否启用 SELinux 支持。缺省值为 false。SELinux 目前尚不支持 overlay 存储驱动；
--storage-opt=[]：驱动后端选项；
--tls=true|false：是否对 Docker daemon 启用 TLS 安全机制，默认为否；
--tlscacert= /.docker/ca.pem：TLS CA 签名的可信证书文件路径；
--tlscert= /.docker/cert.pem：TLS 可信证书文件路径；
--tlscert= /.docker/key.pem：TLS 密钥文件路径；
--tlsverify=true|false：启用 TLS 校验，默认为否；
--userland-proxy=true|false：是否使用用户态代理来实现容器间和出容器的回环通信，默认为 true；
--userns-remap=default|uid:gid|user:group|user|uid：指定容器的用户命名空间，默认是创建新的 UID 和 GID 映射到容器内进程。
```
- **客户端命令**
```
attach：依附到一个正在运行的容器中；
build：从一个 Dockerfile 创建一个镜像；
commit：从一个容器的修改中创建一个新的镜像；
cp：在容器和本地宿主系统之间复制文件中；
create：创建一个新容器，但并不运行它；
diff：检查一个容器内文件系统的修改，包括修改和增加；
events：从服务端获取实时的事件；
exec：在运行的容器内执行命令；
export：导出容器内容为一个 tar 包；
history：显示一个镜像的历史信息；
images：列出存在的镜像；
import：导入一个文件（典型为 tar 包）路径或目录来创建一个本地镜像；
info：显示一些相关的系统信息；
inspect：显示一个容器的具体配置信息；
kill：关闭一个运行中的容器 (包括进程和所有相关资源)；
load：从一个 tar 包中加载一个镜像；
login：注册或登录到一个 Docker 的仓库服务器；
logout：从 Docker 的仓库服务器登出；
logs：获取容器的 log 信息；
network：管理 Docker 的网络，包括查看、创建、删除、挂载、卸载等；
node：管理 swarm 集群中的节点，包括查看、更新、删除、提升/取消管理节点等；
pause：暂停一个容器中的所有进程；
port：查找一个 nat 到一个私有网口的公共口；
ps：列出主机上的容器；
pull：从一个Docker的仓库服务器下拉一个镜像或仓库；
push：将一个镜像或者仓库推送到一个 Docker 的注册服务器；
rename：重命名一个容器；
restart：重启一个运行中的容器；
rm：删除给定的若干个容器；
rmi：删除给定的若干个镜像；
run：创建一个新容器，并在其中运行给定命令；
save：保存一个镜像为 tar 包文件；
search：在 Docker index 中搜索一个镜像；
service：管理 Docker 所启动的应用服务，包括创建、更新、删除等；
start：启动一个容器；
stats：输出（一个或多个）容器的资源使用统计信息；
stop：终止一个运行中的容器；
swarm：管理 Docker swarm 集群，包括创建、加入、退出、更新等；
tag：为一个镜像打标签；
top：查看一个容器中的正在运行的进程信息；
unpause：将一个容器内所有的进程从暂停状态中恢复；
update：更新指定的若干容器的配置信息；
version：输出 Docker 的版本信息；
volume：管理 Docker volume，包括查看、创建、删除等；
wait：阻塞直到一个容器终止，然后输出它的退出符。
```
#### 11.2 资源链接
- **官方网站**
```
Docker 官方主页：https://www.docker.com
Docker 官方博客：https://blog.docker.com/
Docker 官方文档：https://docs.docker.com/
Docker Store：https://store.docker.com
Docker Cloud：https://cloud.docker.com
Docker Hub：https://hub.docker.com
Docker 的源代码仓库：https://github.com/moby/moby
Docker 发布版本历史：https://docs.docker.com/release-notes/
Docker 常见问题：https://docs.docker.com/engine/faq/
Docker 远端应用 API：https://docs.docker.com/develop/sdk/
```
- **实践参考**
```
Dockerfile 参考：https://docs.docker.com/engine/reference/builder/
Dockerfile 最佳实践：https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/
```
- **技术交流**
```
Docker 邮件列表： https://groups.google.com/forum/#!forum/docker-user
Docker 的 IRC 频道：https://chat.freenode.net#docker
Docker 的 Twitter 主页：https://twitter.com/docker
```
- **其它**
```
Docker 的 StackOverflow 问答主页：https://stackoverflow.com/search?q=docker
```