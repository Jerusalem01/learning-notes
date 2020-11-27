## Spring Cloud Alibaba

### 1. 简介

#### 1.1 概述
2018 年 10 月 31 日的凌晨，Spring Cloud Alibaba 正式入驻了 Spring Cloud 官方孵化器，并在 Maven 中央库发布了第一个版本。

Spring Cloud Alibaba 致力于提供微服务开发的一站式解决方案。此项目包含开发分布式应用微服务的必需组件，方便开发者通过 Spring Cloud 编程模型轻松使用这些组件来开发分布式应用服务。

依托 Spring Cloud Alibaba，只需要添加一些注解和少量配置，就可以将 Spring Cloud 应用接入阿里微服务解决方案，通过阿里中间件来迅速搭建分布式应用系统。

#### 1.2 主要功能

- **服务限流降级：**
默认支持 Servlet、Feign、RestTemplate、Dubbo 和 RocketMQ 限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控。
- **服务注册与发现：**
适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持。
- **分布式配置管理：** 
支持分布式系统中的外部化配置，配置更改时自动刷新。
- **消息驱动能力：**
基于 Spring Cloud Stream 为微服务应用构建消息驱动能力。
- **阿里云对象存储：**
阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- **分布式任务调度：**
提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀分配到所有 Worker（schedulerx-client）上执行。

#### 1.3 组件
- **Sentinel：**
把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。
- **Nacos：**
一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。
- **RocketMQ：**
一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。
- **Alibaba Cloud ACM：**
一款在分布式架构环境中对应用配置进行集中管理和推送的应用配置中心产品。
- **Alibaba Cloud OSS:**
阿里云对象存储服务（Object Storage Service，简称 OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。您可以在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- **Alibaba Cloud SchedulerX:** 
阿里中间件团队开发的一款分布式任务调度产品，提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。



