## 聚焦Java性能优化，打造亿级流量秒杀系统

![](C:\Users\86157\AppData\Roaming\Typora\typora-user-images\image-20201204143633860.png)

### 部署生产环境

1. 本地在项目根目录下使用mvn clean package打包生成seckill-1.0.0-SNAPSHOT.jar文件

2. 将jar包服务上传到服务端上并编写额外的application.properties配置文件

3. 编写deploy.sh脚本文件启动对应的项目

   ```
   source /etc/profile;		//使java环境生效
   nohup java -Xms2048m -Xmx2048m  -XX:NewSize=1024m -XX:MaxNewSize=1024m -jar seckill-1.0.0-SNAPSHOT.jar --spring.config.addition-location=/usr/local/seckill/application.properties
   ```

4. 授权并执行使用

   ```
   chmod 777 deploy.sh
   
   ./deploy.sh &
   ```

   启动应用程序

   查看日志

   ```
   tail -f nohup.out
   ```

5. 打开阿里云的网络安全组配置，将80端口开放给外网可访问

**参数说明**

- nohup：以非停止方式运行程序，这样即便控制台退出了程序也不会停止
- java：java命令启动，设置jvm初始和最大内存为2048m，2个g大小，设置jvm中初始新生代和最大新生代大小为1024m，设置成一样的目的是为了减少扩展jvm内存池过程中向操作系统索要内存分配的消耗
- –-spring.config.addtion-location=：指定额外的配置文件地址

### 性能压测 ApacheJMeter

##### 1. 前置知识

##### 2. 组件

- 线程组
- Http请求
- 察看结果树
- 聚合报告

##### 3. 常用操作命令

```
查看服务器性能
top -H

查看某个进程
ps -ef | grep java

通过端口查看进程
netstat -anp | grep 5240

查看某进程的线程
pstree -p [progressId] wc -l

统计某进程的线程数量
pstree -p [progressId] wc -l

杀死进程
kill 5240
```





### 1.  解决容量问题

#### 1.1 服务端并发容量上不去

##### 1.1.1 修改Spring Boot内嵌Tomcat默认配置（spring-configuration-metadata.json ）

```
server.tomcat.accept-count:等待队列长度，默认100 
server.tomcat.max-connections:最大可被连接数，默认10000 
server.tomcat.max-threads:最大工作线程数，默认200 
server.tomcat.min-spare-threads:最小工作线程数，默认10

默认配置下，链接超10000后出现拒绝链接情况 
默认配置下，发出的请求超过200+100后拒绝处理
```

修改配置如下

```
server.tomcat.accept-count: 1000
server.tomcat.max-connections: 10000 
server.tomcat.max-threads: 800
server.tomcat.min-spare-threads: 10000

对4核8G的服务器来说，经验上最好的max-threads是800
```

##### 1.1.2 定制化内嵌Tomcat配置定制化内嵌Tomcat配置

```
@Component
public class WebServerConfiguration implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {
    @Override
    public void customize(ConfigurableWebServerFactory configurableWebServerFactory) {
         //使用对应工厂类提供给我们的接口定制化我们的tomcat connector
        ((TomcatServletWebServerFactory)configurableWebServerFactory).addConnectorCustomizers(new TomcatConnectorCustomizer() {
            @Override
            public void customize(Connector connector) {
                Http11NioProtocol protocol = (Http11NioProtocol) connector.getProtocolHandler();
                //定制化keepalivetimeout(设置30秒内没有请求则服务端自动断开keepalive链接)
                protocol.setKeepAliveTimeout(30000);
                //当客户端发送超过10000个请求则自动断开keepalive链接
                protocol.setMaxKeepAliveRequests(10000);
            }
        });
    }
}
```

**说明：**

- keepAliveTimeOut：多少毫秒后不响应就断开keepAlive
- maxKeepAliveRequests：多少次请求后keepAlive断开失效
- KeepAlive ：建立长连接，保护系统不受客户端连接的拖累，减少网络消耗

#### 1.2  响应时间长，TPS上不去

##### 1.2.1 单web容器上限

