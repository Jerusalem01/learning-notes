## 基于Docker的部署

### 一. GitLab-代码管理平台

#### 1. 基于Docker安装（服务端）

- **docker-compose.yml**

```
version: '3'
services:
    web:
      image: 'twang2218/gitlab-ce-zh:10.5'
      container_name: GitLab
      restart: always
      hostname: '192.168.137.130'
      environment:
        TZ: 'Asia/Shanghai'
        GITLAB_OMNIBUS_CONFIG: |
          external_url 'http://192.168.137.130:8080'   # 外部IP，访问时用到
          gitlab_rails['gitlab_shell_ssh_port'] = 2222
          unicorn['port'] = 8888   # 内部端口，不用管
          nginx['listen_port'] = 8080
      ports:
        - '8080:8080'
        - '8443:443'   # 安全连接
        - '2222:22'
      volumes:
        - /usr/local/docker/gitlab/config:/etc/gitlab
        - /usr/local/docker/gitlab/data:/var/opt/gitlab
        - /usr/local/docker/gitlab/logs:/var/log/gitlab
```

**注意：**

1. 若虚拟机的内存低于`2G`，`GitLab`无法运行。
2. 最好放于固态硬盘，否则卡爆你。

#### 2.登录

```
1. 访问地址：http://192.168.137.130:8080

2. 设置管理员初始密码(至少8位)，管理员账号是 root
```

#### 3.基本设置

- **账户与限制设置**

```
关闭头像功能
由于 Gravatar 头像为网络头像，在网络情况不理想时可能导致访问时卡顿
```

- **注册限制**

```
关闭注册功能
由于是内部代码托管服务器，由管理员统一创建用户即可
```

#### 4. Git 的使用

- **CMD操作方法命令**

```
# 拉取代码
在本地放代码的目录下cmd
git clone HTTP地址

# 修改代码
在本地放代码的目录下cmd
git commit -m "日志"

# 推送代码
在本地放代码的目录下cmd
git push
```

- **使用 SSH 的方式拉取和推送项目**

```
# 生成 SSH KEY

1.使用 ssh-keygen 工具生成
位置：D:\Git\Git\usr\bin

2.输入命令：

ssh-keygen -t rsa -C "ssh-keygen -t rsa -C "3276586184@qq.com""

执行成功后的效果：

ssh-keygen -t rsa -C "3276586184@qq.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/Lusifer/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/Lusifer/.ssh/id_rsa.
Your public key has been saved in /c/Users/Lusifer/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:cVesJKa5VnQNihQOTotXUAIyphsqjb7Z9lqOji2704E topsale@vip.qq.com
The key's randomart image is:
+---[RSA 2048]----+
|  + ..=o=.  .+.  |
| o o + B .+.o.o  |
|o   . + +=o+..   |
|.=   .  oo...    |
|= o     So       |
|oE .    o        |
| .. .. .         |
| o*o+            |
| *B*oo           |
+----[SHA256]-----+
# 复制 SSH-KEY 信息到 GitLab

秘钥位置：C:\Users\用户名\.ssh

1.找到 id_rsa.pub 并使用编辑器打开

2.登录 GitLab，填写“SSH 密钥”
# SSH 客户端的修改

位置：D:\Git\Git\usr\bin\ssh.exe
# 使用 TortoiseGit 克隆项目

1.新建一个存放代码仓库的本地文件夹

2.右键点击“Git 克隆”

3.服务项目地址到 URL（SSH、HTTP）
# 使用 TortoiseGit 推送项目（提交代码）

1.右键点击“Git 提交”

2.点击“全部”并填入“日志信息”

3.点击“提交并推送”
```

### 二. Nexus-依赖管理平台

#### 1.基于 Docker 安装（服务端）

- **docker-compose.yml**

```
version: '3.1'
services:
  nexus:
    restart: always
    image: sonatype/nexus3
    container_name: nexus
    ports:
      - 8081:8081
    volumes:
      - /usr/local/docker/nexus/data:/nexus-data
注： 启动时如果出现权限问题可以使用：chmod 777 /usr/local/docker/nexus/data 赋予数据卷目录可读可写的权限
```

- **登录控制台验证安装**

```
地址：http://ip:port/ 

用户名：admin 密码：admin123
```

#### 2.在项目中使用 Maven 私服

- **配置认证信息**

在 Maven `settings.xml`中添加`Nexus`认证信息(`servers`节点下)：

