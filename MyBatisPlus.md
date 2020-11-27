## MyBatisPlus

### 1. 快速入门

#### 1.1 添加依赖

```
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.2.0</version>
    </dependency>
```

#### 1.2 添加@MapperScan注解

在 Spring Boot 启动类中添加`@MapperScan`注解，扫描`Mapper`文件夹

```
@SpringBootApplication
@MapperScan("com.jerusalem.xxx.mapper")     // 可以加在配置类中
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(QuickStartApplication.class, args);
    }
}
```

#### 1.3 添加配置类

```
@EnableTransactionManagement
@MapperScan("com.jerusalem.xxx.mapper")
@Configuration
public class MybatisPlusConfig {

    @Bean     // 配置分页插件（必备）
    public PaginationInterceptor paginationInterceptor(){
        
        PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
        // 设置请求的页面大于最大页后操作， true调回到首页，false 继续请求  默认false
        // paginationInterceptor.setOverflow(false);
        // 设置最大单页限制数量，默认 500 条，-1 不受限制
        // paginationInterceptor.setLimit(500);
        return new PaginationInterceptor();
    }
    
    @Bean     // Oracle的序列生成器
    
    @Bean     // 注入自定义的拦截器
    public MyInterceptor myInterceptor(){
        return  new MyInterceptor();
    }
    
    @Bean     // 攻击 SQL 阻断解析器
    public SqlExplainInterceptor sqlExplainInterceptor() {

        SqlExplainInterceptor sqlExplainInterceptor = new SqlExplainInterceptor();
        List<ISqlParser> list = new ArrayList<>();
        // 攻击 SQL 阻断解析器，加入解析链
        list.add(new BlockAttackSqlParser());           // 全表更新、删除的阻断器
        sqlExplainInterceptor.setSqlParserList(list);
        return sqlExplainInterceptor;
    }
    
    @Bean    // 性能分析插件
    
    @Bean    // 乐观锁插件
    public OptimisticLockerInterceptor optimisticLockerInterceptor() {
        return new OptimisticLockerInterceptor();
    }
    
    @Bean   // SQL 注入器
    public MySqlInjector mySqlInjector(){
        return new MySqlInjector();
    }
}
```

#### 1.4 常用配置 (yml文件)

```
mybatis-plus:
  config-location: classpath:mybatis-config.xml     # 指定全局的配置文件（不推荐使用，推荐在yml文件中统一配置）
  mapper-locations: classpath*:mybatis/*.xml     # 指定 Mapper.xml文件的路径（Maven多模块项目的扫描路径需以classpath*:开头即加载多个jar包下的XML文件)
  type-aliases-package: com.jerusalem.entity     # 实体对象的扫描包
  configuration:
    map-underscore-to-camel-case: false     # 禁用自定的驼峰映射（与 Configlocation不能共存）
    cache-enabled: false     # 禁用缓存
  global-config:
    db-config:
      id-type: auto     # 全局的 id 生成策略
      table-prefix: tb_     # 全局的表名前缀
      logic-delete-value: 1    # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0    # 逻辑未删除值(默认为 0)
  typeEnumsPackage: com.baomidou.springboot.entity.enums    # 通用枚举
  
```

#### 1.5 常用注解

```
@Data
@ToString
@TableName("tbl_employee")   // 映射数据库相应表
public class Employee {

    @TableId(type = IdType.AUTO)   // id增长策略
    private Integer id;
    @TableField(value = "username")  // 指定数据库表中的字段名
    private String name;
    @TableField(exist = false)  // 在数据库表中是不存在的
    private String password;
    @TableField(select = false)   // 查询时不返回该字段的值
    private String email;
    private Integer gender;
    private Integer age;
}
```

### 2. 通用CRUD

![图片](E:\程序人生\个人学习笔记\学习笔记图床\MP-CRUD.jpeg)

#### 2.1 插入操作

