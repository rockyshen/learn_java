[[Spring家族]]的一员！
[SpringCloud指导手册](https://spring.io/projects/spring-cloud)
## 分布式微服务
Ref：[阳哥：02_零基础微服务架构理论入门](https://b23.tv/DsnfUGD)

将复杂的单体应用拆分为一组小型、独立部署的服务，每个服务都围绕特定的业务功能构建。
一个项目（IDEA中一个Project），拆分成多个Module（订单module，支付module），每个微服务都运行在自己的tomcat上，对应自己的mysql数据库，通过HTTP方式互通（SpringCloud），模块之间相互调用，每个模块内部看成一个单一架构的独立应用。
微服务：2014， martin fowler https://martinfowler.com/microservices/

### 微服务架构与SpringCloud
分布式微服务场景下，需要用到SpringCloud，将一个个的微服务（独立SpringBoot开发的module）管理、连接起来！

分布式架构使用：SpringBoot + [[SpringCloud]]，其他中间件（Kafka等消息中间件等 ^e986ae

SpringCloud提供微服务场景下，不同微服务的管理机制，包括以下组件：

### 模块汇总

| XX        | 最新使用                            | 上一代                                         |
| --------- | ------------------------------- | ------------------------------------------- |
| 服务注册与发现   | Consul<br>Alibaba Nacos         | ~~EUREKA（停止维护）~~                            |
| 服务调用和负载均衡 | ~~LoadBalancer~~<br>OpenFeign   | ~~Netflix OSS Ribbon~~<br>~~Netflix Feign~~ |
| 分布式事务     | Alibaba Seata                   |                                             |
| 服务熔断和降级   | Resilience4j<br>Alibaba Sentiel | ~~HYSTRIX~~                                 |
| 服务链路追踪    | Micrometer                      | ~~Sleuth~~                                  |
| 服务网关      | GateWay                         | ~~Netflix Zuul~~                            |
| 分布式配置管理中心 | Consul<br>Alibaba Nacos         |                                             |

基于订单 --调用--> 支付 的base项目，将一个个cloud组件加进去！
1、Settings-Editor-FileEncoding-UTF-8
2、Setting-Build-Compiler-Annotate Processor勾上
3、File Types把.idea、.iml都忽略显示。

### SpringBoot创建微服务小口诀
1、建module
2、改POM
3、写YML
4、主启动
5、业务类


提取通用模块：XXXDTO、untils、Exception是每个微服务都需要用到的。所以maven工具-clean-install加入到maven-reop，之后其他各个微服务模块，引入依赖即可。
```
<!--        引入自己定义的api通用包-->  
<dependency>  
    <groupId>com.atguigu.cloud</groupId>  
    <artifactId>cloud-api-common</artifactId>  
    <version>1.0-SNAPSHOT</version>  
</dependency>
```

前文，我们已经搭建好了2个微服务模块（Order模块 调用 Pay模块）
```
public static final String PaymentSrv_URL = "http://localhost:8001";
```
Order模块 调用 Pay模块，URL写死，Port端口写死，会存在很多问题，需要引入后续内容！微服务的动态注册和发现！

### 注解汇总

| 注解                     | 什么作用                            | 写在哪里                                  | 来自哪个模块       |
| ---------------------- | ------------------------------- | ------------------------------------- | ------------ |
| @EnableDiscoveryClient | 启动时自动发现服务，注册到Consul             | 主启动类                                  | Consul       |
| @RefreshScope          | 启动时，读取consul服务器的全局配置。利用@Value获取 | 主启动类（也可以加到Controller层上）               | Consul       |
| @LoadBalanced          | 客户端一侧开启负载均衡                     | RestTemplate配置类上加                     | Consul       |
| @EnableFeignClient     | 启动时，开启OpenFeign                 | 主启动类                                  | OpenFeign    |
| @FeignClient           | 标注为是Feign客户端                    | 消费者**Consumer的Feign接口上**，指定远程调用的微服务名称 | OpenFeign    |
| @CircuitBreaker        | 安装断路器                           | 业务类的controller层的handler方法上            | Risilience4j |
|                        |                                 |                                       |              |

## 服务发现：Consul
[Consul官方文档](https://developer.hashicorp.com/consul/docs)
[Spring Consul指南](https://docs.spring.io/spring-cloud-consul/docs/current/reference/html/)
服务注册与发现、分布式配置中心
### Consul 服务注册与发现
两个微服务之间相互调用，URL和端口不能写死。我们期望服务名称是确定的，背后的URL+Port是动态变化的！
服务发现和分布式配置管理中心，Go语言开发！
服务发现、健康监测、KV存储、多数据中心、可视化web界面

Step 1:
1、下载完成后，在目录包下只有一个consul.exe文件
2、cmd运行consul --version  能正常显示版本信息就证明与系统匹配
3、cmd运行`consul agent -dev`    启动！(mac环境也是相同的命令，安装在homebrew目录下）
4、访问：http://localhost:8500  进入web-ui

Step 2:
需要加入服务的module中，pom文件引入consul的依赖
```
<!--        consul服务发现  相关依赖-->
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-consul-discovery</artifactId>  
</dependency>
```

改YAML配置
```
spring:
	cloud:  
	  consul:  
	    host: localhost  
	    port: 8500  
	    discovery:  
	      service-name: ${spring.application.name}  # 这里就是动态指定了
```

主启动类上加：`@EnableDiscoveryClient`注解，表示自动扫描发现服务

Controller层上，调用URL，改为consul提供的服务名称即可。

RestTemplateConfig配置类的@Bean后面加：@LoadBalance，因为consul默认开启负载均衡。

### Consul 配置中心
每个微服务共有的YAML配置文件，最好统一在配置中心动态管理！一次通知，处处生效（端口号等）
通用全局配置信息，直接注册到Consul服务器中，各个微服务都从Consul那里获取。

Step 1：引入配置中心的依赖
```
<!--        consul配置中心  相关依赖-->
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-consul-config</artifactId>  
</dependency>  
  
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-bootstrap</artifactId>  
</dependency>
```

Step 2：`bootstrap.yaml` 替换 `Application.yaml`
bootstrap 比 application的优先级更高！
bootstrap.yml的作用就是去读consul的key-value中存储的全局配置信息！
application.yaml的作用是自身定制化的配置信息！

bootstrap.yaml示例代码
```
spring:  
  application:  
    name: cloud-payment-service    # 注册进consul的微服务的名字
     
# consul 配置中心  
  cloud:  
    consul:  
      host: localhost  
      port: 8500  
      discovery:  
        service-name: ${spring.application.name}  
      config:  
        profile-separator: "-"  
        format: YAML
          
        watch:  
          wait-time: 55          # 默认就是55秒刷新
```

Step 3：Web-UI界面下利用Key-Value存储
注意，web-ui写yaml时，打两个空格，不要用tab，否则报错！
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/%E6%88%AA%E5%B1%8F2024-03-20%2000.25.32.png)

Step 4：之后就能在程序中利用`@Value("${atguigu.info}")` 获取到Consul服务器端通用配置的信息。@Value读取基元类型数据，配置信息都可以读到！

Step 5：动态刷新
在主启动类（也可以加到Controller层上）添加@RefreshScope 动态刷新，使服务器配置一修改立马生效。
bootstrap.yaml，官方建议55秒生效

全局配置数据（Key-Value持久化）
Step 1：在consul.exe目录下，创建mydata文件夹、consul_start.bat启动

windows上利用bat启动（以管理员权限）
```
@echo.服务启动......  
@echo off  
@sc create Consul binpath= "D:\devSoft\consul_1.17.0_windows_386\consul.exe agent -server -ui -bind=127.0.0.1 -client=0.0.0.0 -bootstrap-expect  1  -data-dir D:\devSoft\consul_1.17.0_windows_386\mydata"
@net start Consul
@sc config Consul start= AUTO  
@echo.Consul start is OK......success
@pause
```

mac上执行命令，后台运行consul，并进行配置的持久化；
后台运行时，执行：lsof -i :8500  查看运行在8500端口的进程号PID；执行`kill PID`就可以关闭后台运行的consul进程！
[consul启动命令参数](https://www.cnblogs.com/niceyoo/p/14223158.html)
```
nohup consul agent -server -ui -bootstrap-expect=1 -client=0.0.0.0 -bind=127.0.0.1 -data-dir /Applications/consul/mydata &

appending output to nohup.out会将该进程的标准输出（stdout）和标准错误输出（stderr）重定向到一个名为 `nohup.out` 的文件中

以下是日志重定向，不生效，先不管了，能启动就行
>> /Users/shen/Documents/consul/logs
```

我们在上面的配置中，实现了pay模块提供8001和8002两个端口，但是需要手动切换端口，我们需要利用负载均衡，实现自动选择不同的端口。
## 负载均衡：LoadBalancer【经典白雪】
==负载均衡==、服务调用
客户端软件的负载均衡（客户端有自知之明），Nginx是服务器端负载均衡！
[SpringCloud LoadBalancer 官方文档](https://docs.spring.io/spring-cloud-commons/reference/spring-cloud-commons/loadbalancer.html)

[[Spring FrameWork#RestTemplate]] as a LoadBalancer Client。
LoadBalancer是接口，RestTemplate是实现类。

主要有两个负载均衡算法：轮询、随机

因为LoadBalancer是客户端（消费者端）的负载均衡，所以放在消费者一侧，也就是Order模块上！

Step 1：在客户端一侧（Order模块），引入LoadBalancer依赖
```
<!--        负载均衡，客户端一侧-->  
        <dependency>  
            <groupId>org.springframework.cloud</groupId>  
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>  
        </dependency>
```

Step 2：Order模块`config/RestTemplateConfig`配置类中，一方面@Bean加入ioc容器，一方面@LoadBalanced开启负载均衡。
![[Spring FrameWork#^2f2f58]]
```
@Configuration  
public class RestTemplateConfig {  
    //将RestTemplate装进ioc容器中  
  
    @Bean  
    @LoadBalanced     //RestTemplate as a LoadBalancer Client  
    public RestTemplate restTemplate(){  
        return new RestTemplate();  
    }  
}
```

Step 3：Order模块的Controller层，写对应@GetMapping方法！从Order模块调用Pay模块，会自动利用负载均衡切换两个端口。
## 服务调用：OpenFeign
服务接口调用，也包含负载均衡
	**负载均衡通常在 Gateway 层进行**。虽然 OpenFeign 也支持负载均衡功能，但是在实际中更常用的是将负载均衡的责任交给 Gateway，而不是在 OpenFeign 中处理。
只要在REST接口上注解就可以实现！

~~**RestTemplate提出**：微服务提供者Provider（例如Payment），以往需要调用，利用RestTemplate，需要在每一个消费者Consumer中添加RestTemplateConfig配置类中加入ioc容器，然后再到每一个消费者的Controller中注入RestTemplate，再调用方法。你只要有一个Consumer，就得实现一遍，太麻烦了！~~

**OpenFeign提出**：在微服务提供者Provider（例如Payment）前面写一个标注为`@FeignClient`的接口，Provider中需要对外暴露的方法，列到接口中。之后所有想要使用的消费者Consumer（Consumer主启动中开启`@EnableFeignClient`的支持），只要接到接口上，就能使用暴露出来的方法，也不用麻烦Provider大哥。

### 通用步骤
Step 0：Provider模块的application.yaml/cloud/consul/discovery/加一条配置信息：prefer-ip address:true，表示：优先使用服务ip进行注册！

**Step 1**：在调用者Consumer（例如Order模块需要调用Pay模块）的主启动类上，使用==@EnableFeignClient==，启用feign客户端，会去找标注了@FeignClient的接口

basePackages
补充@雷丰阳：
EnableFeignClient如果不指定basepackages，就是扫描主启动类所在的包及其子包中所有，如果添加了basepackage就指定扫这个包里标注@FeignClient

可以加上属性 @EnableFeignClients(basePackages = "com.atguigu.gulimall.member.feign")
缩小寻找范围，提高性能！

**Step 2**：在通用模块`/cloud-api-commons/api/PayFeignApi`接口，
	1、在通用模块的pom中要引入openfeign的依赖！
	2、创建PayFeignApi接口，==@FeignClient==("cloud-payment-service")，这里参数填写需要保护的Provider（微服务名称）！我的视角是Provider（也就是Pay模块)（我们是后端工程师啊）

Step 3：调用者Consumer的Controller层注入Feign接口，然后直接在方法中使用接口提供的方法即可。
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/aed78662907bad7a1604c5489253791e.PNG)
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/1460a3e21625a5a9b6e70f998ccdd99b.PNG)

==**前半段根据接口暴露的方法名对应；后半段根据URL定位！**==

注意：
1、PayFeignApi这个接口，==**我们不用写实现类**==，因为我们加了@FeignClient注解，启动时，Feign使用动态代理机制自动生成此接口的实现类，这个过程隐藏在SpringCloud后台完成，我们==**直接注入这个接口，去使用**==就行了
2、如果Controller层的类上写了@RequestMapping，那在后半段根据URL定位时，需要==将类和方法的URL组合在一起==。   微服务名+完整URL（类URL+方法URL）

@雷神：两个微服务之间调用时，传递数据会转成JSON，方法签名不一定保持一致，只要确保字段能从JSON中对拷出来。

### 高级功能
目前Consumer调用Provider岁月静好，但是万一Provider被打满了，超时了，应该如何优化相互调用时的优化策略呢？见高级功能：

#### 超时控制
OpenFeign默认超时时间是1秒！Consumer通过Feign接口调用Provider，等待60秒不返回结果就报错！
我们需要设置Consumer侧等待超时时间：
	connectTimeout     连接超时时间
	readTimeout           请求处理超时时间（方法执行）

全局配置，Consumer的YAML中default设置
```
spring:
	cloud:
		openfeign:  
		  client:  
		    config:  
		      default:  
		        #连接超时时间  
		        connect-timeout: 3000  
		        #读取超时时间  
		        read-timeout: 3000
```

指定配置，Consumer去调用不同的Provider时，设置不同的时间。
```
spring:
	cloud:
		openfeign:  
		  client:  
		    cloud-payment-service:    # 指定Pay模块的等待超时时间  
		      default:  
		        #连接超时时间  
		        connect-timeout: 3000  
		        #读取超时时间  
		        read-timeout: 3000
```

#### 重试机制
重试机制是指Consumer通过FeignApi调用Provider时，超时时，从FeignApi再试几次！
默认重试机制是关闭的，所以不能在YAML中写了

新建一个配置类FeignConfig
```
@Configuration
public class FeignConfig {
	@Bean
	public Retryer myRetryer() {
		return Retryer.NEVER_RETYR;      //默认是不重试策略
		return new Retryer.Default(period：100, maxPeriod:1, maxAttempts：3);
		// period   初始间隔时间为100ms，100ms后启动重试机制
		// maxPeriod 重试之间间隔时间1s
		// 最多3次，1（default）+2次
	}

}
```

#### 性能优化 HttpClient5
默认使用JDK自带的HttpURLConnection发送HTTP请求。
用==Apache HttpClient5== 替换 OpenFeign默认的HttpURLConnection

Step 1：Consumer侧的pom中引入两个依赖
```
<!--        httpclient5-->  
        <dependency>  
            <groupId>org.apache.httpcomponents.client5</groupId>  
            <artifactId>httpclient5</artifactId>  
            <version>5.3</version>  
        </dependency>  
  
<!--        feign-hc5-->  
        <dependency>  
            <groupId>io.github.openfeign</groupId>  
            <artifactId>feign-hc5</artifactId>  
            <version>13.1</version>  
        </dependency>
```

Step 2：application.yaml中开启
```
spring.cloud.openfeign.httpclient.hc5.enable: true
```

#### 请求/响应压缩
GZIP压缩格式，节省请求、响应的带宽。
```
# 开启
spring.cloud.openfeign.compression.request.enable=true

# 触发压缩的大小
spring.cloud.openfeign.compression.request.min-requesr-size:2049

# 触发压缩的格式类型；
spring.cloud.openfeign.compression.request.mime-types:text/xml,application/xml,application/json

spring.cloud.openfeign.compression.response.enable=true

```
#### 日志打印
查BUG，看调用链路
对Feign接口的调用情况进行监控

日志级别，Enum枚举类

| 日志级别    | 内容                       |
| ------- | ------------------------ |
| NONE    | 默认的，不显示任何日志              |
| BASIC   | 仅记录请求方法、URL、响应状态码和执行时间   |
| HEADERS | 除了BASIC，还有请求和响应头信息       |
| FULL    | 除了HEADERS，还有请求和响应的正文及元数据 |
Step 1：在Feign配置类中，设置日志级别为FULL
```
@Configuration  
public class FeignConfig {        
    @Bean  
    Logger.Level feignLoggerLevel() {  
        return Logger.Level.FULL;  
    }  
}
```

Step 2：application.yaml 中开启Feign日志客户端
```
logging.level.[接口的包名+Feign接口名]：debug

# 注意，顶格，不在任何下面！

# feign日志以什么级别监控哪个接口  logging:  
  level:  
    com:  
      atguigu:  
        cloud:  
          api:  
            PayFeignApi: debug
```

假如Provider挂了，你一直重试也不是办法，直接服务降级，返回“系统繁忙”

## 服务熔断：Resilience4j
服务熔断和降级，断路器：Circuit Breaker
[Resilience4j中文手册](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/index.md)

分布式微服务系统中，相互调用不可避免会超时、异常。当发生故障时，向调用方返回一个符合预期的、可处理的备选响应（FallBack），能保证出现问题时，不会导致整体服务的失败，提高系统容错率。

先有熔断，后有降级。

SpringCloud Circuit Break 是一个接口，提供两个实现类：
	Resilience4j  (主要)
	Spring Retry

Circuit Breaker的三个状态：CLOSED、OPEN、HALF-OPEN
两个特殊状态：DISABLED(禁用)、FORCED_OPEN（强制开启）
当一个微服务模块出现故障时，Circuit Break会迅速切换到OPEN状态（保险丝跳闸），阻止请求继续访问。过段时间，放几个Request过去探探路（HALF-OPEN）,如果恢复了，就CLOSED（保险丝合闸，通电啦）

### 熔断+降级 CircuitBreaker
类比：保险丝
熔断逻辑，配置在Provider一侧
断路器，安装在Consumer一侧

阳哥建议：用次数！

#### 参数含义
#TODO 补充参数的含义：图

#### COUNT_BASED
Step 1：在Provider的Controller层和通用模块的FeignApi上写熔断逻辑
Pay模块（Provider一侧）的Controller层
```
@GetMapping("pay/circuit/{id}")  
public String myCircuit(@PathVariable("id") Integer id){  
    // 直接报错  
    if(id == -4)  
        throw new RuntimeException("-----------circuit id 不能为负数");  
  
    // 超时  
    if(id == 9999){  
        try {  
            TimeUnit.SECONDS.sleep(5);  
        }catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
    }//一切正常  
        return "Hello,circuit inputId: "+"\t"+ IdUtil.simpleUUID();  
}
```
通用模块的FeignApi上
```
@GetMapping("/pay/circuit/{id}")  
public String myCircuit(@PathVariable("id") Integer id);
```

Step 2：Consumer一侧**①引入依赖**、**②配置断路器触发条件**（~~建Module~~、改POM、写YAML、~~主启动~~、业务类）
POM引入依赖
```
<!--        resilience4j-circuitbreaker-->  
        <dependency>  
            <groupId>org.springframework.cloud</groupId>  
            <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>  
        </dependency>  
  
<!--        由于断路器保护需要AOP实现，必须导入AOP包-->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-aop</artifactId>  
        </dependency>
```
写YAML（复杂，注意别写成）
```
# resilience4j配置  
resilience4j:  
  circuitbreaker:  
    configs:                    # 这里有个s,configs  
      default:  
        failureRateThreshold: 50  
        slidingWindowType: COUNT_BASED    # 基于次数
        slidingWindowSize: 6  
        minimumNumberOfCalls: 6  
        automaticTransitionFromOpenToHalfOpenEnabled: true  
        waitDurationInOpenState: 5s  
        permittedNumberOfCallsInHalfOpenState: 2  
        recordExceptions:  
          - java.lang.Exception  
    instances:  
      cloud-payment-service:  
        baseConfig: default


源代码中的默认配置，详见：io.github.resilience4j.circuitbreaker
```

Step 3：Consumer一侧的Controller中安装断路器，并写FallBack方法，注解方式！
```
@RestController  
public class OrderCircuitController {  
    @Autowired  
    private PayFeignApi payFeignApi;  
  
    @GetMapping("/feign/pay/circuit/{id}")  
    @CircuitBreaker(name = "cloud-payment-service", fallbackMethod = "myCircuitFallback")  
    public String myCircuitBreaker(@PathVariable("id") Integer id){  
        String s = payFeignApi.myCircuit(id);  
        return s;  
    }  
    // 服务降级的兜底方案  
    public String myCircuitFallback(Integer id,Throwable e) {  
        return "myCircuitFallback, 系统繁忙，请稍后再试……";  
    }  
}
```

#### TIME_BASED
基于时间的熔断（慢响应）
```
# resilience4j配置 - TIME_BASEDresilience4j:  
  timelimiter:  
    configs:  
      default:  
        timeout-duration: 10s  # 神坑！默认1s就认为是超时异常，我们改大，为了测 TIME_BASED 熔断  
  circuitbreaker:  
    configs:  
      default:  
        failureRateThreshold: 50  
        slowCallDurationThreshold: 2s  
        slowCallRateThreshold: 30  
        slidingWindowType: TIME_BASED  
        slidingWindowSize: 2  
        minimumNumberOfCalls: 2  
        permittedNumberOfCallsInHalfOpenState: 2  
        waitDurationInOpenState: 5s  
        recordExceptions:  
          - java.lang.Exception  
    instances:  
      cloud-payment-service:  
        baseConfig: default
```

### 隔离 BulkHead
舱壁隔离，一个船舱内部分割了N个隔板，有一个地方漏水，只会影响一小个格子，整体不受影响，不要连坐！
依赖隔离&负载保护：用来限制对==下游服务==调用的最大并发数量的限制。

Resilience4j提供了两种隔离的实现方式，限制并发执行数量（底层是[[JUC]]）：
- SemaphoreBulkhead            使用信号量，JUC下的信号灯  
- FixedThredPoolBulkhead     使用有界队列和固定大小线程池

#### SemaphoreBulkhead
底层使用： [[JUC#Semaphore 信号量]]
信号量，类比停车场停车位上面的绿灯。

step 1：在Provider一侧的Controller层和通用模块的FeignApi上写“舱壁隔离”逻辑

Step 2：Consumer一侧**①引入依赖**、**②配置舱壁触发条件**（~~建Module~~、改POM、写YAML、~~主启动~~、业务类）
```
<!--        resilience-4j  - 舱壁隔离 bulkhead-->        
<dependency>  
	<groupId>io.github.resilience4j</groupId>  
	<artifactId>resilience4j-bulkhead</artifactId>  
</dependency>
```
application.yaml
```
resilience4j:  
  timelimiter:  
    configs:  
      default:  
        timeout-duration: 20s  
  bulkhead:  
    configs:  
      default:  
        maxConcurrentCalls: 2  
        maxWaitDuration: 1s
```

Step 3：Consumer一侧的Controller中安装舱壁隔离，并写FallBack方法，注解方式！
```
//舱壁隔离  
@GetMapping("/feign/pay/bulkhead/{id}")  
@Bulkhead(name="cloud-payment-service", fallbackMethod = "myBulkhead", type = Bulkhead.Type.SEMAPHORE)  
public String myBulkhead(@PathVariable("id") Integer id){  
    String s = payFeignApi.myBulkhead(id);  
    return s;  
}  
// 舱壁隔离的兜底方案  
public String myBulkhead(Integer id,Throwable e) {  
    return "myBulkHead, 隔板超出数量限制，系统繁忙，请稍后再试……";  
}
```

思考：信号量和线程池有什么区别？
答：信号量是线程的辅助类，利用颁发通行证的方式，控制线程；线程池是建一个固定数量的线程池，备用，优化线程创建的时间。

##### 参数含义
- maxConcurrentCalls           隔离允许线程最大并发量，默认25
- maxWaitDuration               当停车场停满时，新的线程阻塞在入口，最长等待时间，默认0

达到最大等待时间，就走fallback兜底方法

#### FixedThredPoolBulkhead
固定线程池舱壁隔离
底层使用：[[JUC#newFixedThreadPool(int)]]
注意：ThreadPoolBulkhead只对CompletableFuture方法有效，所以必须创建返回CompletableFuture类型的方法！
##### 参数含义
- maxThreadPoolSize             线程池最大大小
- coreThreadPoolSize             核心线程池最大大小
- queueCapacity                     等待队列的容量
- keepAliveDuration               当线程数大于核心时，多余空闲线程在回收前等待新任务的最长时间
联想到[[JUC#ThreadPoolExecutor的7个参数]]

applicaiton.yaml的配置
```
# resilience4j bulkhead FixedThreadPoolBulkhead 配置  
resilience4j:  
  timelimiter:  
    configs:  
      default:  
        timeout-duration: 10s  
  thread-pool-bulkhead:  
    configs:  
      default:  
        core-thread-pool-size: 1  
        max-thread-pool-size: 1  
        queue-capacity: 1  
    instances:  
      cloud-payment-service:  
        baseConfig: default


# 当测试FixedThreadPoolBulkhead时，注释掉！
#        group:  
#          enabled: true          # 开启分组，精细。
```

### 限流 RateLimit
对并发访问进行==限速==！

| 限流算法   | 描述                                      | 缺点                     |
| ------ | --------------------------------------- | ---------------------- |
| 漏斗算法   | 固定容量输出，溢出部分丢弃                           | 出水口固定，缺乏效率             |
| √令牌桶算法 | 发放令牌，拿到令牌的执行<br>联想到：redis限流器，对每个请求分配令牌！ |                        |
| 滚动时间窗  | 按固定时间间隔砍断，单位时间内<br>请求数必须小于峰值，否则丢弃       | 时间临界点存在BUG；double kill |
| √滑动时间窗 | 把固定时间片进行划分，并且随<br>时间移动                  | 避免临界点问题。               |

#### 参数含义
- timeoutDuration              线程等待权限时间，5秒
- limitRefreshPeriod           滑动时间窗口为500纳秒
- limitForPeriod                  在一次刷新周期内，允许最大请求数。默认50个

 
 Step 1：在Provider的Controller层和通用模块的FeignApi上写“限流”逻辑
```
 //    Resilience4j rateLimit 实例  
    @GetMapping("/pay/ratelimit/{id}")  
    public String myRateLimit(@PathVariable("id") Integer id){  
        return "Hello, myRateLimit 欢迎到来 inputId"+ id + "\t" + IdUtil.simpleUUID();  
    }
```

```
@GetMapping("/pay/myratelimit/{id}")  
public String myRateLimit(@PathVariable("id") Integer id);  
```

Step 2：Consumer一侧**①引入依赖**、**②配置限流触发条件**（~~建Module~~、改POM、写YAML、~~主启动~~、业务类）
POM
```
<!--        resilience-4j  -  限流  rateLimit-->        <dependency>  
            <groupId>io.github.resilience4j</groupId>  
            <artifactId>resilience4j-ratelimiter</artifactId>  
        </dependency>
```

YAML
```
# resilience4j ratelimit配置  
resilience4j:  
  ratelimiter:  
    configs:  
      default:  
        limitForPeriod: 2  
        limitRefreshPeriod: 1s  
        timeoutDuration: 1  
	  instances:                       # 注意是复数！instances
	    cloud-payment-service:  
	      baseConfig: default

含义：1s为单位的滑动窗口，1s以内允许最多2个请求，等待时间1s
```

Step 3：Consumer一侧的Controller中安装RateLimiter，并写FallBack方法，注解方式！
```
//限流 RateLimiter@GetMapping("/feign/pay/ratelimit/{id}")  
@RateLimiter(name="cloud-payment-service", fallbackMethod = "myRateLimitFallback")  
public String myBulkhead(@PathVariable("id") Integer id){  
    return payFeignApi.myRateLimit(id);  
}  
  
public String myRateLimitFallback(Integer id, Throwable e){  
    return "你被限流了，禁止访问~~~";  
}
```

## 链路追踪：Micrometer
Sleth，只支持SpringBoot 2，被替代了
微服务调用的链路追踪，图形化展示，可以显示微服务之间调用链，也能看到每个微服务之间处理的时间，用于优化维护！
[Micrometer](https://micrometer.io/docs/tracing)

Trace Id      一条链路通过Trace Id唯一标识
Span Id      微服务一发起请求，它自己就有了一个Span Id
parent Id    各span通过parent Id关联

### Zipkin安装运行
[Zipkin](https://zipkin.io/)  是图形化工具
下载地址，官网-Quickstart-Java-Lastest release   下载一个jar包
在存放目录打开cmd，运行：`java -jar zipkin-server-3.1.1-exec.jar`
运行在：`https://localhost:9411`

### Micrometer和Zipkin整合

| XX                               | XX                                       |
| -------------------------------- | ---------------------------------------- |
| micrometer-tracing-bom           | 导入链路追踪版本中心，体系化说明，接口规范<br>只保留在父工程，不用引入子工程 |
| micrometer-tracing               | 指标追踪，产生Span Id、Trace Id                  |
| micrometer-tracing-bridge-brave  | 收集数据                                     |
| micrometer-observation           | 观测模块、度量数据                                |
| feign-micrometer                 | 一个Feign Http客户端的Micrometer模块             |
| zipkin-reporter-brave            | 将Brave跟踪数据提交到Zipkin系统                    |
| 补充包：spring-boot-starter-actuator | SpringBoot的监视和管理模块                       |

Step 1：Cloud-2024 父工程的POM中记录`<properties>`和`<dependencyManagement>`

Step 2：Consumer一侧的POM中加入所有依赖：
`micrometer-tracing-bom`只保留在父工程，不用引入子工程
```
<!--        以下是micrometer整合zipkin的依赖-->  
        <!--micrometer-tracing指标追踪  1-->        <dependency>  
            <groupId>io.micrometer</groupId>  
            <artifactId>micrometer-tracing</artifactId>  
        </dependency>  
        <!--micrometer-tracing-bridge-brave适配zipkin的桥接包 2-->        <dependency>  
            <groupId>io.micrometer</groupId>  
            <artifactId>micrometer-tracing-bridge-brave</artifactId>  
        </dependency>  
        <!--micrometer-observation 3-->  
        <dependency>  
            <groupId>io.micrometer</groupId>  
            <artifactId>micrometer-observation</artifactId>  
        </dependency>  
        <!--feign-micrometer 4-->  
        <dependency>  
            <groupId>io.github.openfeign</groupId>  
            <artifactId>feign-micrometer</artifactId>  
        </dependency>  
        <!--zipkin-reporter-brave 5-->  
        <dependency>  
            <groupId>io.zipkin.reporter2</groupId>  
            <artifactId>zipkin-reporter-brave</artifactId>  
        </dependency>
```

YAML
```
# ========================zipkin===================  
management:  
  zipkin:  
    tracing:  
      endpoint: http://localhost:9411/api/v2/spans  
  tracing:  
    sampling:  
      probability: 1.0 #采样率默认为0.1(0.1就是10次只能有一次被记录下来)，值越大收集越及时。
```

Step 3：业务类更新
```
@RestController  
public class OrderMicrometerController {  
    @Autowired  
    private PayFeignApi payFeignApi;  
  
    @GetMapping("/feign/micrometer/{id}")  
    public String myMicrometer(@PathVariable("id") Integer id) {  
        return payFeignApi.myMicrometer(id);  
    }  
}
```

## 网关：Gateway
总结：网关，在满足某种断言，就路由到指定uri，在放行前后利用过滤给Request和Response加减点内容。

底层用的是Netty（不是tomcat），
联想到：原生servlet包中的Filter过滤器[[JavaWeb#Filter 过滤器]]，但不是哦！Spring Cloud Gateway没有使用传统[[JavaWeb#servlet]] ，而是使用Spring WebFlux响应式编程

官方文档，一定要看稳定版，不要看SANPSHOT快照版，不稳定！
[Spring Cloud Gateway 4.1.4 Reference官方文档](https://docs.spring.io/spring-cloud-gateway/reference/)

提供三大核心功能：
- Route：路由转发，网关基本模块，由一系列断言和过滤器组成，如果断言为true则匹配该路由。
- Predicate：断言判断（布尔值），依据请求头、请求参数来依据条件进行判断。
[[Java_03_高级特性#判断型 Predicate Interface]]
- Filter：执行过滤链，对前两者功能的增强，在路由前/后进行增强。
	联想到[[Spring FrameWork#AOP 面向切面编程]]中的前置增强、返回增强！![[API开放平台#^b83bfd]]

我们不会对外直接暴露8001端口，外面要套一层9527网关，外面的人想要使用8001的支付模块，你告诉我Gateway，我来给你安排！

GateWay默认端口：9527

### 常见应用场景 
鱼皮补充，网关常见应用场景

| XX           | 备注说明                                                        |                                    |
| ------------ | ----------------------------------------------------------- | ---------------------------------- |
| 路由           | aa                                                          |                                    |
| 负载均衡         | bb                                                          |                                    |
| 统一鉴权         | cc                                                          |                                    |
| 统一处理跨域       | [[#配置跨域]]                                                   |                                    |
| 统一业务处理       | 把每个项目通用逻辑，抽象到上层（网关）统一处理，<br>例如接口调用次数统计                      | [[#自定义全局过滤器 - CustomGlobalFilter]] |
| 访问控制         | 黑白名单，比如限制DDos IP                                            |                                    |
| 发布控制         | 灰度发布，上线一个新功能，先给新接口分配20%的流量，<br>老接口80%，再慢慢调整比例               | [[#Weight Route Predicate]]        |
| 流量染色         | 在请求头中加一个标识，确保请求都是从网关来的，<br>不是直接访问微服务模块                      | [[#AddRequestHeader]]              |
| 接口保护         | 隐藏具体微服务的地址，统一走网关<br>信息脱敏，将响应头中真实接口的IP等信息抹去<br>降级、熔断、限流、超时时间 | [[#CircuitBreaker]]<br>            |
| 统一日志<br>统一文档 | Swagger接口文档                                                 |                                    |

有两种网关：
- 接入层网关（全局网关），作用是负载均衡，请求日志，不与业务逻辑绑定，常见的有[[Nginx]]、Kong等 ^3fb2ed
- 微服务网关（业务网关），作用是将请求转发到不同业务/接口/服务，会有业务逻辑。例如Spring Cloud Gateway

### 基础
Step 1：建Module、改POM、写YAML、主启动、~~业务类~~
#### POM：GateWay的Maven依赖
```
<!--gateway-->  
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-gateway</artifactId>  
</dependency>  
<!--服务注册发现consul discovery,网关也要注册进服务注册中心统一管控-->  
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-consul-discovery</artifactId>  
</dependency>  
<!-- 指标监控健康检查的actuator,网关是响应式编程删除掉spring-boot-starter-web dependency-->  
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-actuator</artifactId>  
</dependency>
```

#### YAML：GateWay的配置
##### 路由和断言
1、路由和断言
```
    gateway:  
      routes:  
        - id: pay_routh1 #pay_routh1                
        - #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名  
          uri: http://localhost:8001                
          #匹配后提供服务的路由地址  
          predicates:  
            - Path=/pay/gateway/get/**              
            - # 断言，路径相匹配的进行路由  
        - id: pay_routh2 #pay_routh2                
        - #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名  
          uri: http://localhost:8001                
          #匹配后提供服务的路由地址  
          predicates:  
            - Path=/pay/gateway/info/**              
            - # 断言，路径相匹配的进行路由
```
告诉我Gateway，拦在哪几个端口前面，所有访问这几个端口的请求，都得先过我这关！
Gateway==位于Provider一侧==，拦在Provider前面，避免Provider的端口被公开，隐藏！

##### 配置跨域
[Spring Cloud GateWay配置跨域参考文档]([CORS Configuration](https://docs.spring.io/spring-cloud-gateway/reference/spring-cloud-gateway/cors-configuration.html))
```
spring:
  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOrigins: "https://docs.spring.io"
            allowedMethods:
            - GET

# 对于所有 GET 请求路径，允许来自 `docs.spring.io` 的请求发出 CORS 请求。
```

#### 主启动
```
@SpringBootApplication  
@EnableDiscoveryClient  
public class Main9527 {  
    public static void main(String[] args) {  
        SpringApplication.run(Main9527.class,args);  
        System.out.println("SpringBoot Main 9527 启动！");  
    }  
}
```

Step 2：在Feign接口上，将@FeignClient定向到Gateway，而不是微服务！
```
//@FeignClient("cloud-payment-service")

@FeignClient("cloud-gateway")    // Feign接口不再直接去找pay模块，而是去Gateway大哥那里去！  
public interface PayFeignApi {
```

由于在getaway的yaml中，把URL和端口都写死了！这个肯定不行啊
### 高级
#### Router 路由
根据什么~~条件~~[[#Predicate 断言]]，转发请求到哪里
解决转发微服务时的动态！这样open-feign就能支持负载均衡

##### 配置文件
```
#Gateway相关配置  
    gateway:  
      routes:  
        - id: pay_routh1                      #pay_routh1 
          #uri: http://localhost:8001         # 写死肯定不行啊！  
          uri: lb://cloud-payment-service      # 利用负载均衡-微服务名  
          predicates:  
            - Path=/pay/gateway/get/**         # 断言，路径相匹配的进行路由
		  filters:                             # 过滤
		    - AddRequestHeader=XXX
```

一类的handler方法，可以合并到一个路由中，例如：

| gateway.routes.id | geteway.routes.predicates   | XX                |
| ----------------- | --------------------------- | ----------------- |
| - id: pay_routh1  | - Path=/pay/gateway/get/**  | 走get方法的一批handler  |
| - id: pay_routh2  | - Path=/pay/gateway/info/** | 走info方法的一批handler |
|                   |                             |                   |

##### 编程实现
推荐使用配置方式，有语法提示，不容易出错
```

```

#### Predicate 断言
一组规则、条件，确定如何转发[[#Router 路由]]
==注意：yaml配置文件极易写错，一定要去官网拷贝示例代码，在此基础上修改！==
##### 内置 predicate
ref：[官方文档-断言章节](https://docs.spring.io/spring-cloud-gateway/docs/3.1.9/reference/html/#gateway-request-predicates-factories)
###### After Route Predicate
在这个值的时间之后才能访问，才能放行！
适用场景：限时抢购！
获取ZonedDataTime
```
ZonedDateTime zbj = ZonedDateTime.now();
sout(zbj)

>> 输出：2024-03-22T15:46:53.972085100+08:00[Asia/Shanghai]
```

```
predicates:  
  - After=2024-03-22T15:46:53.972085100+08:00[Asia/Shanghai] # [ZonedDataTime]
```

###### Before Route Predicate
在这个值的时间之前才能访问

###### Between Route Predicate
在时间范围内才能访问

###### Cookie Route Predicate
1个是cookie name，一个是正则表达式
访问过程中必须带着cookie，否则不给访问！
```
predicates:  
  - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由  
  - Cookie=username,rockyshen
```

如何发起带cookie的请求呢？
方法1：命令行：`curl http://localhost:9527/pay/gateway/get/1 --cookie "username=rockyshen"`
方法2：PostMan
方法3：Chrome开发模式

###### Header Route Predicate
请求头中的属性名称与正则表达式匹配，则放行；否则拦在gateway门外。
```
predicates:  
  - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由  
  - Header=X-Request-Id, \d+

请求头中要有X-Request-Id这个属性，并且它的值为整数！
```

###### Host Route Predicate
接收一组域名列表

###### Path Route Predicate
允许的路径
```
predicates:  
  - Path=/pay/gateway/get/** 
```

###### Query Route Predicate
要求用户必须以Param指定参数名访问，例如`localhost/pay/get?username=1`，没有参数不允许访问

###### RemoteAddr route predicate
IP地址绑定，192.168.93.1/24   表示前三位固定，最后一位随意

###### Method route predicate
指定请求方法。

###### Weight Route Predicate
权重比例进行路由，适用于灰度发布！
```
spring:
	cloud:
		gateway:
			routes:
				-id: weight_high
				 uri: https://weighthigh.org
				 predicates:
				 - Weight=group1,8          # 80%的请求定向到这个接口
				-id: weight_low
				 uri: https://weightlow.org
				 predocates:
				 - Weight=gtoup1,2          # 20%的请求定向到这个接口
```



##### 自定义 predicate
可以打开源码，参考`AfterRoutePredicateFactory`来写

Step 1：在Gateway模块中，新建一个类名：XxxRoutePredicateFactory 并继承AbstractRoutePredicateFactory

Full Expanded Config
```
predicates:  
  - name: My
    args:
      userType: diamond
```

需要重写shortcutFiledOrder方法，才能用`- My: diamond` 短促的配置


编程方式实现，可以使用lambda表达式！
[[Java_03_高级特性#判断型 Predicate Interface]]
![[Java_03_高级特性#^f2bb59]]
Route Predicate Factories  是接口，在路由前/后进行条件判断，布尔匹配，有10个实现类

#### Filter 过滤器
对请求进行一系列处理，比如添加请求头、添加请求参数；
就是[[SpringMVC#HandlerInterceptor 拦截器]]，底层用的是AOP

全局默认过滤器，GlobalFilter 是接口，定义规范
单一内置过滤器，主要作用于单一路由

请求权鉴、异常处理、接口调用时常统计

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/spring_cloud_gateway_diagram.png)
##### 内置过滤器
ref：[官方文档：过滤器章节](https://docs.spring.io/spring-cloud-gateway/docs/3.1.9/reference/html/#gatewayfilter-factories)

| 常用内置过滤器                   | XX             |     |
| ------------------------- | -------------- | --- |
| 请求头（RequestHeader）相关组     |                |     |
| 请求参数（RequestParameter）相关组 | bb             |     |
| 响应头（ResponseHeader）相关组    | cc             |     |
| 前缀和路径相关组                  | 对访问路径进行重新编排，隐藏 |     |
| 其他                        |                |     |
```
spring:
	cloud:
		gateway:
			routes:
				filters:
					- AddRequestHeader=X-Request-atguigu1,atguiguValue1  # key val
					- 
```

###### AddRequestHeader
给请求头上打上标识。可以用于请求染色

###### AddRequestParameter
给请求加参数
###### PrefixPath Gateway Filter
自动添加路径前缀
```
predicates:
  - Path=/gateway/filter/**
filters:
  - PrefixPath=/pay      # url = prefix + path
```

###### SetPath Gateway Filter   
修改地址
```
predicates:
  - Path=/XYZ/abc/{segment}           # 浏览器写的是这个
filters:
  - SetPath=/pay/gateway/{segment}    # 实际调用的是这个
```

###### RedirectTo Gateway Filter
重定向
```
filters:
  - Redirect=302, http://www.baidu.com    # 如果报了302错误，就重定向到百度
```

###### DedupeResponseHeader
响应头中去重

###### CircuitBreaker
断路器
[[#服务熔断：Resilience4j]]
需要引用一个依赖
```
<dependency>
	<grouId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
</dependency>
```

###### RequestRateLimiter Gateway Filter
限流！也可以用[[#服务熔断：Resilience4j]]  --> [[#限流 RateLimit]]

官方推荐用[[redis]]实现限流！使用令牌桶算法

1、引入redis限流相关依赖
```
spring-boot-starter-data-redis-reactive
```

redis-rate-limit

###### Retry Gateway Filter
按指数进行重试，2秒重试一次，4秒重试一次，8秒，16秒……

Default Filter
作用到全局路由，阳哥不推荐
```
spring:
  cloud:
    gateway:
      default-filters:
       - AddRequest
       - Prefix
```
##### 全局过滤器
GlobalFilter --> customFilter
全局过滤器，==**作用于所有路由**==，无需配置，实现统一化的业务功能，当请求到来时，GlobalFilter和自身的Filter组成一条过滤链。

通过实现全局过滤器GlobalFilter接口及其中的方法，自定义过滤器

这两个接口，并重写里面约定的方法，即可实现自定义全局过滤器！

###### GlobalFilter 全局过滤器
`Mono<Void>` filter() --> @Override GlobalFilter
Mono类，reactive.core响应式编程中的概念，表示包含零个或一个元素的响应式流。它是 Mono（单个值）的缩写。
Flux类，表示包含零个或多个元素的响应式流

统一的权限校验，示例代码：
[myapi-gateway-customFilter](https://github.com/rockyshen/myapi-gateway/blob/main/src/main/java/com/yupi/myapigateway/CustomGlobalFilter.java)

![[Spring FrameWork#^68dd31]]
```
@Slf4j  
@Component  
public class CustomGlobalFilter implements GlobalFilter, Ordered {

	@Override  
	public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {}

	@Override  
	public int getOrder() {  
	    return -1;  
	}
}
```

###### getOrder() --> @Override Ordered
![[Spring FrameWork#^2f2f58]]
```
@Bean
public GlobalFilter customFilter() {
    return new CustomGlobalFilter();
}

public class CustomGlobalFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("custom global filter");
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```

https://blog.csdn.net/qq_19636353/article/details/126759522

[[SpringMVC#总结各种过滤/拦截方案]]

## 服务注册：Nacos
我在mac/应用程序/nacos安装了两个版本：1.1.3（停止维护）和2.1.1

2024.6.10 妈的，2.X版本在Mac上是大坑啊，各种启动不起来！
### 安装使用
启动nacos服务器
1、使用命令行模式，进入nacos/bin目录
2、`sh startup.sh -m standalone`  使用单机模式启动
3、访问：`localhost:8848/nacos`

默认账号密码都是：nacos

用完记得shutdown关闭，一直跑在后台浪费资源吗！
```
# 在nacos/bin目录下

sh shutdown.sh
```

在Spring Boot中开启Nacos的注册发现
1、添加依赖
```
<!-- 服务注册 -->  
<dependency>  
    <groupId>com.alibaba.cloud</groupId>  
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>  
</dependency>
```
2、配置文件中，写明nacos运行的地址
```
# 微服务名称  
spring.application.name=service-vod  
  
# nacos  
spring.cloud.nacos.discovery-addr:127.0.0.1:8848
```
3、主启动上 @EnableDiscoveryClient

### 命名空间
配置隔离，相当于分组！！
默认是public

```
spring.cloud.nacos.config.namespace="命名空间的UUID"
```

- 开发：dev    测试：test    生产：prod
- 每一个微服务都创建自己的命名空间！  coupon、member

### 配置集ID
一组配置项的集合。类似于配置文件名（Application.yml），就是Nacos网页端的：Data ID
gulimall-coupon.properties
```
spring.cloud.nacos.config.group=1111    # 使用双十一对应配置分组
```

### 配置分组
默认所有的配置集，都属于：DEFAULT_GROUP

总结：微服务任何配置信息，都加到Nacos配置中心
只需要在bootstrap.properties说明加载配置中心的那些内容。

bootstrap.properties
```
spring.application.name=gulimall-coupon  
  
spring.cloud.nacos.config.server-addr=127.0.0.1:8848  
#nacos中coupon命名空间中的配置  
spring.cloud.nacos.config.namespace=f22d4903-2b6f-41cc-95b4-4fe8c62e9453  
spring.cloud.nacos.config.group=dev  
  
spring.cloud.nacos.config.extension-configs[0].data-id=datasource.yml  
spring.cloud.nacos.config.extension-configs[0].group=dev  
spring.cloud.nacos.config.extension-configs[0].refresh=true  
  
spring.cloud.nacos.config.extension-configs[1].data-id=mybatis.yml  
spring.cloud.nacos.config.extension-configs[1].group=dev  
spring.cloud.nacos.config.extension-configs[1].refresh=true  
  
spring.cloud.nacos.config.extension-configs[2].data-id=other.yml  
spring.cloud.nacos.config.extension-configs[2].group=dev  
spring.cloud.nacos.config.extension-configs[2].refresh=true
```

^249dc0

## 服务熔断：Sentinel


## 分布式事务：Seata



## RPC
远程过程调用，Remote Procedu Call，区别于Http方式！底层是Triple协议（免去请求头等信息）
像调用本地方法一样，调用远程接口

### 服务调用：Dubbo
Dubbo是Java中主流的RPC框架。

[Dubbo for Java开发手册](https://cn.dubbo.apache.org/zh-cn/overview/mannual/java-sdk/quick-start/brief/)
[Dubbo示例代码](https://github.com/apache/dubbo-samples)

| Dubbo | @EnableDubbo    | 主启动上，开启dubbo                                                               |
| ----- | --------------- | -------------------------------------------------------------------------- |
|       | @DubboService   | 在服务提供者的“接口实现类”中，注解将此类加入注册中心。<br>类似于@FeignClient，但是FeignClient是加到接口上，这是重要区别 |
|       | @DubboReference | 在需要使用的类中，从注册中心中获取bean，类似于@AutoWired 和 @Resource                            |
[[API开放平台#Dubbo进行RPC调用]]
#### 通用步骤
步骤1：从git clone整个项目实例代码，用IDEA打开，以Spring Boot Application的方式启动（注意，如果端口占用，就换一个端口）
	dubbo中内嵌了一个zookeeper（EmbeddedZooKeeper类，这个文件就是整个的ZooKeeper），类似SpringBoot内嵌tomcat是一样的。

启动顺序：注册中心zookeeper、提供方provider、消费方consumer

不要整合zookeeper，鱼皮都翻车，用nacos
```
<!--            dubbo相关，核心库-->  
<dependency>  
    <groupId>org.apache.dubbo</groupId>  
    <artifactId>dubbo</artifactId>  
    <version>3.1.3</version>  
</dependency>  
  
<!--            dubbo整合注册中心，nacos-->  
<dependency>  
    <groupId>com.alibaba.nacos</groupId>  
    <artifactId>nacos-client</artifactId>  
    <version>2.1.0</version>  
</dependency>
```

#### 整合Nacos
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/IMG_2046.PNG)



### 服务注册：Zookeeper
(整合Dubbo，RPC调用时，存储Provider的方法的存储中心，也叫注册中心。