### 2. 创建依赖管理项目
- **POM**
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.0.RELEASE</version>
    </parent>

    <groupId>com.jerusalem</groupId>
    <artifactId>hello-spring-cloud-alibaba-dependencies</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <name>hello-spring-cloud-alibaba-dependencies</name>
    <url>http://www.jerusalem.com</url>
    <inceptionYear>2019-Now</inceptionYear>

    <properties>
        <!-- 环境设置 -->
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <!-- Spring 设置 -->
        <!-- 注意版本的匹配 -->
        <spring-cloud.version>Finchley.SR2</spring-cloud.version>
        <spring-cloud-alibaba.version>0.2.1.RELEASE</spring-cloud-alibaba.version>
    </properties>
    
    <!-- 引入Spring Cloud的相关依赖 -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring-cloud-alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <!-- Compiler 插件, 设定 JDK 版本 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <showWarnings>true</showWarnings>
                </configuration>
            </plugin>

            <!-- 打包 jar 文件时，配置 manifest 文件，加入 lib 包的 jar 依赖 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <configuration>
                    <archive>
                        <addMavenDescriptor>false</addMavenDescriptor>
                    </archive>
                </configuration>
                <executions>
                    <execution>
                        <configuration>
                            <archive>
                                <manifest>
                                    <!-- Add directory entries -->
                                    <addDefaultImplementationEntries>true</addDefaultImplementationEntries>
                                    <addDefaultSpecificationEntries>true</addDefaultSpecificationEntries>
                                    <addClasspath>true</addClasspath>
                                </manifest>
                            </archive>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <!-- resource -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
            </plugin>

            <!-- install -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-install-plugin</artifactId>
            </plugin>

            <!-- clean -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-clean-plugin</artifactId>
            </plugin>

            <!-- ant -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-antrun-plugin</artifactId>
            </plugin>

            <!-- dependency -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
            </plugin>
        </plugins>

        <pluginManagement>
            <plugins>
                <!-- Java Document Generate -->
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-javadoc-plugin</artifactId>
                    <executions>
                        <execution>
                            <phase>prepare-package</phase>
                            <goals>
                                <goal>jar</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>

                <!-- YUI Compressor (CSS/JS压缩) -->
                <plugin>
                    <groupId>net.alchim31.maven</groupId>
                    <artifactId>yuicompressor-maven-plugin</artifactId>
                    <version>1.5.1</version>
                    <executions>
                        <execution>
                            <phase>prepare-package</phase>
                            <goals>
                                <goal>compress</goal>
                            </goals>
                        </execution>
                    </executions>
                    <configuration>
                        <encoding>UTF-8</encoding>
                        <jswarn>false</jswarn>
                        <nosuffix>true</nosuffix>
                        <linebreakpos>30000</linebreakpos>
                        <force>true</force>
                        <includes>
                            <include>**/*.js</include>
                            <include>**/*.css</include>
                        </includes>
                        <excludes>
                            <exclude>**/*.min.js</exclude>
                            <exclude>**/*.min.css</exclude>
                        </excludes>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>

        <!-- 资源文件配置 -->
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <excludes>
                    <exclude>**/*.java</exclude>
                </excludes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
            </resource>
        </resources>
    </build>

    <!-- 私服 -->
    <repositories>
        <!-- 阿里云 -->
        <repository>
            <id>aliyun-repos</id>
            <name>Aliyun Repository</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>

        <!-- Nexus -->
        <repository>
            <id>sonatype-repos</id>
            <name>Sonatype Repository</name>
            <url>https://oss.sonatype.org/content/groups/public</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>sonatype-repos-s</id>
            <name>Sonatype Repository</name>
            <url>https://oss.sonatype.org/content/repositories/snapshots</url>
            <releases>
                <enabled>false</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
        
        <!-- Spring -->
        <repository>
            <id>spring-snapshots</id>
            <name>Spring Snapshots</name>
            <url>https://repo.spring.io/snapshot</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>

    <pluginRepositories>
        <pluginRepository>
            <id>aliyun-repos</id>
            <name>Aliyun Repository</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>
</project>
```
**说明：**
- `parent`：继承了 Spring Boot 的 Parent，表示我们是一个 Spring Boot 工程
- `package`：pom，表示该项目仅当做依赖项目，没有具体的实现代码
- `spring-cloud-alibaba-dependencies`：properties 配置中预定义版本号为 0.2.1.RELEASE ，表示 Spring Cloud Alibaba 对应的是 Spring Cloud Finchley 版本
- `build`：配置项目所需的各种插件
- `repositories`：配置项目下载依赖时的第三方库
- **依赖版本要匹配，以防不兼容**


### 3. Nacos
#### 3.1 简介

Nacos 是构建以“服务”为中心的现代应用架构(微服务范式、云原生范式) 的服务基础设施。  
Nacos 致力于帮助您发现、配置和管理微服务。  
Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。

#### 3.2 基本架构及概念

- **服务(Service)**   
服务是指一个或一组软件功能，其目的是不同的客户端可以为不同的目的重用。Nacos 支持主流的服务生态，如 Kubernetes Service、gRPC|Dubbo RPC Service 或者 Spring Cloud RESTful Service.

- **服务注册中心(Service Registry)**  
服务实例及元数据的数据库。服务实例在启动时注册到服务注册表，并在关闭时注销。服务和路由器的客户端查询服务注册表以查找服务的可用实例。服务注册中心可能会调用服务实例的健康检查 API 来验证它是否能够处理请求。

- **服务元数据(Service Metadata)**  
服务元数据是指包括服务端点、服务标签、服务版本号、服务实例权重、路由规则、安全策略等描述服务的数据。

- **服务提供方(Service Provider)**  
提供可复用和可调用服务的应用方。

- **服务消费方(Service Consumer)**  
发起对某个服务调用的应用方。

- **配置(Configuration)**  
在系统开发过程中通常会将一些需要变更的参数、变量等从代码中分离出来独立管理，以独立的配置文件的形式存在。目的是让静态的系统工件或者交付物更好地和实际的物理运行环境进行适配。配置管理一般包含在系统部署的过程中，由系统管理员或者运维人员完成这个步骤。配置变更是调整系统运行时的行为的有效手段之一。

- **配置管理(Configuration Management)**  
在数据中心中，系统中所有配置的编辑、存储、分发、变更管理、历史版本管理、变更审计等所有与配置相关的活动统称为配置管理。

- **名字服务(Naming Service)**  
提供分布式系统中所有对象、实体的“名字”到关联的元数据之间的映射管理服务，服务发现和 DNS 就是名字服务的2大场景。

- **配置服务(Configuration Service)**  
在服务或者应用运行过程中，提供动态配置或者元数据以及配置管理的服务提供者。

#### 3.3 安装Nacos
**预备环境准备**   

`Nacos`依赖`Java`环境来运行。如果您是从代码开始构建并运行`Nacos`，还需要为此配置`Maven`环境，请确保是在以下版本环境中安装使用:
```
64 bit OS：支持 Linux/Unix/Mac/Windows，推荐选用 Linux/Unix/Mac。
64 bit JDK 1.8+ ：下载 & 配置。
Maven 3.2.x +：下载 & 配置。
```
**下载安装**  

**1.0**
```
# 下载源码
git clone https://github.com/alibaba/nacos.git

# 安装到本地仓库
cd nacos/
mvn -Prelease-nacos clean install -U

# 启动服务
cd distribution/target/nacos-server-0.7.0/nacos/bin

# Linux
./startup.sh -m standalone
# Windows
startup.cmd

# 访问服务
打开浏览器访问：http://localhost:8848/nacos
（注：从 0.8.0 版本开始，需要登录才可访问，默认账号密码为 nacos/nacos）
```
**2.0**
```
# 基于 Docker 安装 Nacos

git clone https://github.com/nacos-group/nacos-docker.git
cd nacos-docker
docker-compose -f example/standalone-mysql.yaml up

# 访问
http://192.168.137.134:8848/nacos/
默认账号密码为 nacos/nacos
```
```
更多细节内容请查阅官方文档：  
https://nacos.io/zh-cn/docs/quick-start.html
```
#### 3.4 创建服务提供者
- **POM**
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.jerusalem</groupId>
        <artifactId>hello-spring-cloud-alibaba-dependencies</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../hello-spring-cloud-alibaba-dependencies/pom.xml</relativePath>
    </parent>

    <artifactId>hello-spring-cloud-alibaba-nacos-provider</artifactId>
    <packaging>jar</packaging>

    <name>hello-spring-cloud-alibaba-nacos-provider</name>
    <url>http://www.jerusalem.com</url>
    <inceptionYear>2019-Now</inceptionYear>

    <dependencies>
        <!-- Spring Boot Begin -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- 健康检查 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- Spring Boot End -->

        <!-- Spring Cloud Begin -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!-- Spring Cloud End -->
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.jerusalem.hello.spring.cloud.alibaba.nacos.provider.NacosProviderApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```
- **Application**   

通过`@EnableDiscoveryClient`注解表明是一个 Nacos 客户端，该注解是 Spring Cloud 提供的原生注解
```
package com.jerusalem.hello.spring.cloud.alibaba.nacos.provider;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@EnableDiscoveryClient
public class NacosProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(NacosProviderApplication.class, args);
    }

    @RestController
    public class EchoController {
        @GetMapping(value = "/echo/{message}")
        public String echo(@PathVariable String message) {
            return "Hello Nacos Discovery " + message;
        }
    }
}
```
- **application.yml**
```
spring:
  application:
    name: nacos-provider   # 服务名
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848   # 服务注册中心

server:
  port: 8081   # 服务端口

management:   
  endpoints:   # 端点检查（健康检查）
    web:
      exposure:
        include: "*"
```
- **启动工程**
```
通过浏览器访问 http://localhost:8848/nacos，即 Nacos Server 网址

你会发现一个服务已经注册在服务中了，服务名为 nacos-provider

这时打开 http://localhost:8081/echo/hi ，你会在浏览器上看到：

Hello Nacos Discovery hi
```
- **服务的端点检查**
```
spring-cloud-starter-alibaba-nacos-discovery 在实现的时候提供了一个 EndPoint, EndPoint 的访问地址为 http://ip:port/actuator/nacos-discovery。 

EndPoint 的信息主要提供了两类:

1、subscribe: 显示了当前有哪些服务订阅者
2、NacosDiscoveryProperties: 显示了当前服务实例关于 Nacos 的基础配置

通过浏览器访问 http://localhost:8081/actuator/nacos-discovery
```
#### 3.5 创建服务消费者-LoadBalance

最原始的一种方式，即使用 LoadBalanceClient 和 RestTemplate 结合的方式访问。
- **POM**
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.jerusalem</groupId>
        <artifactId>hello-spring-cloud-alibaba-dependencies</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../hello-spring-cloud-alibaba-dependencies/pom.xml</relativePath>
    </parent>

    <artifactId>hello-spring-cloud-alibaba-nacos-consumer</artifactId>
    <packaging>jar</packaging>

    <name>hello-spring-cloud-alibaba-nacos-consumer</name>
    <url>http://www.jerusalem.com</url>
    <inceptionYear>2019-Now</inceptionYear>

    <dependencies>
        <!-- Spring Boot Begin -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- Spring Boot End -->

        <!-- Spring Cloud Begin -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!-- Spring Cloud End -->
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.funtl.hello.spring.cloud.alibaba.nacos.consumer.NacosConsumerApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```
- **Application**
```
package com.jerusalem.hello.spring.cloud.alibaba.nacos.consumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class NacosConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(NacosConsumerApplication.class, args);
    }
}
```
- **Configuration**   （相当于spring-context.xml）

创建一个名为`NacosConsumerConfiguration`的`Java`配置类

主要作用是为了注入`RestTemplate`
```
package com.jerusalem.hello.spring.cloud.alibaba.nacos.consumer.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class NacosConsumerConfiguration {

    
    @Bean
    @LoadBalanced   # 负载均衡 
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```
- **Controller（测试）**   

创建`NacosConsumerController`
```
package com.jerusalem.hello.spring.cloud.alibaba.nacos.consumer.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.loadbalancer.LoadBalancerClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class NacosConsumerController {

    @Autowired
    private LoadBalancerClient loadBalancerClient;

    @Autowired
    private RestTemplate restTemplate;

    @Value("${spring.application.name}")
    private String appName;

    @GetMapping(value = "/echo/app/name")
    public String echo() {
        //使用 LoadBalanceClient 和 RestTemplate 结合的方式来访问
        ServiceInstance serviceInstance = loadBalancerClient.choose("nacos-provider");
        String url = String.format("http://%s:%s/echo/%s", serviceInstance.getHost(), serviceInstance.getPort(), appName);
        return restTemplate.getForObject(url, String.class);
    }
}
```
- **application.yml**
```
spring:
  application:
    name: nacos-consumer
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848

server:
  port: 9091

management:
  endpoints:
    web:
      exposure:
        include: "*
```
- **启动工程**
```
通过浏览器访问 http://localhost:8848/nacos，即 Nacos Server 网址

你会发现多了一个名为 nacos-consumer 的服务

这时打开 http://localhost:9091/echo/app/name ，你会在浏览器上看到：

Hello Nacos Discovery nacos-consumer
```
#### 3.6 创建服务消费者-Feign
- **概述**

1. Feign 是一个声明式的伪 Http 客户端，它使得写 Http 客户端变得更简单。
2. 使用 Feign，只需要创建一个接口并注解。它具有可插拔的注解特性，可使用 Feign 注解和 JAX-RS 注解。Feign 支持可插拔的编码器和解码器。
3. Feign 默认集成了 Ribbon，Nacos 也很好的兼容了 Feign。
4. 默认实现了负载均衡的效果

- **POM**
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.jerusalem</groupId>
        <artifactId>hello-spring-cloud-alibaba-dependencies</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../hello-spring-cloud-alibaba-dependencies/pom.xml</relativePath>
    </parent>

    <artifactId>hello-spring-cloud-alibaba-nacos-consumer-feign</artifactId>
    <packaging>jar</packaging>

    <name>hello-spring-cloud-alibaba-nacos-consumer-feign</name>
    <url>http://www.jerusalem.com</url>
    <inceptionYear>2019-Now</inceptionYear>

    <dependencies>
        <!-- Spring Boot Begin -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- Spring Boot End -->

        <!-- Spring Cloud Begin -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!-- Spring Cloud End -->
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.funtl.hello.spring.cloud.alibaba.nacos.consumer.feign.NacosConsumerFeignApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```
**主要是添加了如下依赖：**
```
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>

```
- **Application**  

通过`@EnableFeignClients`注解开启 Feign 功能
```
package com.jerusalem.hello.spring.cloud.alibaba.nacos.consumer.feign;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class NacosConsumerFeignApplication {
    public static void main(String[] args) {
        SpringApplication.run(NacosConsumerFeignApplication.class, args);
    }
}
```
- **创建Feign接口**   

通过`@FeignClient("服务名")`注解来指定调用哪个服务。  

代码如下：
```
package com.jeruaslem.hello.spring.cloud.alibaba.nacos.consumer.service;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient(value = "nacos-provider")
public interface EchoService {

    @GetMapping(value = "/echo/{message}")
    String echo(@PathVariable("message") String message);
}
```
- **Controller**
```
package com.jeruaslem.hello.spring.cloud.alibaba.nacos.consumer.controller;

import com.jerusalem.hello.spring.cloud.alibaba.nacos.consumer.feign.service.EchoService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class NacosConsumerFeignController {

    @Autowired
    private EchoService echoService;

    @GetMapping(value = "/echo/hi")
    public String echo() {
        return echoService.echo("Hi Feign");
    }
}
```
- **application.yml**
```
spring:
  application:
    name: nacos-consumer-feign
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848

server:
  port: 9092

management:
  endpoints:
    web:
      exposure:
        include: "*"
```
- **启动工程**
```
同上
```
- **测试负载均衡**   

1. 启动多个`consumer-provider`实例
2. 修改`consumer-provider`项目中的`Controller`代码，用于确定负载均衡生效
```
package com.jeruaslem.hello.spring.cloud.alibaba.nacos.provider;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@EnableDiscoveryClient
public class NacosProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(NacosProviderApplication.class, args);
    }

    @Value("${server.port}")
    private String port;

    @RestController
    public class EchoController {
        @GetMapping(value = "/echo/{message}")
        public String echo(@PathVariable String message) {
            return "Hello Nacos Discovery " + message + " i am from port " + port;
        }
    }
}
```
3. 在浏览器上多次访问`http://localhost:9092/echo/hi`，浏览器交替显示:
```
Hello Nacos Discovery Hi Feign i am from port 8081
Hello Nacos Discovery Hi Feign i am from port 8082
```
**常见负载均衡策略**
1. 轮循
2. 加权轮循
3. 最少连接数
4. 源 IP 哈希

#### 3.7 分布式配置
##### 3.7.1 服务端初始化
```
1. 浏览器打开 http://localhost:8848/nacos ，访问 Nacos Server

2. 新建配置文件

spring:
  application:
    name: nacos-consumer-feign
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848

server:
  port: 8081

management:
  endpoints:
    web:
      exposure:
        include: "*"

注意：Data ID 的默认扩展名为 .properties ，希望使用 YAML 配置，此处必须指明是 .yaml
```
##### 3.7.2 客户端的使用

- **添加依赖**
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```
- **bootstrap.properties**   

创建名为`bootstrap.properties`的配置文件，并删除之前创建的`application.yml`配置文件。
```
# 这里的应用名对应 Nacos Config 中的 Data ID，实际应用名称以配置中心的配置为准
spring.application.name=nacos-provider-config

# 指定查找名为 nacos-provider-config.yaml 的配置文件
spring.cloud.nacos.config.file-extension=yaml

# Nacos Server 的地址
spring.cloud.nacos.config.server-addr=127.0.0.1:8848

#多配置加载
spring.cloud.nacos.config.ext-config[0].data-id=datasource.yml
spring.cloud.nacos.config.ext-config[0].group=dev
spring.cloud.nacos.config.ext-config[0].refresh=true

spring.cloud.nacos.config.ext-config[1].data-id=mybatis.yml
spring.cloud.nacos.config.ext-config[1].group=dev
spring.cloud.nacos.config.ext-config[1].refresh=true

spring.cloud.nacos.config.ext-config[2].data-id=other.yml
spring.cloud.nacos.config.ext-config[2].group=dev
spring.cloud.nacos.config.ext-config[2].refresh=true
顺序最好不变
```


**注意：**
```
Spring Boot 配置文件的加载顺序，依次为  
bootstrap.properties -> 
bootstrap.yml -> 
application.properties -> 
application.yml
其中 bootstrap.properties 配置为最高优先级
```
- **配置的动态更新**
```
1. 修改服务端配置，增加一个 user.name 的属性
```
```
2. 修改 Controller，增加一个请求方法，测试配置更新效果

// 注入配置文件上下文
@Autowired
private ConfigurableApplicationContext applicationContext;

// 从上下文中读取配置
@GetMapping(value = "/hi")
public String sayHi() {
    return "Hello " + applicationContext.getEnvironment().getProperty("user.name");
}
```
```
3. 通过浏览器访问该接口，浏览器显示
Hello xxx
```
```
4. 修改服务端配置
```
```
5. 刷新浏览器，浏览器显示内容发生变化
```
**注意：**
```
你可以使用spring.cloud.nacos.config.refresh.enabled=false来关闭动态刷新。
```
##### 3.7.3 多环境的配置

- **配置**  

在`Nacos Server`中增加一个名为`nacos-provider-config-prod.yaml`的配置
```
spring:
  application:
    name: nacos-consumer-feign
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848

server:
  port: 8082

management:
  endpoints:
    web:
      exposure:
        include: "*"
```
- ** **
在项目中增加一个名为`bootstrap-prod.properties`的配置文件
```
spring.profiles.active=prod
spring.application.name=nacos-provider-config
spring.cloud.nacos.config.file-extension=yaml
spring.cloud.nacos.config.server-addr=127.0.0.1:884
```
- **启动应用程序**

**1.0 IDEA启动**

此时我们有两个配置文件，分别为`bootstrap.properties`和`bootstrap-prod.properties`，我们需要指定启动时加载哪一个配置文件。

操作流程如下：
```
Run -> Edit Configurations..

设置需要激活的配置

观察日志，判断是否成功加载配置
```
**2.0 虚拟环境启动**

启动项目时只需要增加一个命令参数
```
--spring.profiles.active=环境配置
```
启动命令如下：
```
java -jar hello-spring-cloud-alibaba-nacos-provider-1.0.0-SNAPSHOT.jar --spring.profiles.active=prod
```
**附：**
```
dev    开发环境
prod   生产环境
test   测试环境
```
### 4. Sentinel
#### 4.1 简介
Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。
#### 4.2 特征
- **丰富的应用场景**： Sentinel 承接了阿里巴巴近 10 年的 双十一大促流量 的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、实时熔断下游不可用应用等。
- **完备的实时监控**： Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
- **广泛的开源生态**： Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
- **完善的 SPI 扩展点**： Sentinel 提供简单易用、完善的 SPI 扩展点。您可以通过实现扩展点，快速的定制逻辑。例如定制规则管理、适配数据源等。

#### 4.3 客户端接入
- **添加依赖**
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```
- **修改配置文件**

使用之前的 Feign 客户端，`application.yml`完整配置如下：
```
spring:
  application:
    name: nacos-consumer-feign
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    sentinel:
      transport:
        port: 8720
        dashboard: localhost:8888

server:
  port: 9092

feign:
  sentinel:
    enabled: true

management:
  endpoints:
    web:
      exposure:
        include: "*"
```
- **访问服务**
```
打开浏览器访问：http://localhost:8888/#/dashboard/home

此时会多一个名为 nacos-consumer-feign 的服务
```
#### 4.4 配置熔断器
- **创建熔断器类并实现对应的 Feign 接口**
```
package com.jerusalem.hello.spring.cloud.alibaba.nacos.consumer.feign.service.fallback;

import com.jerusalem.hello.spring.cloud.alibaba.nacos.consumer.feign.service.EchoService;
import org.springframework.stereotype.Component;

@Component
public class EchoServiceFallback implements EchoService {
    @Override
    public String echo(String message) {
        return "echo fallback";
    }
}
```
- **在 Service 中增加 fallback 指定类**
```
package com.jerusalem.hello.spring.cloud.alibaba.nacos.consumer.feign.service;

import com.jerusalem.hello.spring.cloud.alibaba.nacos.consumer.feign.service.fallback.EchoServiceFallback;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient(value = "nacos-provider", fallback = EchoServiceFallback.class)
public interface EchoService {

    @GetMapping(value = "/echo/{message}")
    String echo(@PathVariable("message") String message);
}
```
- **测试熔断器**
```
此时我们关闭服务提供者，再次请求 http://localhost:9092/echo/hi 浏览器会显示：

echo fallback
```
#### 4.5 Sentinel 控制台
- **简介**

Sentinel 控制台提供一个轻量级的控制台，提供机器发现、单机资源实时监控、集群资源汇总，以及规则管理的功能。

注意: 集群资源汇总仅支持 500 台以下的应用集群，有大概 1 - 2 秒的延时。

- **下载并打包**
```
# 下载源码
git clone https://github.com/alibaba/Sentinel.git

# 编译打包
mvn clean package
```
- **启动控制台**
```
cd sentinel-dashboard\target
java -Dserver.port=8888 -Dcsp.sentinel.dashboard.server=localhost:8888 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard.jar
```
- **访问服务**
```
打开浏览器访问：http://localhost:8888/#/dashboard/home
```

- **配置控制台信息**

`application.yml`配置文件中增加如下配置：
```
spring:
  cloud:
    sentinel:
      transport:
        port: 8719
        dashboard: localhost:8080
```
- **测试**  

使用之前的 Feign 客户端，`application.yml`完整配置如下：
```
spring:
  application:
    name: nacos-consumer-feign
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    sentinel:
      transport:
        port: 8720
        dashboard: localhost:8080

server:
  port: 9092

feign:
  sentinel:
    enabled: true

management:
  endpoints:
    web:
      exposure:
        include: "*"
```
```
打开浏览器访问：http://localhost:8080/#/dashboard/home

此时会多一个名为 nacos-consumer-feign 的服务
```
### 5. Spring Cloud Gateway
#### 5.1 概述

Spring Cloud Gateway 旨在为微服务架构提供一种简单而有效的统一的 API 路由管理方式。**Spring Cloud Gateway 作为 Spring Cloud 生态系中的网关，目标是替代 Netflix ZUUL**，其不仅提供统一的路由方式，并且基于 Filter 链的方式提供了网关基本的功能，例如：安全，监控/埋点，和限流等。（主要作用是整合各个微服务，形成一套系统）
#### 5.2 功能特征

- 基于 Spring Framework 5，Project Reactor 和 Spring Boot 2.0
- 动态路由
- Predicates 和 Filters 作用于特定路由
- 集成 Hystrix 断路器
- 集成 Spring Cloud DiscoveryClient
- 易于编写的 Predicates 和 Filters
- 限流
- 路径重写
- 负载均衡（LoadBalanceClient）

#### 5.3 工作流程

客户端向 Spring Cloud Gateway 发出请求。

在 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。

Handler 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。

#### 5.4 API 网关
- **POM**
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.jerusalem</groupId>
        <artifactId>hello-spring-cloud-alibaba-dependencies</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../hello-spring-cloud-alibaba-dependencies/pom.xml</relativePath>
    </parent>

    <artifactId>hello-spring-cloud-gateway</artifactId>
    <packaging>jar</packaging>

    <name>hello-spring-cloud-gateway</name>
    <url>http://www.jerusalem.com</url>
    <inceptionYear>2019-Now</inceptionYear>

    <dependencies>
        <!-- Spring Boot Begin -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- Spring Boot End -->

        <!-- Spring Cloud Begin -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <!-- Spring Cloud End -->

        <!-- Commons Begin -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
        </dependency>
        <!-- Commons Begin -->
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.funtl.hello.spring.cloud.gateway.GatewayApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

**==特别注意：==**

1. Spring Cloud Gateway 不使用 Web 作为服务器，而是**使用 WebFlux 作为服务器**。
2. Gateway 项目已经依赖了 starter-webflux ，所以这里**千万不要依赖 starter-web**。
3. 由于过滤器等功能依然需要 Servlet 支持，故这里还需要依赖 javax.servlet:javax.servlet-api 。

- **Application**
```
package com.jerusalem.hello.spring.cloud.gateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```
- **application.yml**
```-yml
spring:
  application:
    # 应用名称
    name: spring-gateway
  cloud:
    # 使用 Naoos 作为服务注册发现
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    # 使用 Sentinel 作为熔断器
    sentinel:
      transport:
        port: 8721
        dashboard: localhost:8080
    # 路由网关配置
    gateway:
      # 设置与服务注册发现组件结合，这样可以采用服务名的路由策略
      discovery:
        locator:
          enabled: true
      globalcors:
        cors-configurations:
          # 匹配所有请求
          '[/**]':
            #跨域处理 允许所有的域
            allowedOrigins: "*"
            # 支持的方法
            allowedMethods:
              - GET
              - POST
              - PUT
              - DELETE
      routes:
        # 唯一标识符
        - id: goods-service-consumer
          # 指定要路由到的服务，以下各种请求都将被路由到 goods-service（http://localhost:8081）微服务
#          uri: http://localhost:8081
          # 采用 LoadBalanceClient 方式请求，以 lb:// 开头，后面的是注册在 Nacos 上的服务名[只要应用于集群环境]
          uri: lb://goods-service
          # 路由断言
          predicates:
          # 用户请求的域名规则配置
#          - Host=ace.com**
          # 路径过滤
          - Path=/**
          filters:
          # 将请求路径中的前 N个路径去掉
#          - StripPrefix= N
          # 添加前缀
          - PrefixPath=/brand
          - name: RequestRateLimiter
            args:
              # 用户身份识别唯一标识符
              key-resolver: "#{@userIpKeyResolver}"
              # 令牌桶每秒填充平均速率（每秒钟允许一个请求）
              redis-rate-limiter.replenishRate: 1
              # 令牌桶总容量（允许并发有4个请求，即宽限的个数）
              redis-rate-limiter.burstCapacity: 4
        - id: NACOS-CONSUMER-FEIGN
          uri: lb://nacos-consumer-feign
          predicates:
            - Method=GET,POST

server:
  port: 9000
  
# 目前无效
feign:
  sentinel:
    enabled: true

# 目前无效
management:
  endpoints:
    web:
      exposure:
        include: "*"

# 配置日志级别，方别调试
logging:
  level:
    org.springframework.cloud.gateway: debug
```
- **测试访问**   

依次运行`Nacos`服务、 `NacosProviderApplication`、`NacosConsumerApplication`、`NacosConsumerFeignApplication`、`GatewayApplication`
```
打开浏览器访问：
http://localhost:9000/nacos-consumer/echo/app/name
浏览器显示
Hello Nacos Discovery nacos-consumer i am from port 8082
```
```
打开浏览器访问：
http://localhost:9000/nacos-consumer-feign/echo/hi
浏览器显示
Hello Nacos Discovery Hi Feign i am from port 8082
```
注意：请求方式是
```
http://路由网关IP:路由网关Port/服务名/**
```
#### 5.5 全局过滤器

- **创建全局过滤器**

实现`GlobalFilter`，`Ordered`接口

增加`@Component`注解
```
package com.jerusalem.hello.spring.cloud.gateway.filters;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.google.common.collect.Maps;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.core.io.buffer.DataBuffer;
import org.springframework.http.HttpStatus;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

import java.util.Map;

/**
 * 鉴权过滤器
 */
