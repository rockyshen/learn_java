[[Spring家族]]的一员！
SpringBoot是相当于简化[[SSM]]框架的开发吧，本质上还是SSM框架

本笔记基于版本：SpringBoot 3.0.5
[SpringBoot说明文档](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html)

SpringBoot的好处
1、快速创建独立Spring应用（SSM、导包、写配置、启动运行）
2、直接嵌入了Tomcat、Jetty等servlet应用服务器
	之前：Linux上安装java、tomcat、mysql，把war包放在tomcat/webapps下
	现在：打成jar包，java -jar直接运行

一个示例对比
**在Spring项目中使用mybatis，需要以下步骤：**
1、引入spring、mybatis、jdbc等依赖
2、创建mybatis-config.xml配置文件：
	声明DataSource
	声明SqlSessionFactoryBean
	声明MapperScannerConfigurer
	声明其他配置
3、编写XxxMapper.java 和 XxxMapper.xml
4、业务代码调用

**在SpringBoot项目中使用mybatis，需要一下步骤：**
1、引入mybatis-spring-boot-starter依赖
2、application.yml中添加DataSource等配置条目
3、编写XxxMapper.java 和 XxxMapper.xml
4、业务代码调用。

对比，明显感觉到Spring Boot使用starter之后，不用管理相关依赖，不用写繁琐的配置文件了！
## 基本概念
配置文件太多，当微服务场景下，一个服务作为一个Module就要写一遍！
开箱即用，提供了大量的默认值，只需要将特定需求进行少量调整即可。
Convention over Configuration 约定大于配置！

1、依赖管理机制
2、自动配置机制

### 流程
1、导入starter-web：导入了web开发场景
	1、场景启动起导入了相关场景的所有依赖：starter-json、starter-tomcat、springmvc
	2、每个场景启动器都引入了一个soring-boot-starter，核心场景启动器
	3、核心场景启动器引入了spring-boot-autoconfigure包
	4、spring-boot-autoconfigure里囊括了所有场景的所有配置
	5、只要这个包下所有类都能生效，那么相当于SpringBoot官方写好的整合功能就生效了
	6、SpringBoot默认却扫描不到spring-boot-autoconfigure下写好的所有配置类（这些配置类给我们做了整合操作），**默认只扫描主程序所在包**。
	⚠️：相当于把这些自动配置引到了我们项目，但是暂时扫描不到我们的ioc容器中哦！
2、主程序：@SpringBootApplication
	1、由三个注解组成：@SpringBootConfiguration + @EnableAutoConfiguration + @ComponentScan
	2、SpringBoot默认SpringBoot只会扫描主程序所在的包及其下的子包，扫描不到spring-boot-autoconfigure包中官方写好的配置类
	3、@EnableAutoConfiguration：SpringBoot开启自动配置的核心！有了这个就能扫描到spring-boot-autoconfigure包中官方写好的142个配置类
		1、是由@Import(AutoConfigurationImportSelector.class)提供功能：批量给容器中倒入组件。
		2、SpringBoot启动会默认加载142个配置类
		3、这142个配置类来自于：`/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration,imports`
		4、项目启动的时候，利用@Import批量导入到项目（自动配置类）
	4、虽然导入了143个自动配置类，但是@ConditionalOnClass，如果没有导入这个依赖包，这个自动配置类就不生效。

3、xxxAutoConfiguration 自动配置类  联想到 ---> [[Spring FrameWork#Spring配置文件 - 配置类]]
	1、给容器中使用@Bean放一堆组件，这些组件就能工作了
	2、每个自动配置类，都可能有这个注解@EnableConfigurationProperties("ServerProperties.class")，用来把配置文件yaml中配的指定的前缀的属性值封装到xxxProperties属性类中。
	3、以tomcat为例，在yaml配置文件中都是以“server”开头的。
	4、给容器中放的所有组件的一些核心参数，都来自于xxxProperties。xxxProperties又是和yaml配置文件绑定的，所以就实现了yaml配置文件可以提供一些情况的参数配置（你不改就用默认值，体现：约定大于配置的理念） ^1795fa

4、写业务，全过程无需关注各种整合（底层已经整合好了）

最终效果：导入`starter`，修改yaml配置文件，就能修改底层行为。

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/IMG_1412.jpg)

### 依赖管理机制  `<parent>`

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/IMG_1402.jpg)

spring-boot-starter-parent  继承自  spring-boot-dependencies
spring-boot- dependencies中约定当前SpringBoot版本3.0.5下，所有依赖对应的版本参考
	`<properties> + <dependencyManagement>`

如果是自己需要的依赖，还是常规的写！