- 线程数量：4核8G的单进程调度线程数最好是800，超过1000后即花费巨大的时间在cpu调度上
- 等待队列长度：队列做缓冲池用，但不能无限长，消耗内存，出对入队消耗cpu（一般1000-2000）

##### 1.2.2  Mysql数据库QPS容量问题

- 主键查询：千万级别数据 = 1-10毫秒。 
- 唯一索引查询： 千万级别数据= 10-100毫秒 
- 非唯一索引查询： 千万级别数据= 100-1000毫秒 
- 无索引： 百万条数据= 1000毫秒 +

数据库查询尽量使用到主键查询和唯一索引查询，如果非唯一索引查询在数据达到一定数量级后，则要进行分表分库来优化。

##### 1.2.3  单机容量问题

- 表象：单机cpu使用率增高，memory占用增加，网络带宽使用增加
- cpu us：用户空间的Cpu使用情况
- cpu sy：内核空间的cpu使用情况
- load average：1，5，15分钟load平均值，跟着核数系数，0代表通常，1代表打满，1+代表阻塞
- memory：free空闲内存，used使用内存

### 2. 分布式扩展（服务端水平对称部署）

#### 2.1 引入Nginx实现反向代理、负载均衡

##### 2.1.1 部署使用OpenResty作为Nginx框架

如果对Nginx开发有特殊要求或者OpenResty的Nginx达不到我们的需求，可以到Nginx官网下载，操作之后得到所需的Nginx替换掉openResty/sbin目录下的nginx即可。

```
1.先行条件，需要在linux安装pcre，openssl，gcc，curl等
apt install PCRE
apt install pcre-devel openssl-devel gcc curl

2.下载openresty 下载页面 http://openresty.org/cn/download.html

3.上传并解压
tar -xvzf openresty**.tar.gz

4.解压后执行如下命令
./configure
make
make install
安装完成，nginx默认安装在 /usr/local/openresty/nginx目录下
修改本地和阿里云服务器的host路径，以便于统一访问
```

##### 2.1.2  将Nginx作为Web服务器

- location节点path：指定url映射key
- location节点内容：root指定location path后对应的根路径，index指定默认的访问页
- sbin/nginx -c conf/nginx.conf启动
- 修改配置后直接sbin/nginx -s reload无缝重启

```
1.静态资源部署
进入nginx根目录下的html下，新建resources目录用于存放前端静态资源
设置指向resources目录下的location可以访问对应的html下的静态资源文件
```

![](E:\程序人生\个人学习笔记\学习笔记图床\QQ图片20201204183955.png)

##### 2.1.3 将Nginx作为动静分离服务器

- location节点path特定resources：静态资源路径
- location节点其他路径：动态请求

##### 2.1.4 将Nginx作为反向代理服务器

- 设置upStream sserver

  反向代理配置，配置一个backend server，可以用于指向后端不同的server集群，配置内容为server集群的局域网ip，以及轮训的权重值，并且配置一个location，当访问规则命中location任何一个规则的时候则可以进入反向代理规则

```
    upstream backend_server{
        server 192.168.75.180 weight=1;
        server 192.168.75.181 weight=1;
    }
```

- 设置动态请求location为proxy pass路径

```
    location / {
         proxy_pass http://backend_server;
         #设置请求头
         proxy_set_header Host $http_host:$proxy_port;
         proxy_set_header X-Real-IP $remote_addr;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
```

- 开启tomcat access log认证

```
#日志开关
server.tomcat.accesslog.enabled=true
#日志格式
server.tomcat.accesslog.pattern=%h %l %u %t "%r" %s %b %D
server.tomcat.accesslog.directory=/usr/local/seckill/logs
```

- 设置Nginx与应用服务器的长连接

```
    upstream backend_server{
        server 192.168.75.180 weight=1;
        server 192.168.75.181 weight=1;
        keepalive_timeout 30;			//添加此行
    }
```

```
    location / {
         proxy_pass http://backend_server;
         #设置请求头
         proxy_set_header Host $http_host:$proxy_port;
         proxy_set_header X-Real-IP $remote_addr;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
         # 添加如下两行
         proxy_http_version 1.1;
         #允许重新定义或追加字段到传递给代理服务器的请求头信息（默认是close）
         proxy_set_header Connection "";
    }
```