```
@Test
    public void testInsert(){
        Employee employee = new Employee();
        employee.setUsername("吃鸡");
        employee.setAge(82);
        employee.setEmail("1234886@11.com");
        employee.setGender(2);
        this.employeeMapper.insert(employee);
    }
```

#### 2.2 更新操作

- **根据 id 更新**

```
    @Test
    public void testUpdateById(){                      
        Employee employee = new Employee();
        employee.setId(5);
        employee.setUsername("王者荣耀");
        employee.setAge(18);
        employee.setGender(2);
        this.employeeMapper.updateById(employee);
    }
```

- **根据条件更新**

```
    @Test
    public void testUpdate(){                           
        Employee employee = new Employee();
        employee.setAge(28);
        employee.setEmail("327658@163.com");
        QueryWrapper<Employee> queryWrapper = new QueryWrapper<>();
        queryWrapper.eq("username","Tom");
        this.employeeMapper.update(employee,queryWrapper);

    }

    @Test
    public void testUpdate2(){
        UpdateWrapper<Employee> updateWrapper = new UpdateWrapper<>();
        updateWrapper.set("age",50).set("email","3333@bb.com")
                .eq("username","哈哈哈ha").eq("age",25);                // 使用数据库中的字段名，而不是属性名
        this.employeeMapper.update(null,updateWrapper);
    }
```

#### 2.3 删除操作

```
    @Test
    public void testDeleteById(){                      // 根据 id 删除
        this.employeeMapper.deleteById(3);
    }

    @Test
    public void testDeleteByMap(){
        Map<String,Object> map = new HashMap<>();    //根据map删除数据，多条件之间是and关系
        map.put("username","Jerry");
        map.put("age",50);
        this.employeeMapper.deleteByMap(map);
    }

    @Test
    public void testDelete(){                       // 根据条件删除
        QueryWrapper<Employee> queryWrapper = new QueryWrapper<>();
        queryWrapper.eq("username","哈哈哈ha").eq("age",50);
        this.employeeMapper.delete(queryWrapper);
    }

    @Test
    public void testDelete2(){                   // 根据包装条件删除
        Employee employee = new Employee();
        employee.setUsername("哈哈哈");
        employee.setAge(22);
        QueryWrapper<Employee> queryWrapper = new QueryWrapper<>(employee);
        this.employeeMapper.delete(queryWrapper);
    }

    @Test
    public void testDeleteBatchIds(){           // 根据id集合批量删除
        this.employeeMapper.deleteBatchIds(Arrays.asList(10,12));
    }
```

#### 2.4 查询操作

```
    @Test
    public void testSelectById(){
        // 根据 id 查询
        Employee employee = this.employeeMapper.selectById(1);
        System.out.println(employee);
    }

    @Test
    public void testSelectBatchIds(){
        // 根据 id 集合批量查询
        List<Employee> employees = this.employeeMapper.selectBatchIds(Arrays.asList(5,6,8,9));
        for (Employee employee : employees) {
            System.out.println(employee);
        }
    }

    @Test
    public void testSelectOne(){
        QueryWrapper<Employee> queryWrapper = new QueryWrapper<>();
        // 根据条件查询一条数据，查询结果超过一条会报错
        queryWrapper.eq("username","吃鸡").eq("email","156@163.com").eq("age",52);
        Employee employee = this.employeeMapper.selectOne(queryWrapper);
        System.out.println(employee);
    }

    @Test
    public void testSelectCount(){
        // 根据条件查询数据的条数
        QueryWrapper<Employee> queryWrapper = new QueryWrapper<>();
        queryWrapper.eq("username","吃鸡");
        Integer selectCount = this.employeeMapper.selectCount(queryWrapper);
        System.out.println(selectCount);
    }

    @Test
    public void testSelectList(){
        // 根据条件批量查询
        QueryWrapper<Employee> queryWrapper = new QueryWrapper<>();
        queryWrapper.like("email","123456");
        List<Employee> employees = this.employeeMapper.selectList(queryWrapper);
        for (Employee employee: employees) {
            System.out.println(employee);
        }
        System.out.println();
        System.out.println();
        QueryWrapper<Employee> queryWrapper1 = new QueryWrapper<>();
        queryWrapper1.gt("age",50);
        List<Employee> employees1 = this.employeeMapper.selectList(queryWrapper1);
        for (Employee employee: employees1
             ) {
            System.out.println(employee);
        }
    }

    // 测试分页查询
    @Test
    public void testSelectPage(){
        Page<Employee> page = new Page<>(1,1);     // 查询第一页，查询一条数据
        QueryWrapper<Employee> queryWrapper = new QueryWrapper<>();
        queryWrapper.gt("age",50);
        IPage<Employee> iPage = this.employeeMapper.selectPage(page,queryWrapper);
        System.out.println("数据总条数为："+iPage.getTotal());
        System.out.println("数据总页数为："+iPage.getPages());
        System.out.println("当前页数为："+iPage.getCurrent());
        List<Employee> employees = iPage.getRecords();
        for (Employee employee: employees
             ) {
            System.out.println(employee);
        }
    }
```