@Component
public class AuthFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = exchange.getRequest().getQueryParams().getFirst("token");

        if (token == null || token.isEmpty()) {
            ServerHttpResponse response = exchange.getResponse();

            // 封装错误信息
            Map<String, Object> responseData = Maps.newHashMap();
            responseData.put("code", 401);
            responseData.put("message", "非法请求");
            responseData.put("cause", "Token is empty");

            try {
                // 将信息转换为 JSON
                ObjectMapper objectMapper = new ObjectMapper();
                byte[] data = objectMapper.writeValueAsBytes(responseData);

                // 输出错误信息到页面
                DataBuffer buffer = response.bufferFactory().wrap(data);
                response.setStatusCode(HttpStatus.UNAUTHORIZED);
                response.getHeaders().add("Content-Type", "application/json;charset=UTF-8");
                return response.writeWith(Mono.just(buffer));
            } catch (JsonProcessingException e) {
                e.printStackTrace();
            }
        }

        return chain.filter(exchange);
    }

    /**
    * 设置过滤器的执行顺序
    * @return 
    */
    @Override
    public int getOrder() {
        return Ordered.LOWEST_PRECEDENCE;
    }
}
```
- **测试过滤器**
```
浏览器访问：
http://localhost:9000/nacos-consumer/echo/app/name
网页显示
···

