

## Spring Security OAuth2

## 1. 简介

#### 1.1 OAuth

OAuth 协议为用户资源的授权提供了一个安全的、开放而又简易的标准。与以往的授权方式不同之处是 OAuth 的授权不会使第三方触及到用户的帐号信息，即**第三方无需使用用户的用户名与密码就可以申请获得该用户资源的授权**，因此 OAuth 是安全的。

#### 1.2 Spring Security

Spring Security 是一个安全框架，能够为 Spring 企业应用系统提供声明式的安全访问控制。Spring Security 基于 Servlet 过滤器、IoC 和 AOP，为 Web 请求和方法调用提供身份确认和授权处理，避免了代码耦合，减少了大量重复代码工作。

## 2. 交互过程

oOuth 在 "客户端" 与 "服务提供商" 之间，设置了一个授权层。"客户端" 不能直接登录 "服务提供商"，只能登录授权层，以此将用户与客户端区分开来。
"客户端" 登录授权层所用的令牌，与用户的密码不同。用户可以在登录的时候，指定授权层令牌的权限范围和有效期。"客户端" 登录授权层以后，"服务提供商" 根据令牌的权限范围和有效期，向 "客户端" 开放用户储存的资料。

#### 2.1 流程图

![oAuth流程图](E:\程序人生\个人学习笔记\学习笔记图床\OAuth流程图.jpg)

#### 2.2 名词解释

- **第三方应用程序**： 又称之为客户端（PC、Android、iPhone、TV、Watch），又比如我们的产品想要使用QQ、微信等第三方登录。对我们的产品来说，QQ、微信登录是第三方登录系统，我们需要第三方登录系统的资源。对于QQ、微信等系统我们又是第三方应用程序。
- **HTTP 服务提供商**： 我们的云笔记产品以及QQ、微信等都可以称之为“服务提供商”。
- **资源所有者**： 又称之为用户。
- **用户代理**： 比如浏览器，代替用户去访问这些资源。
- **认证服务器**：服务提供商专门用来处理认证的服务器，简单点说就是登录功能（验证用户的账号密码是否正确以及分配相应的权限）
- **资源服务器**：服务提供商存放用户生成的资源的服务器。它与认证服务器，可以是同一台服务器，也可以是不同的服务器。简单点说就是资源的访问入口。

#### 2.3 交互模型

- 资源拥有者：用户
- 客户端：APP
- 服务提供方：
  - 认证服务器
  - 资源服务器

## 3. 使用

### 3.1 令牌的访问与刷新

- **Access Token**

```
客户端访问资源服务器的令牌。拥有这个令牌代表着得到用户的授权。

授权是临时的。因为 Access Token 在使用的过程中可能会泄露，限定一个较短的有效期可以降低因 Access Token 泄露而带来的风险。

引入了有效期之后，每当 Access Token 过期，客户端就必须重新向用户索要授权。

这样用户可能每隔几天，甚至每天都需要进行授权操作。这是一件非常影响用户体验的事情。

于是 oAuth2.0 引入了 Refresh Token 机制
```

- **Refresh Token**

```
作用是用来刷新 Access Token。认证服务器提供一个刷新接口，例如：

http://www.funtl.com/refresh?refresh_token=&client_id=

传入 refresh_token 和 client_id，认证服务器验证通过后，返回一个新的 Access Token。

为了安全，oAuth2.0 引入了两个措施： 
1. oAuth2.0 要求，Refresh Token 一定是保存在客户端的服务器上 ，而绝不能存放在狭义的客户端（App、PC 端软件）上。
调用 refresh 接口的时候，一定是从服务器到服务器的访问。
2. oAuth2.0 引入了 client_secret 机制。即每一个 client_id 都对应一个 client_secret。
client_secret 会在客户端申请 client_id 时，随 client_id 一起分配给客户端。
客户端必须把 client_secret 妥善保管在服务器上，决不能泄露。
刷新 Access Token 时，需要验证这个 client_secret。

实际上的刷新接口类似于：
http://www.funtl.com/refresh?refresh_token=&client_id=&client_secret=

Refresh Token 的有效期非常长，会在用户授权时，随 Access Token 一起重定向到回调 URL，传递给客户端。
```

### 3.2 客户端授权模式

```
implicit：简化模式（不推荐使用）
authorization code：授权码模式
resource owner password credentials：密码模式
client credentials：客户端模式
```

- **简化模式**

![简化模式](E:\程序人生\个人学习笔记\学习笔记图床\简化模式.jpg)