步骤1、创建一个project，必须是empty project！因为我们要在pom.xml中设置`<parent>`，==引入父工程==，而不是读父工程的pom.xml
和之前做法有所区别：
	![[SSM#^295665]]
```
<parent>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-parent</artifactId>  
    <version>3.0.5</version>  
</parent>
```

2024.6.4  随着项目实践，parent标签不一定要写，如果你想维护稳定版本状态，你就写parent，你想自己管理，你自己写dependenciesManagment进行版本管理也可以。
parent标签不是强制的！！
但是写parent是一个好的实践！

### spring boot starter
spring-boot-starter   核心场景启动器（包含一个：==spring-boot-autoconfigure==）
	spring-boot-starter-web   继承自核心场景启动器

开宗明义，starter的好处：
1、可以直接在yaml中写需要修改的配置，没有写的都是约定的，自动创建模块
2、不同应用场景对应不同的starter，往pom里加就行。

autoconfigure的源码中，写了Spring官方的所有配置类涵盖了所有场景的所有配置！
只要这个包下所有的类都能生效，那么相当于SpringBoot官方写好的整合功能就生效了。（不是所有包都生效的）

步骤2、在module的pom中添加web工厂的场景启动器：`spring-boot-starter-web`
```
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-web</artifactId>  
    <!-- 不用写版本,父工程里写好了 -->
</dependency>
```


SpringBoot自动配置原理：怎么实现导一个starter，写一些简单的配置，应用就能跑起来？不用关心[[SSM#SSM整合]]

默认的包扫描规则：
1、@SpringBootApplication标注的类就是主程序类
2、SpringBoot只会扫描主程序所在的包及其下的子包，自动的ComponentScan功能
	@SpringBootApplication(scanBasePackage="自定义其他包")

配置的默认值：
配置文件的所有配置项和配置属性类一一绑定！

#### 常用Starter
Spring Boot Starter
- 是一组预定义的依赖集合，简化配置和构建过程
- 在程序启动时自动配置所需组件和功能（自动扫描Bean）

官方提供的启动器，一般叫 `spring-boot-starter-*`
第三方提供的启动器，一般叫`xxx-spring-boot-starter`

| 启动器                            | 描述                                                          | 我的本地Maven仓库中的版本 |
| ------------------------------ | ----------------------------------------------------------- | --------------- |
| spring-boot-starter            | SpringBoot基础启动器                                             |                 |
| spring-boot-starter-web        | SpringBoot Web工程启动器；<br>自动配置包含：Spring MVC；Tomcat；SpringBoot | 2.7.7           |
| spring-boot-starter-test       | SpringBoot 测试环境启动器                                          |                 |
| spring-boot-starter-jdbc       | 事务相关                                                        |                 |
| mybatis-plus-boot-starter      | mybatis-plus（整合mybatis）启动器                                  |                 |
| spring-boot-starter-data-redis | redis                                                       |                 |
| druid-spring-boot-3-starter    | Druid连接池启动器（不兼容SpringBoot3，<br>要手动指定                        |                 |

常用依赖
```pom.xml

<dependencies>  
    <dependency>        
	    <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-web</artifactId>  
        <!-- 不用写版本,父工程里写好了 -->  
    </dependency>  
  
    <dependency>        
	    <groupId>com.baomidou</groupId>  
        <artifactId>mybatis-plus-boot-starter</artifactId>  
        <version>3.5.3.1</version>  
        <!--            不用到mybatis,已经集成了-->  
    </dependency>  
  
    <!--        数据库相关配置启动器 jdbcTemplate 事务相关-->  
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-jdbc</artifactId>  
    </dependency>  
    
    <!--        druid启动起的依赖-->  
    <dependency>  
        <groupId>com.alibaba</groupId>  
        <artifactId>druid-spring-boot-3-starter</artifactId>  
        <version>1.2.18</version>  
    </dependency>  
    
    <!--        mysql驱动-->  
    <dependency>  
        <groupId>mysql</groupId>  
        <artifactId>mysql-connector-java</artifactId>  
        <version>8.0.28</version>  
    </dependency>  
    
    <dependency>        
	    <groupId>org.projectlombok</groupId>  
        <artifactId>lombok</artifactId>  
        <version>1.18.28</version>  
    </dependency>  
    
    <dependency>        
	    <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-aop</artifactId>  
    </dependency>  

<!--        spring boot测试的依赖-->  
    <dependency>        
	    <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-test</artifactId>  
    </dependency>  
    
<!--        spring boot热部署的依赖-->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-devtools</artifactId>  
            <scope>runtime</scope>  
            <optional>true</optional>  
        </dependency>
  
</dependencies>
```

#### 自定义一个starter
具体示例代码，可去看myapi-client项目模块中的使用。
[[API开放平台#自定义starter]]
[Spring Boot如何实现一个自定义starter](https://mp.weixin.qq.com/s/stbp9QmCtYOUC3mhy-t5dA)
##### 基本步骤
1、引入依赖：spring-boot-configuration-processor
```
<!-- 这个一般整合到spring boot Starter中了，可以不写-->
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-configure</artifactId>  
</dependency>

<!-- 自动生成yml的代码提示-->
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-configuration-processor</artifactId>  
    <optional>true</optional>  
</dependency>
```

2、编写属性类，并加上@ConfigurationProperties
这个类中Field字段，会读取到yml中
```
@Configuration 
@ConfigurationProperties("myapi.client")  
@Data  
public class MyApiClientProperties {  
    //利用@ConfigurationProperties，将配置文件中的参数注入到此组件的Field上  
    private String accessKey;  
    private String secretKey;  
}
```

yupi的做法是把属性类和配置类整合到一起！

3、配置类上加@EnableConfigurationProperties
```
@Configuration
@EnableConfigurationProperties(MyApiClientProperties.class)
public class MyApiClientConfig {

	@Autowired
	private MyApiClientProperties properties;
	  
    @Bean  
    public MyApiClient myApiClient() {  
        MyApiClient myApiClient = new MyApiClient();
        myApiClient.setAccessKey(properties.getAccessKey());
        myApiClient.setSecretKey(properties.getSecretKey());
        return myApiClient;  
    }  
}
```

搭配@ConfigurationProperties注解一起使用。
Spring Boot提供的一个用于启用@ConfigurationProperties绑定的注解。它指示Spring Boot应用程序启用指定的类作为@ConfigurationProperties bean。

当一个类上添加了`@EnableConfigurationProperties`注解之后，Spring Boot会自动将这个类作为配置类，用于绑定`application.properties`或`application.yml`中的属性值到该类的对应属性中。这样，就可以在应用程序中轻松地使用`@ConfigurationProperties`注解将配置文件中的属性值映射到Java类中，实现属性的注入和配置。

通常情况下，`@EnableConfigurationProperties`注解在`@Configuration`注解的类上使用，以便该类能够被Spring Boot自动扫描并应用@ConfigurationProperties。

4、src/main/resources/METE_INF/spring.factories文件
```
#spring boot starter  
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.yupi.myapiclientsdk.MyApiClientConfig
```
maven/install，直接上传到本地maven仓库。如果有测试代码删掉
其他项目引入依赖，在其他项目的application.yml中就可以填写配置信息了！

### 主程序  @SpringBootApplication注解
3、主方法上加@SpringBootApplication注解，并调用静态方法SpringApplication.run()即可。
```
@SpringBootApplication  
public class Main {  
    public static void main(String[] args) {  
        SpringApplication.run(Main.class, args);    
        //固定写法
        1、Init初始化加载ioc容器
        2、启动内置的tomcat服务器
    }  
}
```

@SpringBootApplication = @SpringBootConfiguration + @EnableAutoConfiguration + @ComponentScan
	1、@Configuration，有点类似于SpringIocInit配置类，最后启动的那个配置类！
	2、@EnableAutoConfiguration，自动加载配置类（starter中写好的常规配置文件，自己写的配置类都会加载） 
	3、@ComponentScan，默认扫描Main类所在的所有包！

#### CommandLineRunner接口
函数式接口，只有一个方法run()，在Spring应用程序起起来之后，执行具体方法。

注意，和InitializingBean接口的区别，InitializingBean接口是在Bean创建的生命周期前执行的方法，CommandLineRunner接口是在Spring应用程序起来之后执行的。

## Spring Initializer
IDEA中利用Spring初始化工程
下面有两个SpringBoot的源

| 源                         | 提供     | 备注                              |
| ------------------------- | ------ | ------------------------------- |
| https://start.spring.io/  | 官方提供   | 只支持Java 17<br>只支持SpringBoot 3.X |
| https://start.aliyun.com/ | 阿里巴巴提供 | 支持Jdk<br>支持SpringBoot 2.X       |

## 配置文件
SpringBoot统一配置管理，集中到一个固定位置的固定名字的文件中。
`src/main/resources/application.properties`  或 `application.yml`

### 加载顺序
1、首先加载启动配置，bootstrap.yml
2、再加载应用配置，application.yml
3、接着加载特定命名的配置文件，applicaiton-dev.yml
加载过程中，后加载的会覆盖先加载的配置文件，所以application-pro.yml才会覆盖applicaiton.yml，从而生效！

### properties【不推荐】
application.properties
```
# 使用springboot提供的key,修改我们需要修改的部分！  
server.port=8081  
server.servlet.context-path=/ergouzi

# 自定义配置  
jdbc.name = root  
jdbc.password = p@ssw0rd

对应类中引用就行了
@Value(${jdbc.name})
private String name;
```

properties是key=value的格式，为了保证key的唯一值，有多层：X.X.X.X

[SpringBoot约定的配置信息 -> properties.yaml](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html)

约定大于配置，也就是你不去yaml配置文件中指定，就按默认的来！
### yaml
application.yaml
有层次，可以继承的配置文件格式

注意：一定要识别为小绿叶才能读取，上面那种就读不到！
![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202408231320592.png)

联想到docker-compose配置文件也是用yaml格式！
![[docker#^9f2610]]

```
server:  
  port: 80  
  servlet:  
    context-path:/book

# 集合
gfs:  
  - 高圆圆  
  - 范冰冰  
  - 杨幂
```

application-test.yaml   application-dev.yaml   用==-==分割
搭配profiles指定不同的配置文件！

#### 常用配置
##### 微服务名称
```
# 端口号
server:  
  port: 8002  

# 微服务名称
spring:  
  application:  
    name: cloud-payment-service  

# 指定配置文件：dev, prod, test
  profiles:  
    active: dev          # 指定载入application-dev.yaml的配置文件，默认就是default  
```

##### SpringCloud相关
```

```
##### 数据库连接池
```
  datasource:  
    type: com.alibaba.druid.pool.DruidDataSource  
    driver-class-name: com.mysql.cj.jdbc.Driver  
    url: jdbc:mysql://118.25.46.207:3306/springcloud_db?characterEncoding=utf8&useSSL=false&serverTimezone=GMT%2B8&rewriteBatchedStatements=true&allowPublicKeyRetrieval=true  
    username: root  
    password: AmRI3@n2
```

##### mybatis相关
```
mybatis:  
  mapper-locations: classpath:mapper/*.xml    # mybatis指定xml文件路径，与主启动类的@MapperScan打配合！  
  type-aliases-package: com.atguigu.cloud.pojo  
  configuration:  
    map-underscore-to-camel-case: true
```
##### 日志级别

| 日志级别     | 描述                                              |
| -------- | ----------------------------------------------- |
| TRACE    | 最详细的应用程序执行路径信息                                  |
| DEBUG    | 用于记录程序的调试信息，                                    |
| INFO【默认】 | 用于记录一般的运行信息<br>例如应用程序启动或关闭时记录的信息。               |
| WARN     | 用于记录潜在的问题或警告信息，但这些问题不会立即导致错误。<br>开发者可能需要注意这些信息。 |
| ERROR    | 用于记录错误信息。这些信息表明应用程序已经遇到了错误，<br>可能需要立即进行处理。      |

```
logging:  
  level:  
    root: trace
```


#### @ConfigurationProperties（prefix = "druid"）
将配置文件中属性的值，自动绑定到Bean的属性中，简化配置属性的注入过程。
不用每个都@Value("${druid.name}")，一个个指定了！

![[#^1795fa]]

联想到：[[Spring FrameWork#InitializingBean接口]]

### 多环境配置
[多环境设计-程序员鱼皮]([https://blog.csdn.net/weixin_41701290/article/details/120173283](https://blog.csdn.net/weixin_41701290/article/details/120173283))

| XX                   | XX      |       |
| -------------------- | ------- | ----- |
| application.yml      | 共用配置    |       |
| application-dev.yml  | 开发环境的配置 | 数据库连接 |
| application-test.yml | 测试环境的配置 |       |
| application-prod.yml | 生产环境的配置 |       |
可以让SpringBoot启动时，读取指定后缀的配置文件，例如读取开发环境的配置，读取生产环境的配置。

多环境运行，使用application-prod.xml
```
java -jar .\targer\...\xxx-SNAPSHOT.jar --spring.profiles.active=prod
```


## Actuator
Spring Boot Actuator是Spring Boot提供的一个用于监控和管理应用程序的模块，健康检查、日志监控、指标收集、报警功能等。

截止到目前为止，都没怎么用过！

## Spring基本配置
```
spring:  
  application:  
    name: myapi-client
```

## 整合SpringMVC

### SpringMVC的SpringBoot配置
一般SpringMVC相关配置都在 `11. Server Properties` 下面！
[Common Application Properties](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html)
```
server:  
  port: 10080
```

这里的port就是内置tomcat监听的端口     spring-boot端口  = tomcat端口

默认静态资源路径，只要放在这几个文件夹下，静态资源都能访问。WEB-INF里肯定不能访问！
	/META-INF/resources/
	/resources/
	/static/
	/public/

自定义拦截器，自己手写拦截器，再手写一个WebMvcConfig，自动就会被SpringBoot扫描进去。

## 整合Spring Cloud

### SpringCloud的SpringBoot配置
[[SpringCloud#YAML：GateWay的配置]]
```
spring:
	cloud:  
	  nacos:  
	    discovery:  
	      server-addr: 192.168.5.53:8848  
	      username: nacos  
	      password: nacos
		gateway:
			……
```

## 整合Druid

使用druid需要的依赖，示例
```pom.xml
<dependencies>  

<!--        数据库相关配置启动器 jdbcTemplate 事务相关-->  
	<dependency>  
		<groupId>org.springframework.boot</groupId>  
		<artifactId>spring-boot-starter-jdbc</artifactId>  
	</dependency>  
	
<!--        druid的依赖，对应spring-boot-3 -->  
	<dependency>  
		<groupId>com.alibaba</groupId>  
		<artifactId>druid-spring-boot-3-starter</artifactId>  
		<version>1.2.18</version>  
	</dependency>  
	
<!--        mysql驱动-->  
	<dependency>  
		<groupId>mysql</groupId>  
		<artifactId>mysql-connector-java</artifactId>  
		<version>8.0.28</version>  
	</dependency> 
</dependencies>
```

spring boot 2需要用druid的这个版本
```
<dependency>  
    <groupId>com.alibaba</groupId>  
    <artifactId>druid-spring-boot-starter</artifactId>  
    <version>1.2.23</version>  
</dependency>
```

### mysql的SpringBoot配置
```
spring:  
  datasource:  
    type: com.alibaba.druid.pool.DruidDataSource    # 指定使用Druid连接池  
    druid:  
      username: root  
      password: AmRI3@n2  
      driver-class-name: com.mysql.cj.jdbc.Driver  
      url: jdbc:mysql://118.25.46.207:3306/mybatis_example
```

```
druid:  
	# 当关掉空闲连接时，需要保留的有效连接；联想到线程池的核心线程数
	min-idle: 5  
	
	# 连接池里允许的最大活跃连接数，超过这个数就不能连进去了，默认是8
	max-active: 20
	
	# 最大等待时间，应用线程等待连接的超时，超过这时间，就判定为超时，单位毫秒，1秒
	max-wait: 60000
	
	# 一个连接如果5分钟空闲，就应该被回收掉，单位是毫秒，5分钟
	min-evictable-idle-time-millis: 300000 
	
	# 多久进行一次扫描需要关闭的空闲连接，单位是毫秒，，1分钟
	time-between-eviction-runs-millis: 60000
	
	# 初始化时建立物理连接的个数  
	initial-size: 5  
```

```
 


下面不常用，用默认即可！
 |
 v
      # 申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunMillis，执行validationQuery检测连接是否有效  
      test-while-idle: true  
      
      # 用来检测数据库连接是否有效的sql，必须是一个查询语句（oracle中为select 1 from dual)  
      validation-query: select 1  
      
      # 申请连接时会执行validationQuery检测连接是否有效，开启会降低性能，默认为true 
      test-on-borrow: false  
      
      # 归还连接时会执行validationQuery检测连接是否有效，开启会降低性能，默认为true 
      test-on-return: false 
       
      # 是否缓存preparedStatement，也就是PSCache，PSCache对支持游标的数据库性能提升巨大，比如说Oracle,在mysql中建议关闭  
      pool-prepared-statements: false 
       
      # 要启用PSCache,必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true，在Druid中，不会存在Oracle下PSCacheXXX  
      max-pool-prepared-statement-per-connection-size: -1  
      
      # 合并多个DruidDataSource的监控数据  
      use-global-data-source-stat: true
```

在`resources/META-INF/spring`下创建这个文件名：`org.springframework.boot.autoconfigure.AutoConfiguration.imports`
```
com.alibaba.druid.spring.boot3.autoconfigure.DruidDataSourceAutoConfigure
```
作用就是告诉SpringBoot加载Druid这个配置类。因为Druid没有适配SpringBoot3的Starter

==TODO ： Druid 1.2.20已经修复了这个问题，测试通过==


## 整合mybatis
Step 1：导入mybatis的启动器
mybatis整合SpringBoot的依赖，就用这个版本，本地maven仓库已有！避免重复下载
```
<dependency>  
    <groupId>org.mybatis.spring.boot</groupId>  
    <artifactId>mybatis-spring-boot-starter</artifactId>  
    <version>3.0.1</version>
</dependency>
```

^d8c52e


Step 2：mybatis相关的yaml
mybatis 彻底抛弃mybatis-config.xml，所有的设置都在这里指定 ！

### mybatis的SpringBoot配置
主要干两方面的事情  
	1、mybatis设置`<settings>`、别名`<typeAliases>`、插件`<Plugins>`  
	2、druid连接池数据库信息`<environments>`    -->  也可以在这里连接mysql[[#整合DruidDataSource]]
	3、mapper.xml的位置指定`<mapper resource="">  
```
mybatis:  
  mapper-locations: classpath:/mapper/*.xml    # 配置中指定映射文件！
  # 与主启动类的@MapperScan打配合！@SpringBootApplication里包含了MapperScan
  
  type-aliases-package: com.atguigu.pojo  
  configuration:  
    map-underscore-to-camel-case: true      # 开启驼峰命名
    auto-mapping-behavior: full  
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

Step 3： 在主程序启动类上，加==@MapperScan==(base-package)注解。
指定Mapper接口所在的位置。
将Mapper接口加入到ioc容器，可以利用注入来使用Mapper层的方法。

Spring Boot中，AutoConfiguration类自动扫描，@MapperScan定制性更高！
示例：
```
@SpringBootApplication
@MapperScan("com.atguigu.mapper")
public class Main{
	public static void main()
}
```

mapper通过启动类扫描；xml通过yaml激活！
之前是利用mybatis-config扫描，需要保证文件层次一致，现在不需要了，因为是分开扫描的
![[Mybatis#^ec06c7]]


声明式事务，只需要导入jdbc启动器：`spring-boot-starter-jdbc`
启动器内置了：
![[Spring FrameWork#^151dab]]

现在，有了SpringBoot大哥，我们直接@Transactional注解加到方法上就能使用！

AOP整合，导入aop启动器：`spring-boot-starter-aop`
启动器内置了：
![[Spring FrameWork#^e51f84]]

现在，有了SpringBoot大哥，我们直接@Aspect注解加到方法上就能使用！

## 整合mybatis-plus

### mybatis-plus的SpringBoot配置
```
mybatis-plus:  
  configuration:  
    map-underscore-to-camel-case: false
```

注意：mybaits-plus默认开启：mapUnderscoreToCamelCase = true
就会自动将Java实体类的驼峰，转为 _ 下划线
如果Java字段 和 mysql列名 一样的话，就关了
![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202409081441851.png)

## 整合Redis
1、redisson
```
<dependency>  
    <groupId>org.redisson</groupId>  
    <artifactId>redisson-spring-boot-starter</artifactId>  
    <version>3.16.4</version>  
</dependency>
```

2、Jedis，比较简单
```
<dependency>  
	<groupId>redis.clients</groupId>  
	<artifactId>jedis</artifactId>  
	<version>2.9.3</version>  
</dependency>
```
### Redisson的SpringBoot配置
```
spring:
  redis:  
    # 数据库  
    database: 8  
    # 主机  
    host: 192.168.5.52  
    # 端口  
    port: 6379  
    # 密码  
    password: "123456"  
    # 读超时  
    timeout: 5s  
    # 连接超时  
    connect-timeout: 5s  
#  redis:  
#    host: 192.168.5.52  
#    port: 6379  
#    password: 123456  
#    database: 8  
#    jedis:  
#      pool:  
#        max-active: 8 #最大连接数
```

## 整合Logback
只要引入了spring-boot-starter-X，默认都会自动导入spring-boot-starter-logging。（里面就包含了Logback的底层实现）SpringBoot中，默认使用 Logback 作为日志框架
![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202408062042804.PNG)

简化情况下，可以使用[[标准包.类#@slf4j]]临时性的使用！

导入依赖
```
<dependency>  
	<groupId>org.springframework.boot</groupId>  
	<artifactId>spring-boot-starter-logging</artifactId>  
</dependency>
```

千万不要用log4j2，贼麻烦，你看上图，log4j2根本不在体系中，你想用他，还得把spring-boot-starter-logging从父级中排除，然后在引入log4j2，我费这劲干吗！不用！！ ^54c984

### 配置文件
有两种配置方式：1、直接在application.yml中的logging下配置，这是简单、基本配置；
2、如果想要提供更精细化的配置，需要新建一个logback-spring.xml中写入更加精细的日志配置输出。（暂时应该还用不上把！） ^c0dd59

Logback的日志级别：**TRACE、DEBUG、INFO、WARN、ERROR和FATAL** （低  -> 高）
#### application.yml中配置
跟随项目启动时的，简单的日志配置，足够用了！
```
org.springframework.boot.context.logging

logging:  
	level:  
	    root: WARN         # 全局的日志级别是WARN及以上的才会打
	    cn.bitoffer.lottery: ERROR # 本项目的日志级别是ERROR及以上的才会打
	    # 哪些启动INFO不想看，就都忽略了！这样输出的日志比较简单
	file: /path/to/logfile.log
	pattern: "%d{yyyy-MM-dd HH:mm:ss.SSS} [%level] [%thread] [%logger] - %msg%n"

```
#### logback-spring.xml中配置
Logback的高级配置，xml格式

待补充！

## SpringBoot项目部署
普通web项目 -> war包 -> 放到tomcat/webapps

SpringBoot工程(==内置tomcat==)  ->  jar包  ->  terminal命令执行（java -jar)
	注意：即使SpringBoot可以被打包成jar，但是不能被其他项目作为普通jar进行引用依赖。本质原因是内部文件目录结构和普通jar包不一样，打包后的核心代码中`\BOOT-INF\classes`目录下，所以无法被引用依赖。

0、【检查】在pom.xml中必须导入SpringBoot构建的插件！用来构建springboot项目
```
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
		</plugin>

		# 可选，jar包启动时，告诉从哪里开始启动，指定main方法
		<plugin>  
            <groupId>cn.bitstorm.lottery-svr</groupId>  
            <artifactId>bitstorm-lottery-svr</artifactId>  
            <version>0.0.1-SNAPSHOT</version>
            <configuration>
	            <mainClass>cn.bitstorm.MyApplication</mainClass>
	        </configuration>
        </plugin>
        
	</plugins>
</build>
```

1、开发机上定版的项目，执行maven package，jar包输出到target中

2、利用宝塔面板的上传文件功能，将jar包上传到  /www/wwwroot/java 目录中

3、云主机的数据库上提前创建好空表

4、运行jar包
```
nohup java -jar my_project.jar &

java -jar [选项] [参数] -jar文件名

# 临时替换掉application.yaml中的端口的配置
java -jar -Dserver.port=8080 myapp.jar  

# 指定使用测试环境的yaml配置文件
java -jar -Dspring.profiles.active=test myapp.jar
```

多环境运行，使用application-prod.xml
```
java -jar .\targer\...\xxx-SNAPSHOT.jar --spring.profiles.active=prod
```


java程序运行起来后，可以用下面两条命令来检查一下：
1、netstat -ntlp   查看对应端口是不是启动了，对应的进程对不对
2、jsp                   查看所有运行的java进程 ^3015e1


高阶的运行方式，添加JVM参数
-X   设置[[JVM]]参数，例如内存大小、垃圾回收策略等等（高级了）

5、阿里云和宝塔面板，开放springboot项目的端口（防火墙）

6、可以访问（后续再考虑配置域名）

7、下线服务的话：[[linux#ps]]  [[linux#kill]]

Alibaba cloud tool kit 上传部署插件

2024.3.14  在学完SSM的基础上，学习SpringBoot就非常快，一晚上就搞定了！主要理解SpringBoot从哪几个方面替换掉了SSM中的环节即可！

实战项目：微头条
技术栈：SpringBoot + Mybatis_plus

封装结果集
JWT依赖，以及工具类

登录模块

| 需求  | 接口和请求方式            | 请求参数                                                   | controller层方法名 | service层方法名 | mapper层方法名 | 返回值                                                                                          |
| --- | ------------------ | ------------------------------------------------------ | -------------- | ----------- | ---------- | -------------------------------------------------------------------------------------------- |
| 登录  | user/login    POST | {<br>“username”:"zhangsan",<br>"userpwd":"123456"<br>} |                |             |            | {<br>“code”:"200",<br>"message":"success",<br>"data":{<br>      “token”:"...."<br>    }<br>} |
| 2   | bb                 |                                                        |                |             |            |                                                                                              |
| 3   | cc                 |                                                        |                |             |            |                                                                                              |
| 4   | dd                 |                                                        |                |             |            |                                                                                              |

## 常用工具

### [[SpringMVC#统一返回结果类 Result]]

### API文档：Swagger 3
大名鼎鼎的丝袜哥！😄
API测试文档生成工具！

安装使用步骤
1、

配置文件
结果输出到这个页面
http://localhost:8080/swagger-ui.html#/

默认访问地址：微服务启动ip:端口号/swagger-ui.html

| 核心注解                            | XX     |
| ------------------------------- | ------ |
| @Tag(name ,description)         | 标注在类上  |
| @Operation(summary,description) | 标注在方法上 |
| 3                               | cc     |
#### Swagger 2
| 核心注解                       | XX     | 备注                       |
| -------------------------- | ------ | ------------------------ |
| @Api(tags = "")            | 标注在类上  | 这个首页上展示，有用！<br>其他几个没什么用！ |
| @ApiOperation(values = "") | 标注在方法上 |                          |
| @ApiPrams                  | 标注在参数上 |                          |

### API文档：Knife4j整合Swagger
是一个集 [[#Swagger 2]] 和  [[#OpenAPI规范]]3 为一起的增强解决方案。
默认访问地址
```
http://localhost:8101/doc.html
```

[Knife4j 集成SpringBoot指导文档](https://doc.xiaominfo.com/docs/action/springboot)
步骤1：依赖注入
```
<dependency>  
    <groupId>com.github.xiaoymin</groupId>  
    <artifactId>knife4j-spring-boot-starter</artifactId>  
    <version>3.0.3</version>  
</dependency>
```

步骤2：knife4j的配置类
```
@Configuration  
@EnableSwagger2  
@Profile({"dev", "test"})  
public class Knife4jConfig {  
    @Bean  
    public Docket defaultApi2() {  
        return new Docket(DocumentationType.SWAGGER_2)  
                .apiInfo(new ApiInfoBuilder()  
                        .title("接口文档")  
                        .description("myapi-backend")  
                        .version("1.0")  
                        .build())  
                .select()  
                // 指定 Controller 扫描包路径  
                .apis(RequestHandlerSelectors.basePackage("com.yupi.project.controller"))  
                .paths(PathSelectors.any())  
                .build();  
    }  
}
```

![[React#^6eab4a]]
#### OpenAPI规范
[OpenAPI规范（中文版）](https://openapi.apifox.cn/)  OpenAPI 3.0.0是OpenAPI规范的第一个正式版本，2015年从Swagger规范重命名未OpenAPI规范。是一个标准的、与具体编程语言无关的RESFful API的规范，只要你定义的API符合标准，就能利用工具来展示API，代码生成工具基于后端API生成前端API调用。

### 日志工具：Slf4j


@Slf4j是lombak整合的注解（加到类上），直接获取log对象就能使用！


利用Slf4j的原生底层代码，实例化一个log对象，很麻烦！
```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LogDemo {
	private static final Logger log = LoggerFactory.getLogger((LogDemo.class));

	log.error("日志输出")
}
```

直接利用lombak的注解 @Slf4j 就能直接获得一个log对象，进行使用！



### 接口测试：PostMan
接口测试工具！
PostMan   好用！


### 压力测试：Jmeter
基于java环节的，压测工具
模拟高并发环境下的Http请求，计算单个接口的QPS
联想到：[[linux#wrk]]

```
├── 压力测试计划
	├── 线程组         # 一个线程组包含多个线程，每个线程代表一个用户
		├── HTTP请求    # 线程：50  循环次数：1000，模拟5万次
		├── 查看结果树
		├── 汇总报告
		├── 整合报告
		├── 后端监听器
```

[尚硅谷-宋红康-Jmeter配置](https://www.bilibili.com/video/BV1Dz4y1A7FB?p=62&vd_source=72e3f029ff16aef4d56c5c73e18a983f)
#### 使用步骤
1、在中断进入/bin目录，运行sh jmeter启动

2、注意：一个终端、一个GUI界面都不能关

3、创建线程组（测试计划 - 右键 - 添加 - hreads - 线程组）

4、指定：（并发的）线程数，循环次数
取样错误后执行的动作：继续呗
Ramp-up时间：所有线程在多长时间内启动。例如60个线程，Ramp-up=30s，表示30s内起60个线程，也即2秒起一个线程
循环次数：几秒执行一轮

线程数：50
Ramp-Up:1
循环次数：1000
表示：每次50个并发线程，一共循环1000次，总共就是5万次请求。

线程数：5000
Ramp-Up:1
循环次数：1
表示：一次性并发5000个线程，一次全部打过来，总共5000次请求。

调度器-持续时间：测试计划持续多长时间
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-07-02%20at%2023.10.27%402x.png)

5、基于设置好的线程组：添加：取样器 - HTTP请求
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-07-02%20at%2023.11.07%402x.png)

6、配置请求地址、请求头等信息

7、添加：监听器 - 聚合报告 （Aggregate Report)
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-07-02%20at%2023.16.20%402x.png)
顶部小扫把按钮，可以把上一次的报告清掉！


8、顶部 - 绿色按钮 -启动（开启请求）


### 压测工具- 2 [[linux#wrk]]
```
wrk <options> <url>
```

常用参数
-c   --connections  保持的连接数
-d   --duration        持续时间
-t     --threads        线程数（我的mac上，不要超过500
-s    --script            加载lua脚本，必须要在有lua脚本的路径上执行哦
`-H`   --header         加入请求头

--latency   打印延迟统计数据

--timeout     指定超时时间的标准，例如 -T1s  如果1秒内没给响应，就判定为超时，我就走了！

```
wrk -t8 -c100 -d10s -T1s --script=lottery.lua --latency "http://localhost:10080/lottery/v1/get_lucky"

# 开启8个线程，100个连接，持续10秒往这个网址发送POST请求！
```

![CleanShot 2024-07-19 at 12.38.20.png](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-07-19%20at%2012.38.20.png)