浏览器访问：
http://localhost:9000/nacos-consumer/echo/app/name?token=123456 
网页显示
Hello Nacos Discovery nacos-consumer i am from port 8082
```
### 6. SkyWalking
#### 为什么需要链路追踪？
微服务架构是通过业务来划分服务的，使用 REST 调用。对外暴露的一个接口，可能需要很多个服务协同才能完成这个接口功能，如果链路上任何一个服务出现问题或者网络超时，都会形成导致接口调用失败。随着业务的不断扩张，服务之间互相调用会越来越复杂。

面对以上情况，我们就需要一些可以**帮助理解系统行为**、**用于分析性能问题**的工具，以便发生故障的时候，能够**快速定位和解决问题**，这就是所谓的**APM（应用性能管理）**。

#### 6.1 简介
Apache SkyWalking 是**观察性分析平台和应用性能管理系统**。提供**分布式追踪**、**服务网格遥测分析**、**度量聚合**和**可视化一体化解决方案**。
#### 6.2 组件
- **Skywalking Agent**： 使用 JavaAgent 做字节码植入，无侵入式的收集，并通过 HTTP 或者 gRPC 方式发送数据到 SkyWalking Collector。
- **SkyWalking Collector**： 链路数据收集器，对 agent 传过来的数据进行整合分析处理并落入相关的数据存储中。
- **Storage**： SkyWalking 的存储，时间更迭，SW 已经开发迭代到了 6.x 版本，在 6.x 版本中支持以 ElasticSearch(支持 6.x)、Mysql、TiDB、H2、作为存储介质进行数据存储。
- **UI**： Web 可视化平台，用来展示落地的数据。

#### 6.3 功能特性
- 多种监控手段，语言探针和服务网格(Service Mesh)
- 多语言自动探针，Java，.NET Core 和 Node.JS
- 轻量高效，不需要大数据
- 模块化，UI、存储、集群管理多种机制可选
- 支持告警
- 优秀的可视化方案

#### 6.4 服务端配置
##### 6.4.1 基于 Docker 安装 ElasticSearch
- **docker-compose.yml**
```
version: '3.3'
services:
  elasticsearch:
    image: wutang/elasticsearch-shanghai-zone:6.3.2
    container_name: elasticsearch
    restart: always
    ports:
      - 9200:9200
      - 9300:9300
    environment:
      cluster.name: elasticsearch