```
<server>
  <id>nexus-releases</id>
  <username>admin</username>
  <password>admin123</password>
</server>

<server>
  <id>nexus-snapshots</id>
  <username>admin</username>
  <password>admin123</password>
</server>
```

**Snapshots 与 Releases 的区别：**

1. nexus-releases: 用于发布 Release 版本
   nexus-snapshots: 用于发布 Snapshot 版本（快照版）
2. Release 版本与 Snapshot 定义如下：

```
Release: 1.0.0/1.0.0-RELEASE
Snapshot: 1.0.0-SNAPSHOT
```

1. SNAPSHOT 版本会自动加一个时间作为标识

`1.0.0-SNAPSHOT`发布后为变成

```
1.0.0-SNAPSHOT-20180522.123456-1.jar
```

- **自动化部署**

在 pom.xml 中添加如下代码：

```
<!-- 上传依赖 -->
<distributionManagement>  
  <repository>  
    <id>nexus-releases</id>  
    <name>Nexus Release Repository</name>  
    <url>http://192.168.137.134:8081/repository/maven-releases/</url>  
  </repository>  
  <snapshotRepository>  
    <id>nexus-snapshots</id>  
    <name>Nexus Snapshot Repository</name>  
    <url>http://192.168.137.134:8081/repository/maven-snapshots/</url>  
  </snapshotRepository>  
</distributionManagement> 
```

**注意事项：**

1. ID 名称必须要与`settings.xml`中`Servers`配置的 ID 名称保持一致。
2. 项目版本号中有`SNAPSHOT`标识的，会发布到`Nexus Snapshots Repository`, 否则发布到`Nexus Release Repository`，并根据 ID 去匹配授权账号。

- **部署到仓库**

IDEA控制台：

```
mvn deploy
mvn deploy -Dmaven.test.skip=true
```

- **IDEA Settings**

```
总是使用最新的快照版依赖
```

- **上传第三方 JAR 包**

CMD 中使用 Maven 命令：

```
# 如第三方 JAR 包：aliyun-sdk-oss-2.2.3.jar
mvn deploy:deploy-file 
  -DgroupId=com.aliyun.oss 
  -DartifactId=aliyun-sdk-oss 
  -Dversion=2.2.3 
  -Dpackaging=jar 
  -Dfile=D:\aliyun-sdk-oss-2.2.3.jar 
  -Durl=http://127.0.0.1:8081/repository/maven-3rd/ 
  -DrepositoryId=nexus-releases
```

**注意事项：**

1. 建议在上传第三方`JAR`包时，创建单独的第三方`JAR`包管理仓库，便于管理有维护。（maven-3rd）
2. `-DrepositoryId=nexus-releases`对应的是`settings.xml`中`Servers`配置的 ID 名称。（授权）

- **配置代理仓库**

```
<!-- 拉取依赖 -->
<repositories>
    <repository>
        <id>nexus</id>
        <name>Nexus Repository</name>
        <url>http://192.168.137.134:8081/repository/maven-public/</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
        <releases>
            <enabled>true</enabled>
        </releases>
    </repository>
</repositories>
<pluginRepositories>
    <pluginRepository>
        <id>nexus</id>
        <name>Nexus Plugin Repository</name>
        <url>http://192.168.137.134:8081/repository/maven-public/</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
        <releases>
            <enabled>true</enabled>
        </releases>
    </pluginRepository>
</pluginRepositories>
```

**总结一下使用私服的好处：**

1. 官服依赖下载到私服，加快本地开发下载依赖的速度。
2. 协同开发过程中，快照版本，及时更新代码。
3. 官服没有的jar包，可手动上传到私服，以便后期快速拉取使用。

**实际开发时下载依赖的流程：**

首先

项目本地仓库

本地仓库没有

本地仓库私服

私服没有

私服官服

### 三. Registry-镜像管理平台

#### 1. 安装 Docker Registry 私服(服务端)

- **docker-compose.yml**

```
version: '3.1'
services:
  registry:
    image: registry
    restart: always
    container_name: registry
    ports:
      - 5000:5000
    volumes:
      - /usr/local/docker/registry/data:/var/lib/registry
```

- **测试**

1. 浏览器端访问

```
http://192.168.137.134:5000/v2/
```

1. 终端访问

```
curl http://192.168.137.134:5000/v2/
```

#### 2. 配置 Docker Registry 客户端

- **配置**