简化模式适用于纯静态页面应用。所谓纯静态页面应用，也就是应用没有在服务器上执行代码的权限（通常是把代码托管在别人的服务器上），只有前端 JS 代码的控制权。

这种场景下，应用是没有持久化存储的能力的。因此，按照 oAuth2.0 的规定，这种应用是拿不到 Refresh Token 的。

该模式下，access_token 容易泄露且不可刷新。

- **授权码模式**

![授权码模式](E:\程序人生\个人学习笔记\学习笔记图床\授权码模式.jpg)

授权码模式适用于有自己的服务器的应用，它是一个一次性的临时凭证，用来换取 access_token 和 refresh_token。认证服务器提供了一个类似这样的接口：

```
https://www.funtl.com/exchange?code=&client_id=&client_secret=
```

需要传入 code、client_id 以及 client_secret。验证通过后，返回 access_token 和 refresh_token。一旦换取成功，code 立即作废，不能再使用第二次。

code 的作用是保护 token 的安全性。简单模式下，token 是不安全的。这是因为在第 4 步当中直接把 token 返回给应用。而这一步容易被拦截、窃听。引入了 code 之后，即使攻击者能够窃取到 code，但是由于他无法获得应用保存在服务器的 client_secret，因此也无法通过 code 换取 token。

而第 5 步，为什么不容易被拦截、窃听呢？首先，这是一个从服务器到服务器的访问，黑客比较难捕捉到；其次，这个请求通常要求是 https 的实现。即使能窃听到数据包也无法解析出内容。

有了这个 code，token 的安全性大大提高。因此，oAuth2.0 鼓励使用这种方式进行授权，而简单模式则是在不得已情况下才会使用。

- **密码模式**

![密码模式](E:\程序人生\个人学习笔记\学习笔记图床\密码模式.jpg)

密码模式中，用户向客户端提供自己的用户名和密码。客户端使用这些信息，向 "服务商提供商" 索要授权。在这种模式中，用户必须把自己的密码给客户端，但是客户端不得储存密码。这通常用在用户对客户端高度信任的情况下，比如客户端是操作系统的一部分。

认证服务器需要对客户端的身份进行验证，确保是受信任的客户端。

一个典型的例子是同一个企业内部的不同产品要使用本企业的 oAuth2.0 体系。在有些情况下，产品希望能够定制化授权页面。由于是同个企业，不需要向用户展示“xxx将获取以下权限”等字样并询问用户的授权意向，而只需进行用户的身份认证即可。这个时候，由具体的产品团队开发定制化的授权界面，接收用户输入账号密码，并直接传递给鉴权服务器进行授权即可。

- **客户端模式**

![客户端模式](E:\程序人生\个人学习笔记\学习笔记图床\客户端模式.jpg)

如果信任关系再进一步，或者调用者是一个后端的模块，没有用户界面的时候，可以使用客户端模式。鉴权服务器直接对客户端进行身份验证，验证通过后，返回 token。

## 4. 令牌的存储

**操作流程：**

![图片](E:\程序人生\个人学习笔记\学习笔记图床\令牌存储.jpg)

### 4.1 基于内存存储令牌

### 4.2 基于 JDBC 存储令牌

#### 4.2.1 初始化 oAuth2 相关表

官方提供的建表脚本，地址如下：

```
https://github.com/spring-projects/spring-security-oauth/blob/master/spring-security-oauth2/src/test/resources/schema.sql
```

由于我们使用的是 MySQL 数据库，默认建表语句中主键为 VARCHAR(256)，这超过了最大的主键长度，请手动修改为 128，并用 BLOB 替换语句中的 LONGVARBINARY 类型，修改后的建表脚本如下：