```
说明：

`9200`为 SkyWalking 配置 ElasticSearch 所需端口号  
`cluster.name`为 SkyWalking 配置 ElasticSearch 集群的名称

- **测试是否启动成功**
```
浏览器访问 http://elasticsearchIP:9200/ 
```
##### 6.4.2 下载并启动 SkyWalking
- **下载 SkyWalking**
```
下载地址为 http://skywalking.apache.org/downloads/
下载 6.x releases 版本
```

- **配置 SkyWalking**

1. 解压缩
2. 进入`apache-skywalking-apm-incubating/config`目录
3. 修改`application.yml`配置文件
```
注释掉默认的 H2 存储方案
启用 ElasticSearch 存储方案
修改 ElasticSearch 服务器地址
```

- **启动 SkyWalking**

进入`apache-skywalking-apm-incubating\bin`目录

运行`startup.bat`启动服务端

通过浏览器访问 
```
http://localhost:8080
```
默认的用户名密码为：admin/admin
#### 6.5 客户端配置
##### 6.5.1 Java Agent 服务器探针
按照实际情况我们需要实现三种部署方式

- IDEA 部署探针
- Java 启动方式部署探针（Spring Boot 应用程序，需要使用 java -jar 的方式启动应用）
- Docker 启动方式部署探针（做到一次构建到处运行的持续集成效果）

**探针文件在`apache-skywalking-apm-incubating/agent`目录下**

- **IDEA 部署探针**

1. 创建一个名为`hello-spring-cloud-external-skywalking`的目录，并将`agent`整个目录拷贝进来
2. 修改项目的 VM 运行参数

点击菜单栏中的 Run -> EditConfigurations...，此处我们以 nacos-provider 项目为例，修改参数如下
```
-javaagent:D:\Workspace\Others\hello-spring-cloud-alibaba\hello-spring-cloud-external-skywalking\agent\skywalking-agent.jar
-Dskywalking.agent.service_name=nacos-provider
-Dskywalking.collector.backend_service=localhost:11800
```
说明：  
-javaagent：用于指定探针路径   
-Dskywalking.agent.service_name：用于重写    agent/config/agent.config 配置文件中的服务名   
-Dskywalking.collector.backend_service：用于重写 agent/config/agent.config 配置文件中的服务地址   

- **Java 启动方式**
```
java -javaagent:/path/to/skywalking-agent/skywalking-agent.jar -Dskywalking.agent.service_name=nacos-provider -Dskywalking.collector.backend_service=localhost:11800 -jar yourApp.jar
```
##### 6.5.2 SkyWalking Trace 监控
SkyWalking 通过业务调用监控进行依赖分析，提供服务之间的服务调用拓扑关系、以及针对每个 Endpoint 的 Trace 记录。

- **调用链路监控**  

点击 Trace 菜单，进入追踪页

点击 Trace ID 展开详细信息

Span（基本工作单元，表示一次完整的请求，包含响应，即请求并响应）

Span `/echo/{message}` 说明如下：  
Duration：响应时间 185 毫秒  
component：组件类型为 SpringMVC  
url：请求地址  
http.method：请求类型  
- **服务性能指标监控**

点击 Service 菜单，进入服务性能指标监控页

选择希望监控的服务

- Avg SLA： 服务可用性（主要是通过请求成功与失败次数来计算）
- CPM： 每分钟调用次数
- Avg Response Time： 平均响应时间

点击 More Server Details... 查看详细信息

- 服务可用性指标 SLA
- 每分钟平均响应数
- 平均响应时间
- 服务进程 PID
- 服务所在物理机的 IP、Host、OS
- 运行时 CPU 使用率
- 运行时堆内存使用率
- 运行时非堆内存使用率
- GC 情况

#### 6.6 Maven Assembly 插件
##### 6.6.1 简介
Assembly 插件目的是提供一个把工程依赖元素、模块、网站文档等其他文件存放到单个归档文件里。

##### 6.6.2 支持的归档文件类型
- zip
- tar.gz
- tar.bz2
- jar
- dir
- war

##### 6.6.3 使用
- **POM**  

在`pom.xml`中增加插件配置
```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <executions>
                <!-- 配置执行器 -->
                <execution>
                    <id>make-assembly</id>
                    <!-- 绑定到 package 生命周期阶段上 -->
                    <phase>package</phase>
                    <goals>
                        <!-- 只运行一次 -->
                        <goal>single</goal>
                    </goals>
                    <configuration>
                        <finalName>skywalking</finalName>
                        <descriptors>
                            <!-- 配置描述文件路径 -->
                            <descriptor>src/main/resources/assembly.xml</descriptor>
                        </descriptors>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```
- **assembly.xml**

创建`src/main/resources/assembly.xml`配置文件
```
<assembly>
    <id>6.0.0-Beta</id>
    <formats>
        <!-- 打包的文件格式，支持 zip、tar.gz、tar.bz2、jar、dir、war -->
        <format>tar.gz</format>
    </formats>
    <!-- tar.gz 压缩包下是否生成和项目名相同的根目录，有需要请设置成 true -->
    <includeBaseDirectory>false</includeBaseDirectory>
    <dependencySets>
        <dependencySet>
            <!-- 是否把本项目添加到依赖文件夹下，有需要请设置成 true -->
            <useProjectArtifact>false</useProjectArtifact>
            <outputDirectory>lib</outputDirectory>
            <!-- 将 scope 为 runtime 的依赖包打包 -->
            <scope>runtime</scope>
        </dependencySet>
    </dependencySets>
    <fileSets>
        <fileSet>
            <!-- 设置需要打包的文件路径 -->
            <directory>agent</directory>
            <!-- 打包后的输出路径 -->
            <outputDirectory></outputDirectory>
        </fileSet>
    </fileSets>