在`/etc/docker/daemon.json`中增加如下内容

```
{
  "registry-mirrors": [
    "https://wzwr26i4.mirror.aliyuncs.com"
  ],
  "insecure-registries": [
    "192.168.137.134:5000"
  ]
}
```

- **重启服务**

```
sudo systemctl daemon-reload
sudo systemctl restart docker
```

- **检查客户端配置是否生效**

```
docker info
```

显示如下内容，说明配置成功

```
Insecure Registries:
 192.168.137.135:5000
 127.0.0.0/8
```

**注意：**

服务端与客户端是分离的，即两台虚拟机。

- **部署 Docker Registry WebUI（可视化界面）**

1. docker-registry-web
2. docker-registry-frontend

下面以`docker-registry-fontend`为例

**docker-compose.yml**

```
version: '3.1'
services:
  frontend:
    image: konradkleine/docker-registry-frontend:v2
    ports:
      - 8080:80
    volumes:
      - ./certs/frontend.crt:/etc/apache2/server.crt:ro
      - ./certs/frontend.key:/etc/apache2/server.key:ro
    environment:
      - ENV_DOCKER_REGISTRY_HOST=192.168.137.134
      - ENV_DOCKER_REGISTRY_PORT=5000
```

运行成功后在浏览器访问：

```
http://192.168.137.134:8080
```

#### 3. 基本操作命令

```
# 拉取镜像
docker pull xxx

# 查看全部镜像
docker images

# 标记本地镜像并指向目标仓库
docker tag 镜像名 192.168.137.134:5000/镜像名
（ip:port/image_name:tag，该格式为标记版本号）

# 提交镜像到仓库
docker push 192.168.137.134:5000/镜像名

# 查看全部镜像
终端：curl -XGET http://192.168.137.134:5000/v2/_catalog
浏览器： http://192.168.137.134:5000/v2/_catalog

# 查看指定镜像
终端：curl -XGET http://192.168.137.134:5000/v2/镜像名/tags/list
浏览器：http://192.168.137.134:5000/v2/镜像名/tags/list

# 删除镜像
docker rmi 镜像名
docker rmi 192.168.137.134:5000/镜像名

# 拉取镜像
docker pull 192.168.137.134:5000/镜像名
```

### 四. MySQL-数据库

- **docker-compose.yml**

**MySQL5**

```
version: '3.1'
services:
  mysql:
    restart: always
    image: mysql:5.7.22
    container_name: mysql
    ports:
      - 3306:3306
    environment:
      TZ: Asia/Shanghai
      MYSQL_ROOT_PASSWORD: 123456
    command:
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
      --lower_case_table_names=1
      --max_allowed_packet=128M
      --sql-mode="STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION,NO_ZERO_DATE,NO_ZERO_IN_DATE,ERROR_FOR_DIVISION_BY_ZERO"
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```

**MySQL8**

```
version: '3.1'
services:
  db:
    image: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 123456
    command:
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
      --lower_case_table_names=1
    ports:
      - 3306:3306
    volumes:
      - ./data:/var/lib/mysql

  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080
```

**命令参数说明：**

```
-p 3306:3306：将容器的3306端口映射到主机的3306端口

-v /usr/local/docker/mysql/conf:/etc/mysql：将主机当前目录下的 conf 挂载到容器的 /etc/mysql

-v /usr/local/docker/mysql/logs:/var/log/mysql：将主机当前目录下的 logs 目录挂载到容器的 /var/log/mysql

-v /usr/local/docker/mysql/data:/var/lib/mysql：将主机当前目录下的 data 目录挂载到容器的 /var/lib/mysql

-e MYSQL\_ROOT\_PASSWORD=123456：初始化root用户的密码
```

- **使用客户端工具连接 MySQL**

**注意：**

应该将`MySQL`数据库的配置信息做成数据卷，放在目录

`/usr/local/docker/mysql/conf`目录下，以便持久化使用。

具体的操作步骤可观看视频：

```
https://www.bilibili.com/video/av45705941?p=41
```

### 五. Tomcat-Web应用服务器

- **docker-compose**

```
version: '3.1'
services:
  tomcat:
    restart: always
    image: tomcat
    container_name: tomcat
    ports:
      - 8080:8080
    volumes:
      - /usr/local/docker/tomcat/webapps/test:/usr/local/tomcat/webapps/test
    environment:
      TZ: Asia/Shanghai
```