```
CREATE TABLE `clientdetails` (
  `appId` varchar(128) NOT NULL,
  `resourceIds` varchar(256) DEFAULT NULL,
  `appSecret` varchar(256) DEFAULT NULL,
  `scope` varchar(256) DEFAULT NULL,
  `grantTypes` varchar(256) DEFAULT NULL,
  `redirectUrl` varchar(256) DEFAULT NULL,
  `authorities` varchar(256) DEFAULT NULL,
  `access_token_validity` int(11) DEFAULT NULL,
  `refresh_token_validity` int(11) DEFAULT NULL,
  `additionalInformation` varchar(4096) DEFAULT NULL,
  `autoApproveScopes` varchar(256) DEFAULT NULL,
  PRIMARY KEY (`appId`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `oauth_access_token` (
  `token_id` varchar(256) DEFAULT NULL,
  `token` blob,
  `authentication_id` varchar(128) NOT NULL,
  `user_name` varchar(256) DEFAULT NULL,
  `client_id` varchar(256) DEFAULT NULL,
  `authentication` blob,
  `refresh_token` varchar(256) DEFAULT NULL,
  PRIMARY KEY (`authentication_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `oauth_approvals` (
  `userId` varchar(256) DEFAULT NULL,
  `clientId` varchar(256) DEFAULT NULL,
  `scope` varchar(256) DEFAULT NULL,
  `status` varchar(10) DEFAULT NULL,
  `expiresAt` timestamp NULL DEFAULT NULL,
  `lastModifiedAt` timestamp NULL DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `oauth_client_details` (
  `client_id` varchar(128) NOT NULL,
  `resource_ids` varchar(256) DEFAULT NULL,
  `client_secret` varchar(256) DEFAULT NULL,
  `scope` varchar(256) DEFAULT NULL,
  `authorized_grant_types` varchar(256) DEFAULT NULL,
  `web_server_redirect_uri` varchar(256) DEFAULT NULL,
  `authorities` varchar(256) DEFAULT NULL,
  `access_token_validity` int(11) DEFAULT NULL,
  `refresh_token_validity` int(11) DEFAULT NULL,
  `additional_information` varchar(4096) DEFAULT NULL,
  `autoapprove` varchar(256) DEFAULT NULL,
  PRIMARY KEY (`client_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `oauth_client_token` (
  `token_id` varchar(256) DEFAULT NULL,
  `token` blob,
  `authentication_id` varchar(128) NOT NULL,
  `user_name` varchar(256) DEFAULT NULL,
  `client_id` varchar(256) DEFAULT NULL,
  PRIMARY KEY (`authentication_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `oauth_code` (
  `code` varchar(256) DEFAULT NULL,
  `authentication` blob
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `oauth_refresh_token` (
  `token_id` varchar(256) DEFAULT NULL,
  `token` blob,
  `authentication` blob
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

#### 4.2.2 数据库中配置客户端

在表`oauth_client_details`中增加一条客户端配置记录，需要设置的字段如下：

- `client_id`：客户端标识
- `client_secret`：客户端安全码（此处不能是明文，需要加密）

使用 BCryptPasswordEncoder 为客户端安全码加密，代码如下：

```
System.out.println(new BCryptPasswordEncoder().encode("secret"));
```

- `scope`：客户端授权范围
- `authorized_grant_types`：客户端授权类型
- `web_server_redirect_uri`：服务器回调地址

#### 4.2.3 添加相关依赖

数据库连接池部分弃用 Druid 改为 HikariCP （号称全球最快连接池）

```
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>${hikaricp.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
    <exclusions>
        <!-- 排除 tomcat-jdbc 以使用 HikariCP -->
        <exclusion>
            <groupId>org.apache.tomcat</groupId>
            <artifactId>tomcat-jdbc</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>${mysql.version}</version>
</dependency>
```

#### 4.2.4 配置认证服务器

```
package com.jerusalem.oauth.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerEndpointsConfigurer;
import org.springframework.security.oauth2.provider.ClientDetailsService;
import org.springframework.security.oauth2.provider.client.JdbcClientDetailsService;
import org.springframework.security.oauth2.provider.token.TokenStore;
import org.springframework.security.oauth2.provider.token.store.JdbcTokenStore;

import javax.sql.DataSource;

/****
 * @Author: jerusalem
 * @Description: 配置类
 * 认证服务器的配置
 ****/
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfiguration extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private BCryptPasswordEncoder passwordEncoder;

    /***
     * 配置客户端
//     * @param clients
//     * @throws Exception
     *  配置项
     * .inMemory()                      使用内存配置
     * .withClient("")                  客户端标识
     * .secret("")                      客户端安全码
     * .authorizedGrantTypes("")        授权类型（{授权码模式：authorization_code}）
     * .scopes("")                      授权范围
     * .redirectUris("")                注册回调地址
     */
//    @Override
//    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
//        clients
//                .inMemory()
//                .withClient("client")
//                // 需要为 secret 加密
//                .secret(passwordEncoder.encode("secret"))
//                .authorizedGrantTypes("authorization_code")
//                .scopes("app")
//                .redirectUris("https://www.funtl.com");
        @Bean
        @Primary
        @ConfigurationProperties(prefix = "spring.datasource")
        public DataSource dataSource() {
            // 配置数据源（注意，我使用的是 HikariCP 连接池），以上注解是指定数据源，否则会有冲突
            return DataSourceBuilder.create().build();
        }

        @Bean
        public TokenStore tokenStore() {
            // 基于 JDBC 实现，令牌保存到数据
            return new JdbcTokenStore(dataSource());
        }

        @Bean
        public ClientDetailsService jdbcClientDetails() {
            // 基于 JDBC 实现，需要事先在数据库配置客户端信息
            return new JdbcClientDetailsService(dataSource());
        }

        @Override
        public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
            // 设置令牌
            endpoints.tokenStore(tokenStore());
        }

        @Override
        public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
            // 读取客户端配置
            clients.withClientDetails(jdbcClientDetails());
    }
}
```

#### 4.2.5 服务器安全配置

```
package com.jerusalem.oauth.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

/****
 * @Author: jerusalem
 * @Description: 配置类
 * 服务器安全配置
 ****/
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)       //全局方法拦截 
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {

    // 设置默认的加密方式
    @Bean
    public BCryptPasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                // 在内存中创建用户并为密码加密
                .withUser("admin").password(passwordEncoder().encode("123456")).roles("ADMIN")
                .and()
                .withUser("user").password(passwordEncoder().encode("123456")).roles("USER");

    }
}
```

#### 4.2.6 application.yml

```
server:
  port: 8080

spring:
  application:
    name: spring-security-oauth2
  datasource:
    type: com.zaxxer.hikari.HikariDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    jdbc-url: jdbc:mysql://localhost:3306/springsecurity-oauth2?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=UTC
    username: root
    password: 123456
    hikari:
      minimum-idle: 5
      idle-timeout: 600000
      maximum-pool-size: 10
      auto-commit: true
      pool-name: MyHikariCP
      max-lifetime: 1800000
      connection-timeout: 30000
      connection-test-query: SELECT 1
```

#### 4.2.7 访问获取授权码

```
http://localhost:8080/oauth/authorize?client_id=client&response_type=code
```

第一次访问会跳转到登录页面

![图片](E:\程序人生\个人学习笔记\学习笔记图床\登陆图.jpg)

验证成功后会询问用户是否授权客户端

![图片](E:\程序人生\个人学习笔记\学习笔记图床\授权图.jpg)

选择授权后会跳转到**回调地址**，浏览器地址上还会包含一个**授权码**（code=XXXXX）

#### 4.2.8 通过授权码向服务器申请令牌

通过 CURL 或是 Postman 请求

```
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d 'grant_type=authorization_code&code=1JuO6V' "http://client:secret@localhost:8080/oauth/token"
```

![图片](E:\程序人生\个人学习笔记\学习笔记图床\测试图.jpg)

得到响应结果如下：

```
{
    "access_token": "016d8d4a-dd6e-4493-b590-5f072923c413",
    "token_type": "bearer",
    "expires_in": 43199,
    "scope": "app"
}
```

操作成功后数据库`oauth_access_token`表中会增加一笔记录，效果图如下：

![图片](E:\程序人生\个人学习笔记\学习笔记图床\令牌.jpg)

**附：默认的端点 URL**

- `/oauth/authorize`：授权端点
- `/oauth/token`：令牌端点
- `/oauth/confirm_access`：用户确认授权提交端点
- `/oauth/error`：授权服务错误信息端点
- `/oauth/check_token`：用于资源服务访问的令牌解析端点
- `/oauth/token_key`：提供公有密匙的端点（使用 JWT 令牌

## 5. 权限控制

```
RBAC        基于角色
ACL         访问控制列表
ABAC        基于属性
PBAC        基于策略
```

### 5.1 RBAC 基于角色的权限控制

#### 5.1.1 简介

RBAC（Role-Based Access Control），就是用户通过角色与权限进行关联。简单地说，一个用户拥有若干角色，每一个角色拥有若干权限。这样，就构造成“**用户-角色-权限**”的授权模型。在这种模型中，用户与角色之间，角色与权限之间，一般是多对多的关系。

![图片](E:\程序人生\个人学习笔记\学习笔记图床\RBAC.jpg)

#### 5.1.2 对象关系

**关系图**

![图片](E:\程序人生\个人学习笔记\学习笔记图床\关系图.jpg)

**模块图**

![图片](E:\程序人生\个人学习笔记\学习笔记图床\权限图.jpg)

#### 5.1.3 自定义认证

- **初始化 RBAC 相关表**

```
CREATE TABLE `tb_permission` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `parent_id` bigint(20) DEFAULT NULL COMMENT '父权限',
  `name` varchar(64) NOT NULL COMMENT '权限名称',
  `enname` varchar(64) NOT NULL COMMENT '权限英文名称',
  `url` varchar(255) NOT NULL COMMENT '授权路径',
  `description` varchar(200) DEFAULT NULL COMMENT '备注',
  `created` datetime NOT NULL,
  `updated` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=44 DEFAULT CHARSET=utf8 COMMENT='权限表';
insert  into `tb_permission`(`id`,`parent_id`,`name`,`enname`,`url`,`description`,`created`,`updated`) values 
(37,0,'系统管理','System','/',NULL,'2019-04-04 23:22:54','2019-04-04 23:22:56'),
(38,37,'用户管理','SystemUser','/users/',NULL,'2019-04-04 23:25:31','2019-04-04 23:25:33'),
(39,38,'查看用户','SystemUserView','',NULL,'2019-04-04 15:30:30','2019-04-04 15:30:43'),
(40,38,'新增用户','SystemUserInsert','',NULL,'2019-04-04 15:30:31','2019-04-04 15:30:44'),
(41,38,'编辑用户','SystemUserUpdate','',NULL,'2019-04-04 15:30:32','2019-04-04 15:30:45'),
(42,38,'删除用户','SystemUserDelete','',NULL,'2019-04-04 15:30:48','2019-04-04 15:30:45');

CREATE TABLE `tb_role` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `parent_id` bigint(20) DEFAULT NULL COMMENT '父角色',
  `name` varchar(64) NOT NULL COMMENT '角色名称',
  `enname` varchar(64) NOT NULL COMMENT '角色英文名称',
  `description` varchar(200) DEFAULT NULL COMMENT '备注',
  `created` datetime NOT NULL,
  `updated` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=38 DEFAULT CHARSET=utf8 COMMENT='角色表';
insert  into `tb_role`(`id`,`parent_id`,`name`,`enname`,`description`,`created`,`updated`) values 
(37,0,'超级管理员','admin',NULL,'2019-04-04 23:22:03','2019-04-04 23:22:05');

CREATE TABLE `tb_role_permission` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `role_id` bigint(20) NOT NULL COMMENT '角色 ID',
  `permission_id` bigint(20) NOT NULL COMMENT '权限 ID',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=43 DEFAULT CHARSET=utf8 COMMENT='角色权限表';
insert  into `tb_role_permission`(`id`,`role_id`,`permission_id`) values 
(37,37,37),
(38,37,38),
(39,37,39),
(40,37,40),
(41,37,41),
(42,37,42);

CREATE TABLE `tb_user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) NOT NULL COMMENT '用户名',
  `password` varchar(64) NOT NULL COMMENT '密码，加密存储',
  `phone` varchar(20) DEFAULT NULL COMMENT '注册手机号',
  `email` varchar(50) DEFAULT NULL COMMENT '注册邮箱',
  `created` datetime NOT NULL,
  `updated` datetime NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `username` (`username`) USING BTREE,
  UNIQUE KEY `phone` (`phone`) USING BTREE,
  UNIQUE KEY `email` (`email`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=38 DEFAULT CHARSET=utf8 COMMENT='用户表';
insert  into `tb_user`(`id`,`username`,`password`,`phone`,`email`,`created`,`updated`) values 
(37,'admin','$2a$10$9ZhDOBp.sRKat4l14ygu/.LscxrMUcDAfeVOEPiYwbcRkoB09gCmi','15888888888','lee.lusifer@gmail.com','2019-04-04 23:21:27','2019-04-04 23:21:29');

CREATE TABLE `tb_user_role` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `user_id` bigint(20) NOT NULL COMMENT '用户 ID',
  `role_id` bigint(20) NOT NULL COMMENT '角色 ID',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=38 DEFAULT CHARSET=utf8 COMMENT='用户角色表';
insert  into `tb_user_role`(`id`,`user_id`,`role_id`) values 
(37,37,37);
```

由于使用了 BCryptPasswordEncoder 的加密方式，故用户密码需要加密，代码如下：

```
System.out.println(new BCryptPasswordEncoder().encode("123456"));
```

- **添加相关依赖**

```
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
</dependency>
```

- **application.yml**

```
server:
  port: 8080

spring:
  application:
    name: spring-security-oauth2
  datasource:
    type: com.zaxxer.hikari.HikariDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    jdbc-url: jdbc:mysql://localhost:3306/springsecurity-oauth2?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=UTC
    username: root
    password: 123456
    hikari:
      minimum-idle: 5
      idle-timeout: 600000
      maximum-pool-size: 10
      auto-commit: true
      pool-name: MyHikariCP
      max-lifetime: 1800000
      connection-timeout: 30000
      connection-test-query: SELECT 1

mybatis:
  type-aliases-package: com.jerusalem.oauth.domain
  mapper-locations: classpath:mapper/*.xml
```