</assembly>
```
- **打包**
```
mvn clean package
mvn clean install
```

**package**：会在 target 目录下创建名为`skywalking-6.0.0-Beta.tar.gz`的压缩包   
**install**：会在本地仓库目录下创建名为 `hello-spring-cloud-external-skywalking-1.0.0-SNAPSHOT-6.0.0-Beta.tar.gz`的压缩包

### 7. RocketMQ-异步通信

**消息队列MQ**

消息队列作为高并发系统的核心组件之一，能够帮助业务系统解构提升开发效率和系统稳定性。

主要具有以下优势：

**削峰填谷**： 主要解决瞬时写压力大于应用服务能力导致消息丢失、系统奔溃等问题  
**系统解耦**： 解决不同重要程度、不同能力级别系统之间依赖导致一死全死  
**提升性能**： 当存在一对多调用时，可以发一条消息给消息系统，让消息系统通知相关系统  
**蓄流压测**： 线上有些链路不好压测，可以通过堆积一定量消息再放开来压测  
**数据分发**

目前主流的 MQ 主要是 **RocketMQ**、kafka、RabbitMQ
```
讲解视频
https://www.bilibili.com/video/av29725868
相关文档
https://www.funtl.com/zh/spring-cloud-alibaba/%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97%E7%9A%84%E6%B5%81%E6%B4%BE.html#%E6%9C%AC%E8%8A%82%E8%A7%86%E9%A2%91
```

#### 7.1 简介
##### 7.1.1 概述

**Apache Alibaba RocketMQ** 是一款分布式、队列模型的消息中间件。

消息中间件中有两个角色：消息生产者和消息消费者。

RocketMQ 里同样有这两个概念，消息生产者负责创建消息并发送到 RocketMQ 服务器，RocketMQ 服务器会将消息持久化到磁盘，消息消费者从 RocketMQ 服务器拉取消息并提交给应用消费。

##### 7.1.2 特点

- 支持严格的消息顺序
- 支持 Topic 与 Queue 两种模式
- 亿级消息堆积能力
- 比较友好的分布式特性
- 同时支持 Push 与 Pull 方式消费消息
- 历经多次天猫双十一海量消息考验

##### 7.1.3 优势

- 支持事务型消息（消息发送和 DB 操作保持两方的最终一致性，RabbitMQ 和 Kafka 不支持）
- 支持结合 RocketMQ 的多个系统之间数据最终一致性（多方事务，二方事务是前提）
- 支持 18 个级别的延迟消息（RabbitMQ 和 Kafka 不支持）
- 支持指定次数和时间间隔的失败消息重发（Kafka 不支持，RabbitMQ 需要手动确认）
- 支持 Consumer 端 Tag 过滤，减少不必要的网络传输（RabbitMQ 和 Kafka 不支持）
- 支持重复消费（RabbitMQ 不支持，Kafka 支持）

##### 7.1.4 消息队列对比参照表
```
https://www.funtl.com/assets1/12619159-ebd12b24d5ae33d9.png
```

#### 7.2 基于 Docker 安装 RocketMQ

- **docker-compose.yml**

**注意：启动 RocketMQ Server + Broker + Console 至少需要 2G 内存**
```
version: '3.5'
services:
  rmqnamesrv:
    image: foxiswho/rocketmq:server
    container_name: rmqnamesrv
    ports:
      - 9876:9876
    volumes:
      - ./data/logs:/opt/logs
      - ./data/store:/opt/store
    networks:
        rmq:
          aliases:
            - rmqnamesrv

  rmqbroker:
    image: foxiswho/rocketmq:broker
    container_name: rmqbroker
    ports:
      - 10909:10909
      - 10911:10911
    volumes:
      - ./data/logs:/opt/logs
      - ./data/store:/opt/store
      - ./data/brokerconf/broker.conf:/etc/rocketmq/broker.conf
    environment:
        NAMESRV_ADDR: "rmqnamesrv:9876"
        JAVA_OPTS: " -Duser.home=/opt"
        JAVA_OPT_EXT: "-server -Xms128m -Xmx128m -Xmn128m"
    command: mqbroker -c /etc/rocketmq/broker.conf
    depends_on:
      - rmqnamesrv
    networks:
      rmq:
        aliases:
          - rmqbroker

  rmqconsole:
    image: styletang/rocketmq-console-ng
    container_name: rmqconsole
    ports:
      - 8080:8080
    environment:
        JAVA_OPTS: "-Drocketmq.namesrv.addr=rmqnamesrv:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false"
    depends_on:
      - rmqnamesrv
    networks:
      rmq:
        aliases:
          - rmqconsole