##### 2.1.5 Nginx高性能原因

- epoll多路复用
  - java bio模型 - 阻塞式进程
  - linux select模型 - 变更触发轮询查找，有1024数量上限
  - **epoll模型 - 变更触发回调直接读取，理论上无上限**

- master-worker进程模型

![](E:\程序人生\个人学习笔记\学习笔记图床\master-worker模型.png)

- 协程机制
  - 依附于线程的内存模型，切换开销小
  - 遇阻塞及归还执行权，代码同步
  - 无需加锁

#### 2.2 引入Redis实现分布式会话管理存储

##### 2.2.1 基于cookie传输sessionid（不适用于移动端【安卓、IOS、微信小程序】）

- 引入依赖

  ```
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
      </dependency>
      <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-data-redis</artifactId>
        <version>2.0.5.RELEASE</version>
      </dependency>
  ```

- 配置类

  ```
  @Component
  @EnableRedisHttpSession(maxInactiveIntervalInSeconds = 3600)
  public class RedisSessionConfig {
      
  }
  ```

##### 2.2.2 基于token传输类似sessionid(推荐)

- 修改后端代码

```
        //生成登录凭证token，UUID
        String uuidToken = UUID.randomUUID().toString();
        uuidToken = uuidToken.replace("-","");
        //建议token和用户登陆态之间的联系
        redisTemplate.opsForValue().set(uuidToken,userModel);
        redisTemplate.expire(uuidToken,1, TimeUnit.HOURS);
        //下发token
        return CommonReturnType.create(uuidToken);
```

- 修改前端代码

### 3. 查询性能优化 - 多级缓存

#### 3.1 缓存设计原则

- 用快速存取设备，用内存
- 将缓存推到离用户最近的地方
- 脏缓存清理

#### 3.2 实现多级缓存

##### 3.2.1 Redis缓存

- 单机版

```
//存入Redis
redisTemplate.opsForValue().set("item_"+id,itemModel);
redisTemplate.expire("item_"+id,10, TimeUnit.MINUTES);
```

```
//序列化
public class JodaDateTimeJsonSerializer extends JsonSerializer<DateTime> {
    @Override
    public void serialize(DateTime dateTime, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
        jsonGenerator.writeString(dateTime.toString("yyyy-MM-dd HH:mm:ss"));
    }
}
```

```
//反序列化
public class JodaDateTimeJsonDeserializer extends JsonDeserializer<DateTime> {
    @Override
    public DateTime deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException, JsonProcessingException {
        String dateString =jsonParser.readValueAs(String.class);
        DateTimeFormatter formatter = DateTimeFormat.forPattern("yyyy-MM-dd HH:mm:ss");
        return DateTime.parse(dateString,formatter);
    }
}
```

```
//自定义RedisTemplates
@Component
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 3600)
public class RedisSessionConfig {

    @Bean
    public RedisTemplate redisTemplate(RedisConnectionFactory redisConnectionFactory){
        RedisTemplate redisTemplate = new RedisTemplate();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        //解决key的序列化方式 -> String
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        redisTemplate.setKeySerializer(stringRedisSerializer);
        //解决value的序列化方式 -> Json
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper objectMapper =  new ObjectMapper();
        SimpleModule simpleModule = new SimpleModule();
        simpleModule.addSerializer(DateTime.class,new JodaDateTimeJsonSerializer());
        simpleModule.addDeserializer(DateTime.class,new JodaDateTimeJsonDeserializer());
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        objectMapper.registerModule(simpleModule);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        return redisTemplate;
    }
}
```

- Sentinal哨兵模式
- 集群cluster模式

















##### 3.2.2 本地热点缓存

**特点：**

- 热点数据
- 脏读非常不敏感
- 内存可控

**解决方案： Guava cache**

- 可控制的大小和超时时间
- 可配置的lru策略（达到存储上限后，最近最少被访问的key优先被淘汰）
- 线程安全

##### 3.2.3 nginx proxy cache 缓存

##### 3.2.4 nginx lua 缓存