#### 2.5 SQL 注入原理

[SQL注入原理](https://www.bilibili.com/video/av71464838?p=26)

### 3. 条件构造器

#### 3.1 AllEq的使用

```
    @Test
    public void testAllEq() {
        Map<String, Object> map = new HashMap<>();
        map.put("username","詹姆斯");
        map.put("email","123456@163.com");
        map.put("age",null);
        QueryWrapper<Employee> queryWrapper = new QueryWrapper<>();
        QueryWrapper<Employee> queryWrapper1 = new QueryWrapper<>();
        QueryWrapper<Employee> queryWrapper2 = new QueryWrapper<>();
        queryWrapper.allEq(map);     // null作为查询条件
        queryWrapper1.allEq(map,false);     // null不作为查询条件
        queryWrapper2.allEq((k,v) -> (k.equals("username") || k.equals("email")),map);     // 是否作为查询条件
        List<Employee> employees = this.employeeMapper.selectList(queryWrapper);
        List<Employee> employees1 = this.employeeMapper.selectList(queryWrapper1);
        List<Employee> employees2 = this.employeeMapper.selectList(queryWrapper2);
        for (Employee employee: employees) {
            System.out.println(employee);
        }
        System.out.println();
        for (Employee employee: employees1 ) {
            System.out.println(employee);
        }
        System.out.println();
        for (Employee employee: employees2) {
            System.out.println(employee);
        }
    }
```

#### 3.2 基本比较操作

```
    @Test
    public void testEq(){
        QueryWrapper<Employee>  queryWrapper = new QueryWrapper<>();
        queryWrapper.eq("username","吃鸡");                  // 等于
        queryWrapper.ne("email","123456@163.com");          // 不等于
        queryWrapper.gt("age",20);          // 大于
        queryWrapper.ge("age",21);          // 大于等于
        queryWrapper.lt("age",50);          // 小于
        queryWrapper.le("age",40);          // 小于等于
        queryWrapper.in("id",3,6,8);            // 在...内
        queryWrapper.notIn("id",1,2,3);          // 不在...内
        queryWrapper.between("age",20,50);          // 在...之间
        queryWrapper.notBetween("age",25,35);       // 不在...之间
    }
```

#### 3.3 模糊查询

```
    @Test
    public void testLike(){
        QueryWrapper<Employee> queryWrapper = new QueryWrapper<>();
        queryWrapper.like("username","哈哈");             // 像    
        queryWrapper.notLike("username","haha");         // 不像
        queryWrapper.likeLeft("username","吃");          // 像左边
        queryWrapper.likeRight(("username","鸡");        // 像右边
        List<Employee> employees = this.employeeMapper.selectList(queryWrapper);
    }
```

#### 3.4 排序查询

```
    @Test
    public void testOrderBy(){
        QueryWrapper<Employee> queryWrapper = new QueryWrapper<>();
        queryWrapper.orderBy("true","true","id","age");      // 有点问题
        queryWrapper.orderByAsc("age");             // 正序排列
        queryWrapper.orderByDesc("age");            // 倒序排列
        List<Employee> employees = this.employeeMapper.selectList(queryWrapper);
        for (Employee employee: employees
             ) {
            System.out.println(employee);
        }
    }
```

#### 3.5 逻辑查询

```
    @Test
    public void testOrAnd(){
        QueryWrapper<Employee> queryWrapper = new QueryWrapper<>();
        QueryWrapper<Employee> queryWrapper1 = new QueryWrapper<>();
        queryWrapper.eq("username","吃鸡").or().eq("email","123456@163.com");
        queryWrapper1.eq("username","吃鸡").eq("email","123456@163.com");
        List<Employee> employees = this.employeeMapper.selectList(queryWrapper);
        List<Employee> employees1 = this.employeeMapper.selectList(queryWrapper1);
        for (Employee employee: employees
             ) {
            System.out.println(employee);
        }
        System.out.println();
        for (Employee employee: employees1
             ) {
            System.out.println(employee);
        }
    }
```

#### 3.6 Select（指定查询字段）

```
    @Test
    public void testSelect(){
        QueryWrapper<Employee> queryWrapper = new QueryWrapper<>();
        queryWrapper.eq("username","吃鸡").eq("email","123456@163.com")
                    .select("id","username","age");
        List<Employee> employees = this.employeeMapper.selectList(queryWrapper);
        for (Employee employee: employees
        ) {
            System.out.println(employee);
        }
        System.out.println();
    }
```

### 4. ActiveRecord

#### 4.1 说明

```
@Data
@ToString
@TableName("tbl_employee")   
public class Employee extends Model<Employee> {         // 继承 Model (AR)
    
    private Integer id;
    private String username;
    private String email;
    private Integer gender;
    private Integer age;
}
```

#### 4.2 CRUD 操作

```
@SpringBootTest
@RunWith(SpringJUnit4ClassRunner.class)
public class TestMPAR {

    // 无需注入相应的 Mapper

    @Test
    public void testSelectById(){
        Employee employee = new Employee();
        Employee employee1 = employee.selectById(5);
        System.out.println(employee1);
    }

    // 新增数据
    @Test
    public void testInsert(){
        Employee employee = new Employee();
        employee.setUsername("霍华德");
        employee.setGender(2);
        employee.setAge(35);
        employee.setEmail("huohuade@163.com");
        employee.insert();
    }

    // 更新数据
    @Test
    public void testUpdate(){
        Employee employee = new Employee();
        // 根据 id 更新
        employee.setId(8);                      // 更新条件
        // 修改内容
        employee.setUsername("库里");
        employee.setEmail("curry666@163.com");
        employee.setAge(31);
        employee.updateById();
        // 根据条件更新
        Employee employee1 = new Employee();
        UpdateWrapper<Employee> updateWrapper = new UpdateWrapper<>();
        updateWrapper.set("username","戴维斯").set("age",26).set("email","davis@163.com")
                    .eq("username","戴维斯").eq("email","curry666@163.com");
        employee1.update(updateWrapper);
    }

    // 删除数据
    @Test
    public void testDeleteById(){
        Employee employee = new Employee();
        employee.deleteById(11);
    }

    // 根据条件查询数据
    @Test
    public void testSelect(){
        Employee employee = new Employee();
        QueryWrapper<Employee> queryWrapper = new QueryWrapper<>();
        queryWrapper.gt("age",30);
        List<Employee> employees = employee.selectList(queryWrapper);
        for (Employee employee1: employees
             ) {
            System.out.println(employee1);
        }
    }
    
    // 根据 id 查询数据
    @Test
    public void testSelectById(){
        Employee employee = new Employee();
        Employee employee1 = employee.selectById(5);
        System.out.println(employee1);
    }
}
```

### 5. 插件

#### 5.1 Mybatis的插件机制

- **拦截器**

1. 拦截执行器的方法
2. 拦截参数的处理
3. 拦截结果集的处理
4. 拦截Sql语法构建的处理

有如下两种配置方法：

```
// InterceptorPlugin.java
@Intercepts({@Signature(
        type= Executor.class,
        method = "update",
        args = {MappedStatement.class,Object.class})})
public class MyInterceptor implements Interceptor {

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        // 拦截方法，具体业务逻辑编写的位置
        return invocation.proceed();
    }

    @Override
    public Object plugin(Object target){
        // 创建target对象的代理对象，目的是将当前拦截器加入到该对象中
        return Plugin.wrap(target,this);
    }

    @Override
    public void setProperties(Properties properties) {
        // 属性设置
    }
}
<!-- mybatis-config.xml -->
<plugins>
  <plugin interceptor="org.mybatis.example.ExamplePlugin">
    <property name="someProperty" value="100"/>
  </plugin>
</plugins>
```

默认情况下，MyBatis 允许使用插件来拦截的方法调用包括：

- **Executor** (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
- **ParameterHandler** (getParameterObject, setParameters)
- **ResultSetHandler** (handleResultSets, handleOutputParameters)
- **StatementHandler** (prepare, parameterize, batch, update, query)

#### 5.2 执行分析插件

仅适用于开发环境

#### 5.3 性能分析插件

仅适用于开发环境

#### 5.4 乐观锁插件

##### 5.4.1 配置

```
    @Bean    // 乐观锁插件
    public OptimisticLockerInterceptor optimisticLockerInterceptor() {
        return new OptimisticLockerInterceptor();
    }
```

##### 5.4.2 注解实体字段

```
@Version
private Integer version;
```

特别说明：

```
1.支持的数据类型只有:int,Integer,long,Long,Date,Timestamp,LocalDateTime   
2.整数类型下 newVersion = oldVersion + 1
3.newVersion 会回写到 entity 中
4.仅支持 updateById(id) 与 update(entity, wrapper) 方法
5.在 update(entity, wrapper) 方法下, wrapper 不能复用!
```

### 6. SQL 注入器

#### 6.1 创建一个自己的通用`MyBaseMapper`接口来定义方法

```
package com.jerusalem.xxx.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;

public interface MyBaseMapper<T> extends Basemapper<T> {
    
    ···   //扩展方法
}
```

#### 6.2 创建一个自己的Sql注入器`MySqlInjector`类

```
package com.jerusalem.xxx.injectors;

import com.baomidou.mybatisplus.core.injector.AbstractSqlInjector;

public class MySqlInjector extends DefaultSqlInjector {

    /**
     * 如果只需增加方法，保留 MP自带方法
     * 可以 super.getMethodList() 再 add
     */

    @Override
    public List<AbstractMethod> getMethodList(){
        //扩充自定义方法,并获取父类中的集合
        List<AbstractMethod> methodList = super.getMethodList();
        methodList.add(new 方法名());
        return mothodList;
    }
}
```

#### 6.3 编写自己的方法类，继承 AbstractMethod

```
package com.jerusalem.xxx.injectors;

impot ···;

public class MysqlInsertAllBatch extends AbstractMethod {

    @Override
    public MappedStatement injectMappedStatement(Class<?> mapperClass, Class<?> modelClass, TableInfo tableInfo) {
        String sqlMethod = "方法名";
        String sql = "sql语句";
        SqlSource sqlSource = languageDriver.createSqlSourse(Configuration,sql,modelClass);
        return this.addSelectMappedStatement(mapperClass, "方法名",sqlSource,modelClass,tableInfo);
    }
```

#### 6.4 注入 Spring 容器

```
  @Bean
    public MySqlInjector mySqlInjector(){
        return new MySqlInjector();
    }
```

### 7. 代码生成器

#### 7.1 添加相关依赖

```
<!-- 代码生成器依赖 -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.2.0</version>
</dependency>

<!-- 引入模板引擎（多选一）-->
<!-- Freemarker -->
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.29</version>
</dependency>

<!-- Velocity -->
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>2.1</version>
</dependency>

<!-- Beetl -->
<dependency>
    <groupId>com.ibeetl</groupId>
    <artifactId>beetl</artifactId>
    <version>3.0.15.RELEASE</version>
</dependency>
```

**注意：**

```
用户可以选择自己熟悉的模板引擎，如果选择了非默认引擎，需要在 AutoGenerator 中设置模板引擎。
```

#### 7.2 模板配置文件

```
package com.jerusalem.mybatisplus.generator;

import com.baomidou.mybatisplus.core.exceptions.MybatisPlusException;
import com.baomidou.mybatisplus.core.toolkit.StringPool;
import com.baomidou.mybatisplus.core.toolkit.StringUtils;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.InjectionConfig;
import com.baomidou.mybatisplus.generator.config.*;
import com.baomidou.mybatisplus.generator.config.po.TableInfo;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
import com.baomidou.mybatisplus.generator.engine.FreemarkerTemplateEngine;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class MyGenerator {

    /**
     * <p>
     * 读取控制台内容
     * </p>
     */
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder help = new StringBuilder();
        help.append("请输入" + tip + "：");
        System.out.println(help.toString());
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotEmpty(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }

    public static void main(String[] args) {
        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir(projectPath + "/src/main/java/");
        gc.setAuthor("jerusalem");
        gc.setOpen(false);
        // gc.setSwagger2(true); 实体属性 Swagger2 注解
        mpg.setGlobalConfig(gc);

        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/blog?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8");
        // dsc.setSchemaName("public");
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("123456");
        mpg.setDataSource(dsc);

        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setModuleName(scanner("模块名"));
        pc.setParent("com.jerusalem.mybatisplus");
        mpg.setPackageInfo(pc);

        // 自定义配置
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };
        // 如果模板引擎是 freemarker
        String templatePath = "/templates/mapper.xml.ftl";
        // 如果模板引擎是 velocity
        // String templatePath = "/templates/mapper.xml.vm";

        // 自定义输出配置
        List<FileOutConfig> focList = new ArrayList<>();
        // 自定义配置会被优先输出
        focList.add(new FileOutConfig(templatePath) {

            @Override
            public String outputFile(TableInfo tableInfo) {
                // 自定义输出文件名 ， 如果你 Entity 设置了前后缀、此处注意 xml 的名称会跟着发生变化！！
                return projectPath + "/src/main/resources/mapper/" + pc.getModuleName()
                        + "/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
            }
        });
        /*
        cfg.setFileCreate(new IFileCreate() {
            @Override
            public boolean isCreate(ConfigBuilder configBuilder, FileType fileType, String filePath) {
                // 判断自定义文件夹是否需要创建
                checkDir("调用默认方法创建的目录");
                return false;
            }
        });
        */
        cfg.setFileOutConfigList(focList);
        mpg.setCfg(cfg);

        // 配置模板
        TemplateConfig templateConfig = new TemplateConfig();

        // 配置自定义输出模板
        //指定自定义模板路径，注意不要带上.ftl/.vm, 会根据使用的模板引擎自动识别
        // templateConfig.setEntity("templates/entity2.java");
        // templateConfig.setService();
        // templateConfig.setController();

        templateConfig.setXml(null);
        mpg.setTemplate(templateConfig);

        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        strategy.setSuperEntityClass("com.baomidou.mybatisplus.extension.activerecord.Model");
        strategy.setEntityLombokModel(true);
        strategy.setRestControllerStyle(true);
        // 公共父类
        //strategy.setSuperControllerClass("com.baomidou.ant.common.BaseController");
        // 写于父类test中的公共字段
        strategy.setSuperEntityColumns("id");
        strategy.setInclude(scanner("表名，多个英文逗号分割").split(","));
        strategy.setControllerMappingHyphenStyle(true);
        strategy.setTablePrefix(pc.getModuleName() + "tb_");
        mpg.setStrategy(strategy);
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());
        mpg.execute();
    }

}
```

### 8. 自动填充功能

#### 8.1 注解填充字段

```
public class User {

    // 注意！这里需要标记为填充字段
    @TableField(.. fill = FieldFill.INSERT)
    private String fillField;

    ....
}
```

#### 8.2 自定义实现类`MyMetaObjectHandler`

```
package com.jerusalem.xxx.handler;

@Component
public class MyMetaObjectHandler implements MetaObjectHandler {

    private static final Logger LOGGER = LoggerFactory.getLogger(MyMetaObjectHandler.class);

    @Override
    public void insertFill(MetaObject metaObject) {
        Object xxx = getFieldValByName("xxx",metaObject)
        if(xxx == null){
            //字段为空，可以进行补充
            setFieldValByName("xxx", "xxx的内容", metaObject);
        }
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        
    }
}
```

注意事项：

1. 字段必须声明 `TableField` 注解，属性 `fill` 选择对应策略，该申明告知 Mybatis-Plus 需要预留注入 SQL 字段
2. 填充处理器 `MyMetaObjectHandler` 在 Spring Boot 中需要声明 `@Component` 注入
3. 必须使用父类的 `setFieldValByName()` 或者 `setInsertFieldValByName` `setUpdateFieldValByName`方法，否则不会根据注解 `FieldFill.xxx` 来区分

### 9. 逻辑删除

#### 9.1 配置

```
mybatis-plus:
  global-config:
    db-config:
      logic-delete-value: 1   # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0   # 逻辑未删除值(默认为 0)
```

#### 9.2 修改表结构

```

```

#### 9.3 实体类字段上加上 `@TableLogic` 注解

```
@TableLogic
private Integer deleted;
```

**说明**

```
1.逻辑删除是为了方便数据恢复和保护数据本身价值等等的一种方案，但实际就是删除。
2.如果你需要再查出来就不应使用逻辑删除，而是以一个状态去表示。
3.若确需查找删除数据，如老板需要查看历史所有数据的统计汇总信息，请单独手写sql。
```

### 10. 通用枚举

#### 10.1 配置

```
mybatis-plus:
    # 支持统配符 * 或者 ; 分割
    typeEnumsPackage: com.baomidou.springboot.entity.enums
```

#### 10.2 申明通用枚举属性

方式一：使用 `@EnumValue` 注解枚举属性

```
public enum GradeEnum {

    PRIMARY(1, "小学"),  SECONDORY(2, "中学"),  HIGH(3, "高中");

    GradeEnum(int code, String descp) {
        this.code = code;
        this.descp = descp;
    }

    @EnumValue   // 标记数据库存的值是code
    private final int code;
}
```

方式二：

枚举属性，实现 IEnum 接口（推荐）

```
package com.jerusalem.xxx.enums;

import ···;

public enum AgeEnum implements IEnum<Integer> {
    ONE(1, "一岁"),
    TWO(2, "二岁"),
    THREE(3, "三岁");
    
    private int value;
    private String desc;
    
    @Override
    public Integer getValue() {
        return this.value;
    }
    
    @Override
    public Integer getDesc() {
        return this.desc;
    }
}
```

实体属性使用枚举类型

```
public class User{
    /**
     * 名字
     * 数据库字段: name varchar(20)
     */
    private String name;
    
    /**
     * 年龄，IEnum接口的枚举处理
     * 数据库字段：age INT(3)
     */
    private AgeEnum age;
        
        
    /**
     * 年级，原生枚举（带{@link com.baomidou.mybatisplus.annotation.EnumValue}):
     * 数据库字段：grade INT(2)
     */
    private GradeEnum grade;
}
```

#### 10.3 JSON序列化处理

- **Jackson**

```
在需要响应描述字段的 get 方法上添加 @JsonValue 注解即可
```

- **Fastjson**

```
详情请见 MyBatis-Plus官方文档
```

### 11. MybatisX 快速开发插件

- **Java 与 XML 来回跳转**

- **Mapper 方法自动生成 XML**