networks:
  rmq:
    name: rmq
    driver: bridge
```
- **broker.conf**

在` ./data/brokerconf/`目录下创建一个名为`broker.conf`的配置文件，内容如下：
```
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
#  limitations under the License.


# 所属集群名字
brokerClusterName=DefaultCluster

# broker 名字，注意此处不同的配置文件填写的不一样，如果在 broker-a.properties 使用: broker-a,
# 在 broker-b.properties 使用: broker-b
brokerName=broker-a

# 0 表示 Master，> 0 表示 Slave
brokerId=0

# nameServer地址，分号分割
# namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876

# 启动IP,如果 docker 报 com.alibaba.rocketmq.remoting.exception.RemotingConnectException: connect to <192.168.0.120:10909> failed
# 解决方式1 加上一句 producer.setVipChannelEnabled(false);，解决方式2 brokerIP1 设置宿主机IP，不要使用docker 内部IP
# brokerIP1=192.168.0.253

# 在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4

# 是否允许 Broker 自动创建 Topic，建议线下开启，线上关闭 ！！！这里仔细看是 false，false，false
autoCreateTopicEnable=true

# 是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true

# Broker 对外服务的监听端口
listenPort=10911

# 删除文件时间点，默认凌晨4点
deleteWhen=04

# 文件保留时间，默认48小时
fileReservedTime=120

# commitLog 每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824

# ConsumeQueue 每个文件默认存 30W 条，根据业务情况调整
mapedFileSizeConsumeQueue=300000

# destroyMapedFileIntervalForcibly=120000
# redeleteHangedFileInterval=120000
# 检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
# 存储路径
# storePathRootDir=/home/ztztdata/rocketmq-all-4.1.0-incubating/store
# commitLog 存储路径
# storePathCommitLog=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/commitlog
# 消费队列存储
# storePathConsumeQueue=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/consumequeue
# 消息索引存储路径
# storePathIndex=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/index
# checkpoint 文件存储路径
# storeCheckpoint=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/checkpoint
# abort 文件存储路径
# abortFile=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/abort
# 限制的消息大小
maxMessageSize=65536

# flushCommitLogLeastPages=4
# flushConsumeQueueLeastPages=2
# flushCommitLogThoroughInterval=10000
# flushConsumeQueueThoroughInterval=60000

# Broker 的角色
# - ASYNC_MASTER 异步复制Master
# - SYNC_MASTER 同步双写Master
# - SLAVE
brokerRole=ASYNC_MASTER

# 刷盘方式
# - ASYNC_FLUSH 异步刷盘
# - SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH

# 发消息线程池数量
# sendMessageThreadPoolNums=128
# 拉消息线程池数量
# pullMessageThreadPoolNums=128
```
- **RocketMQ 控制台**
```
访问 http://rmqIP:8080 登入控制台
```

#### 7.3 RocketMQ 生产者

- **概述**

RocketMQ 是一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。

- **Spring Cloud Stream**

**Spring Cloud Stream 是一个用于构建基于消息的微服务应用框架**。它基于 Spring Boot 来创建具有生产级别的单机 Spring 应用，并且使用 Spring Integration 与 Broker 进行连接。

Spring Cloud Stream 提供了消息中间件配置的统一抽象，推出了 publish-subscribe、consumer groups、partition 这些统一的概念。

Spring Cloud Stream 内部有两个概念：

**Binder**： 跟外部消息中间件集成的组件，用来创建 Binding，各消息中间件都有自己的 Binder 实现。  
**Binding**： 包括 Input Binding 和 Output Binding。
Binding 在消息中间件与应用程序提供的 Provider 和 Consumer 之间提供了一个桥梁，实现了开发者只需使用应用程序的 Provider 或 Consumer 生产或消费数据即可，屏蔽了开发者与底层消息中间件的接触。

- **解决连接超时问题**

我们采用 Docker 部署 RocketMQ 服务，此时 RocketMQ Broker 暴露的地址和端口(10909，10911)是基于容器的，会导致我们开发机无法连接，从而引发`org.apache.rocketmq.remoting.exception.RemotingTooMuchRequestException: sendDefaultImpl call timeout`异常

开发机与容器不在一个局域网所以无法连接。

解决方案是在`broker.conf`配置文件中增加 
```
brokerIP1=宿主机IP 
```
即可

- **POM**

增加 `org.springframework.cloud:spring-cloud-starter-stream-rocketmq`依赖
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.jerusalem</groupId>
        <artifactId>hello-spring-cloud-alibaba-dependencies</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../hello-spring-cloud-alibaba-dependencies/pom.xml</relativePath>
    </parent>

    <artifactId>hello-spring-cloud-alibaba-rocketmq-provider</artifactId>
    <packaging>jar</packaging>

    <name>hello-spring-cloud-alibaba-rocketmq-provider</name>
    <url>http://www.jerusalem.com</url>
    <inceptionYear>2019-Now</inceptionYear>

    <dependencies>
        <!-- Spring Boot Begin -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- Spring Boot End -->

        <!-- Spring Cloud Begin -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rocketmq</artifactId>
        </dependency>
        <!-- Spring Cloud End -->
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.funtl.hello.spring.cloud.alibaba.rocketmq.provider.RocketMQProviderApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```
- **消息生产者服务**
```
package com.jerusalem.hello.spring.cloud.alibaba.rocketmq.provider.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.stereotype.Service;

@Service
public class ProviderService {
    @Autowired
    private MessageChannel output;

    public void send(String message) {
        output.send(MessageBuilder.withPayload(message).build());
    }
}
```
- **Application**

