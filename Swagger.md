## Swagger

### 1. 简介

Swagger 是一个规范且完整的框架，用于生成、描述、调用和可视化 RESTful 风格的 Web 服务。

**Swagger 的优势**

- **支持 API 自动生成同步的在线文档**：使用 Swagger 后可以直接通过代码生成文档，不再需要自己手动编写接口文档了。
- **提供 Web 页面在线测试 API**：Swagger 生成的文档还支持在线测试。参数和格式都定好了，直接在界面上输入参数对应的值即可在线测试接口。

**Swagger 的主要项目**

- **Swagger-tools**: 提供各种与Swagger进行集成和交互的工具。例如模式检验、Swagger 1.2文档转换成Swagger 2.0文档等功能。
- **Swagger-core**: 用于Java/Scala的的Swagger实现。与JAX-RS(Jersey、Resteasy、CXF...)、Servlets和Play框架进行集成。
- **Swagger-js**: 用于JavaScript的Swagger实现。
- **Swagger-node-express**: Swagger模块，用于node.js的Express web应用框架。
- **Swagger-ui**：一个无依赖的HTML、JS和CSS集合，可以为Swagger兼容API动态生成优雅文档。
- **Swagger-codegen**：一个模板驱动引擎，通过分析用户Swagger资源声明以各种语言生成客户端代码。

### 2. 集成使用

#### 2.1 引入相关依赖

```
        <!--swagger-->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>
```

#### 2.2 配置类

```
package com.jerusalem.swagger.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;
import org.springframework.core.env.Profiles;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

/****
 * @Author: jerusalem
 * @Description: Swagger2配置类
 * @Date 2020/3/19 8:53
 *****/
@Configuration
@EnableSwagger2         //启用Swagger2
public class SwaggerConfig {

    /**
     * 创建API应用
     * 分组A
     * .apiInfo()：配置API接口文档的基本信息
     * .select()：返回一个ApiSelectorBuilder实例，用来控制哪些接口暴露给Swagger来实现
     * .apis(RequestHandlerSelectors.):配置扫描接口的方式
     *                              .basePackage("XXX")：指定需要扫描的包（常用）
     *                              .any()：扫描全部
     *                              .none()：都不扫描
     *                              .withMethodAnnotation：扫描方法上的注解
     *                              .withClassAnnotation：扫描类上的注解
     * .paths(PathSelectors.any())：过滤
     * .enable()：是否启用Swagger
     * .groupName("XXX")：分组名（可以配置多个分组）
     * @return
     */
    @Bean
    public Docket createRestApiA(Environment environment) {

        //设置要显示的Swagger环境
        Profiles profiles = Profiles.of("dev","test");
        //获取项目的环境（检测到dev，test环境为true，否则为false）
        boolean flag = environment.acceptsProfiles(profiles);
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .groupName("A")
                .enable(flag)
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.jerusalem.swagger.controller"))
//                .paths(PathSelectors.ant("/jerusalem/**"))
                .build();
    }

    /***
     * 分组B
     * @param environment
     * @return
     */
    @Bean
    public Docket createRestApiB(Environment environment) {

        //设置要显示的Swagger环境
        Profiles profiles = Profiles.of("dev","test");
        //获取项目的环境（检测到dev，test环境为true，否则为false）
        boolean flag = environment.acceptsProfiles(profiles);
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .groupName("B")
                .enable(flag)
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.jerusalem.swagger.controller"))
//                .paths(PathSelectors.ant("/jerusalem/**"))
                .build();
    }

    /***
     * 分组C
     * @param environment
     * @return
     */
    @Bean
    public Docket createRestApiC(Environment environment) {

        //设置要显示的Swagger环境
        Profiles profiles = Profiles.of("dev","test");
        //获取项目的环境（检测到dev，test环境为true，否则为false）
        boolean flag = environment.acceptsProfiles(profiles);
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .groupName("C")
                .enable(flag)
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.jerusalem.swagger.controller"))
//                .paths(PathSelectors.ant("/jerusalem/**"))
                .build();
    }


    /**
     * 配置API的基本信息
     * 访问地址：http://项目实际地址/swagger-ui.html
     * @return
     */
    private ApiInfo apiInfo() {
        // 作者信息
        Contact contact = new Contact("jerusalem","http://www.baidu.com","3276586184@qq.com");
        return new ApiInfoBuilder()
                .title("爱思商城API接口文档")
                .description("细节决定成败")
                .termsOfServiceUrl("http://www.baidu.com")
                .contact(contact)
                .version("v 1.0")
                .build();
    }
}
```

#### 2.3 注解的使用

- **实体类**

```
package com.jerusalem.swagger.pojo;

import io.swagger.annotations.Api;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

/****
 * @Author: jerusalem
 * @Description: User实体类
 * @Date 2020/3/19 11:11
 * 可以通过注解为 API文档添加注释，方便理解
 *****/
@Data
@Api("注释")
@ApiModel("用户表")
public class User {

    @ApiModelProperty("用户ID")
    private Long id;
    @ApiModelProperty("用户名")
    private String username;
    @ApiModelProperty("密码")
    private String password;
}
```

- **控制层**

```
package com.jerusalem.swagger.controller;

import com.jerusalem.swagger.pojo.User;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import io.swagger.annotations.ApiParam;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/****
 * @Author:jerusalem
 * @Description: 控制层
 *****/
@Api("注释")
@RestController
@RequestMapping("/user")
public class UserController {

    /***
     * 只要控制层方法的返回值存在实体类，该实体类就会被扫描到
     * @return
     */
    @ApiOperation("获取用户列表")
    @GetMapping("/list")
    public User findAll(@ApiParam("用户名") String username){
        return null;
    }
}
```

### 3. 实战效果图

![](E:\程序人生\个人学习笔记\学习笔记图床\swagger1.png)

![swagger2](E:\程序人生\个人学习笔记\学习笔记图床\swagger2.png)

#### 注意：项目正式发布时，要关闭Swagger。

**详细内容参考Swagger示例项目**