配置 Output`(Source.class)` 的 Binding 信息并配合 `@EnableBinding`注解使其生效
```
package com.jerusalem.hello.spring.cloud.alibaba.rocketmq.provider;

import com.jerusalem.hello.spring.cloud.alibaba.rocketmq.provider.service.ProviderService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.messaging.Source;

@SpringBootApplication
@EnableBinding({Source.class})
public class RocketMQProviderApplication implements CommandLineRunner {

    @Autowired
    private ProviderService providerService;

    public static void main(String[] args) {
        SpringApplication.run(RocketMQProviderApplication.class, args);
    }

    /**
     * 实现了 CommandLineRunner 接口，只是为了 Spring Boot 启动时执行任务，不必特别在意
     * @param args
     * @throws Exception
     */
    @Override
    public void run(String... args) throws Exception {
        providerService.send("Hello RocketMQ");
    }
}
```
- **application.yml**
```
spring:
  application:
    name: rocketmq-provider
  cloud:
    stream:
      rocketmq:
        binder:
          # RocketMQ 服务器地址
          namesrv-addr: 192.168.10.149:9876
      bindings:
        # 这里是个 Map 类型参数，{} 为 YAML 中 Map 的行内写法
        output: {destination: test-topic, content-type: application/json}

server:
  port: 9093

management:
  endpoints:
    web:
      exposure:
        include: '*'
```
运行成功后即可在 RocketMQ 控制台的消息列表中选择`test-topic`主题即可看到发送的消息。

#### 7.4 RocketMQ 消费者

- **POM**

增加`org.springframework.cloud:spring-cloud-starter-stream-rocketmq`依赖
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.jerusalem</groupId>
        <artifactId>hello-spring-cloud-alibaba-dependencies</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../hello-spring-cloud-alibaba-dependencies/pom.xml</relativePath>
    </parent>

    <artifactId>hello-spring-cloud-alibaba-rocketmq-consumer</artifactId>
    <packaging>jar</packaging>

    <name>hello-spring-cloud-alibaba-rocketmq-consumer</name>
    <url>http://www.jerisalem.com</url>
    <inceptionYear>2019-Now</inceptionYear>

    <dependencies>
        <!-- Spring Boot Begin -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- Spring Boot End -->

        <!-- Spring Cloud Begin -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rocketmq</artifactId>
        </dependency>
        <!-- Spring Cloud End -->
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.funtl.hello.spring.cloud.alibaba.rocketmq.consumer.RocketMQConsumerApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```
- **消息消费者服务**

主要使用`@StreamListener("input")`注解来订阅从名为`input`的 Binding 中接收的消息
```
package com.jerusalem.hello.spring.cloud.alibaba.rocketmq.consumer.receive;

import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.stereotype.Service;

@Service
public class ConsumerReceive {

    @StreamListener("input")
    public void receiveInput(String message) {
        System.out.println("Receive input: " + message);
    }
}
```
- **Application**

配置 Input(Sink.class) 的 Binding 信息并配合`@EnableBinding`注解使其生效
```
package com.jerusalem.hello.spring.cloud.alibaba.rocketmq.consumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.messaging.Sink;

@SpringBootApplication
@EnableBinding({Sink.class})
public class RocketMQConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(RocketMQConsumerApplication.class, args);
    }
}
```
- **application.yml**
```
spring:
  application:
    name: rocketmq-consumer
  cloud:
    stream:
      rocketmq:
        binder:
          namesrv-addr: 192.168.10.149:9876
        bindings:
          input: {consumer.orderly: true}
      bindings:
        input: {destination: test-topic, content-type: text/plain, group: test-group, consumer.maxAttempts: 1}

server:
  port: 9094

management:
  endpoints:
    web:
      exposure:
        include: '*'
```
运行成功后即可在控制台接收到消息：`Receive input: Hello RocketMQ`

#### 7.5 RocketMQ 自定义 Binding
- **概述**

实际生产中，我们需要发布和订阅的消息可能不止一种 Topic ，故此时就需要使用自定义 Binding 来帮我们实现多 Topic 的发布和订阅功能。

- **生产者**
自定义 Output 接口，代码如下：
```
public interface MySource {
    @Output("output1")
    MessageChannel output1();

    @Output("output2")
    MessageChannel output2();
}
```
发布消息的案例代码如下：
```
@Autowired
private MySource source;

public void send(String msg) throws Exception {
    source.output1().send(MessageBuilder.withPayload(msg).build());
}
```
- **消费者**

自定义 Input 接口，代码如下：
```
public interface MySink {
    @Input("input1")
    SubscribableChannel input1();

    @Input("input2")
    SubscribableChannel input2();

    @Input("input3")
    SubscribableChannel input3();

    @Input("input4")
    SubscribableChannel input4();
}
```
接收消息的案例代码如下：
```
@StreamListener("input1")
public void receiveInput1(String receiveMsg) {
    System.out.println("input1 receive: " + receiveMsg);
}
```
- **Application**

配置 Input 和 Output 的 Binding 信息并配合`@EnableBinding`注解使其生效，代码如下：
```
@SpringBootApplication
@EnableBinding({ MySource.class, MySink.class })
public class RocketMQApplication {
	public static void main(String[] args) {
		SpringApplication.run(RocketMQApplication.class, args);
	}
}
```
- **application.yml**
```
# 生产者

spring:
  application:
    name: rocketmq-provider
  cloud:
    stream:
      rocketmq:
        binder:
          namesrv-addr: 192.168.10.149:9876
      bindings:
        output1: {destination: test-topic1, content-type: application/json}
        output2: {destination: test-topic2, content-type: application/json}
```
```
# 消费者

spring:
  application:
    name: rocketmq-consumer
  cloud:
    stream:
      rocketmq:
        binder:
          namesrv-addr: 192.168.10.149:9876
        bindings:
          input: {consumer.orderly: true}
      bindings:
        input1: {destination: test-topic1, content-type: text/plain, group: test-group, consumer.maxAttempts: 1}
        input2: {destination: test-topic1, content-type: text/plain, group: test-group, consumer.maxAttempts: 1}
        input3: {destination: test-topic2, content-type: text/plain, group: test-group, consumer.maxAttempts: 1}
         input4: {destination: test-topic2, content-type: text/plain, group: test-group, consumer.maxAttempts: 1}
```
`