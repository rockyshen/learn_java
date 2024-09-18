![[Django#^7656e9]]
Spring MVC是Spring FrameWork下的一个Web子框架，作用在表述层！
在Java的底层实现中，能接收来自Web的接口是：Filter和Servlet
- SpringMVC是基于[[JavaWeb#servlet]]API的一个应用层封装！
- 依赖名称：Maven Search：spring-webmvc
- 另一款表述层框架：Strus是基于Filter的封装！
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/713def6bb7e985fcf21e19120aa94186.JPG) ^ea8758

## SpringMVC 基本概念
1、简化前端数据的接收（handler方法的形参列表上接收数据） ^a4f504

2、简化后端数据的响应（返回值） ^0040ff

SpringMVC将其他淘汰，因为它有个好爸爸！Spring框架，例如写完配置类，不用手动写ioc配置文件启动容器，通过SpringMVC内置的方法就能起来！

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/5f5857e0858e3fe4a52ccf915d65c9dd.PNG)


DispatcherServlet
继承自Servlet的子类！DispatcherServlet是Spring MVC的CEO，负责处理来自客户端的全部请求，再分发给秘书（handlerMapping）、经理(handlerAdapter)

[[设计模式#Adapter 适配器模式]]
![[设计模式#^32dbcf]]

DispatcherServlet和HttpServlet有什么区别？
- HttpServlet 属于 Java Servlet API 的一部分
- DispatcherServlet 是 Spring Framework 的一部分

### 基本步骤
1、声明一个handler方法，Controller层
	@Controller    类上加注解，加入ioc容器
	@RequestMapping("springmvc/hello")    方法上加注解，到秘书中注册，类似[[JavaWeb#`<servlet-mapping>`]]
	@ResponseBody     忽略视图解析器
2、使用配置类，将组件放入ioc容器    ->  MvcConfig
	类上加注解，声明配置类，==@Configuration==
	自定义的组件（handler）   ==@ComponentScan(“类全限定符”)==
	配置类内部的方法上，return new 第三方依赖实例     ==@Bean==
	自动把秘书handlerMaping + 经理handlerAdaptor 加到ioc容器中   ==@EnableWebMvc==   
	web容器，需要==implements WebMvcConfiturer==接口（添加拦截器、静态资源处理等需求才需要实现），root容器不需要！
3、SpringMVC环境搭建  -> SpringMvcInit
初始化`public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServlet`

回忆之前：
1、普通工程：之前在Spring FrameWork中创建ioc容器，加载配置类，我们需要在测试环境中实例化`ClassPathXmlApplicationContext` -> ioc容器，并指定配置类
![[Spring FrameWork#^d56778]]

2、Web工程：在web.xml中实例化
```xml
<servlet>
	<servlet-name>dispatcherServlet</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
</servlet>

	<servlet-mapping>
		<servlet-name>dispatcherServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>

注意：DispatcherServlet是spring Framework对Servlet的实现！
```

### 核心组件和调用流程
#### handlerMapping
秘书

#### handlerAdapter
经理
[[设计模式#Adapter 适配器模式]]

`public class SpringMvcInit extends AbstractAnnotationConfigDispatcherServletInitializer {}`
#### AbstractAnnotationConfigDispatcherServletInitializer
其中AbstractAnnotationConfigDispatcherServletInitializer这个类
	继承自WebApplicationInitializer类中
		onStartUp()方法，tomcat一启动就执行的方法！

替代了web.xml方式初始化。直接@override onStartUp方法即可

##### 重写：getServletConfigClasses()
1、在tomcat一启动时，就执行创建ioc容器
2、创建ioc容器，读取配置类

##### 重写：getServletMapping()
映射访问URL

## SpringMVC 接收数据
SpringMVC如何![[#^a4f504]]
### 设置访问路径  --> @RequestMapping("url")
在handler方法加上==@RequestMapping==注解：将请求的URL地址和处理请求的handler方法关联起来，建立映射关系。也就是将此handler方法注册到[[#handlerMapping]]中。

联想到 ：[[JavaWeb#方式2：注解 @WebServlet()]]
1、Servlet的 @WebServlet()，必须带斜杠开头！
2、SpringMVC的 @RequestMapping注解，无所谓，用不用斜杠开头都无所谓！

#### URL规则

| XX                     | XX                   |                     |
| ---------------------- | -------------------- | ------------------- |
| 类上的@RequestMapping     | ("product/category") | **头上不要加斜杠！**        |
| handler方法上的@GetMapping | ("/brand/list")      | **头上必须加斜杠，尾巴不要加斜杠** |

注意URL的准确性，否则方法写对了，访问的时候也会出错，访问不到！这个已经在实战项目中吃过几次教训了！

正确：http://localhost:8080/ssm/schedule
错误：http://localhost:8080/ssm/schedule==/==     <--不要多此一举，加个斜线，postman就找不到资源了！

指定请求方式，属性 method
```
	@RequestMapping("login", method=RequestMethod.POST)
	public String login(){}
```

| XX              | XX               |                                                                                         |
| --------------- | ---------------- | --------------------------------------------------------------------------------------- |
| @RequestMapping | 限定请求方式，用method属性 |                                                                                         |
| @GetMapping     | 只允许GET方式         | @GetMapping("{id}")  动态路径设计，![[#^81140b]]                                               |
| @PostMapping    | 只允许POST方式        | 如果维持类路径，就啥都不写,==！不写( ) ！==，示例 @PostMapping<br>public R delete(@PathVariable Integer id) |
| @PutMapping     | 只允许POST方式        |                                                                                         |
|                 |                  |                                                                                         |
[Spring MVC - Annotated Controllers - Mapping Requests 说明文档](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-requestmapping.html#mvc-ann-requestmapping-params-and-headers)
#### @PostMapping

| 参数                                           | 说明                                    |
| -------------------------------------------- | ------------------------------------- |
| value = "/v1/get_lucky"                      | 该方法的访问路径                              |
| consumes = "application/json; charset=utf-8" | 按内容类型缩小映射范围，只接受JSON格式的POST<br>其他格式不接受 |


注意：@GetMapping  @PostMapping等只能用在handler方法上，不能用在controller类上！

~~在==controller类==上加注解，提取通用访问地址~~
```
@Controller
@RequestMapping("user")
public class UserController{

	@RequestMapping("login")
	public String login(){}

	@RequestMapping("register")
	public String register(){}
}
```

一个经验：在微服务之间调用的时候，尽量还是写在handler方法上，不要在类顶部写@RequestMapping，不要用总的！，区分不同方法写Mapper比较好，不然会很麻烦！
```
@RESTController
public class PayController {  
    @Autowired  
    private PayService payService;  
  
    @PostMapping("pay/add")  
    public Result<String> addPay(@RequestBody Pay pay) {  
        System.out.println(pay.toString());  
        int rows = payService.insert(pay);  
        return Result.success("成功插入记录，影响行数："+ rows);  
    }

	@GetMapping("pay/get/{id}")  
	public Result<Pay> getById(@PathVariable("id") Integer id){  
	    //运行时异常  
	    if (id <= 0) {  
	        throw new RuntimeException("id不能为负数");  
	    }  
	    Pay pay = payService.selectById(id);  
	    return Result.success(pay);  
	}
```


### 参数接收

| 对比   | Param                     | Json                       |
| ---- | ------------------------- | -------------------------- |
| 格式   | key1=value1 & key2=value2 | {key1:value1, key2:value2} |
| 编码   | ASCII                     | UTF-8                      |
| 顺序   | 无顺序                       | 有顺序，有嵌套                    |
| 数据类型 | 只支持String                 | 复杂类型，数组、对象                 |
| 适用场景 | 单一数据                      | 复杂数据                       |
| 请求方式 |                           | POST                       |
实际开发中，大量使用JSON！
#### 参数接值
@RequestParam
/param/data==?name=root&age=18==

1、形参名与参数名一致的话，不用加任何注解，自动接值
	得益于handlerAdapter经理的帮忙，帮我们解析好了。
	1、形参名 与 参数名必须完全相同
	2、可以不传递，不报错！ ^366db2
```
/*
url:/param/data?name=root&age=18  
需求：获取URL中的name和age的值  
 */
 
@RequestMapping("data")
@ResponseBody  
public String data(String name, int age){    // 形参上不用加任何注解！
    System.out.println("name = " + name + ", age = " + age);  
    return "name = " + name + ", age = " + age;  
}
```

2、注解指定@RequestParam
	name    指定请求参数名，和形参名绑定
	value     同上
	required   默认true，必须传餐
	defaultValue    required为false时，设定一个默认值
```
/*注解指定  
url：/param/account=root&page=1  
需求：account必须传递，page可以不用，如果不传递默认就是1  
 */
 
@GetMapping("data1")  
@ResponseBody  
public String data1(
	@RequestParam(value="account") String account, 
	@RequestParam(required = false,defaultValue = "1") int page)
	{  
    System.out.println("account = " + account + ", page = " + page);  
    return "account = " + account + ", page = " + page;  
}
```

感悟：@RequestParam(required = false,defaultValue = "1") int page)，springmvc会自动进行类型转换，将defaultValue的字符串的1，转换类型为int类型的1，赋值给page！这块不用担心！！

3、使用集合接值，必须加@RequestParam，否则数据类型对应不上
```
/*特殊值  
url: /param/data2?hobbys=吃&hobbys=喝&hobbys=学习  
需求:一个key,多个值，用集合接值，必须加@RequestParam注解  
 */

@GetMapping("data2")  
@ResponseBody  
public String data2(@RequestParam List<String> hobbys){  
    System.out.println("hobbys = " + hobbys);  
    return "ok";  
}
```

4、实体类接值
==实体类属性名与参数名必须一致==。就能接到值，不用加任何注解！
```
/*实体对象接值  
url: /param/data3?name=二狗子&age=18  
先准备好pojo  
需求：将param传入的对象属性，自动赋值到实体类的Field上！  
 */
 
@GetMapping("data3")
@ResponseBody  
public String data3(User user){  
    System.out.println("user = " + user);  
    return user.toString();  
}
```
#### 路径接值
/User/root/123456   相当于路径中的值就是我们需要的参数，而不是用?后面

注意：
1、动态路径设计：@RequestMapping("{account}/{passowrd}") ^81140b

2、接收动态路径参数笔记在形参前面加：==@PathVariable==注解，否则接收param参数 

如果路径变量名 不等于 形参名，需要在`@PathVariable("account") String userName`来指定！

```
@Controller  
@RequestMapping("path")  
@ResponseBody  
public class PathController {  
    @RequestMapping("{account}/{password}")  
    public String login(
				    @PathVariable String account, 
					@PathVariable String  password){  
        System.out.println("account = " + account + ", password = " + password);  
        return "account = " + account + ", password = " + password;  
    }  
}
```

#### JSON接值
必须用POST方式！
浏览器不能模拟发起POST方式，请求体中带有JSON数据的请求，所以使用工具:PostMan（Body-Raw-JSON)

**==步骤：==**
1、接收处理json的handler方法加上@PostMapping注解（只支持POST方式），==传递的形参加上@RequestBody注解==！
2、导入json处理的第三方依赖(jackson-databind)，为handlerAdapter配置json转化器（==在配置类上加@EnableWebMvc注解==）！（因为：java原生只支持param和路径接收参数）

2024.9.16 项目的Controller中的方法，接收不到@RequestBody传递的对象，JSON和实体类对应不上。在实体类的属性上加上@JsonProperty(value="xxx")注解(value是JSON中的字段），就好使了，相当于指定以JSON中每个字段和实体类属性的对应关系
	用户中心中，spring-boot-starter-web 2.6.13没有额外操作就可以
	硅谷商场总，spring-boot-starter-web 2.3.2.RELEASE就不行！

@EnableWebMvc  -> 配置类上
组合功能：
1、自动将：秘书handlermapping、经理handlerAdapter 加入ioc容器中
2、给经理添加json处理器
```
@EnableWebMvc  //在handlerAdapter经理上加了Json处理器  
@Configuration  
@ComponentScan({"com.atguigu.param","com.atguigu.path","com.atguigu.json"})  
public class MvcConfig {  

不用再把经理和秘书加到ioc容器了，因为@EnableWebMvc已经帮我们干了！
//    @Bean  
//    public RequestMappingHandlerMapping handlerMapping(){  
//        return new RequestMappingHandlerMapping();  
//    }  
//  
//    @Bean  
//    public RequestMappingHandlerAdapter handlerAdapter(){  
//        return new RequestMappingHandlerAdapter();  
//    }  
}
```

#### Cookie接值
==@CookieValue("cookie的名字")==
```
@RequestMapping("data")  
public String data(@CookieValue(value = "cookieName") String value){  
    System.out.println("value = " + value);  
    return value;  
}
```

cookie会在请求头带回服务器

#### header接值
@RequestHeader("key")
从请求头中获取接收数据
==@RequestHeader("请求头的名字")==
```
public String data(@RequestHeader("Host") String host) {
	sout("Host:" + host);
	return host
}
```

### Servlet原生对象获取
Spring MVC是基于[[JavaWeb#Servlet接口]]的，我们可以使用Servlet原生的一些类的对象。
- HttpServletRequest
- HttpServletResponse
- HttpSession
- InputStream  + Reader
- OutputStream + Writer

ServletContext比较特殊，不能自动获取
方案1，借助HttpSevletRequest 或 HttpSession的getServletContext方法获取
```
ServletContext servletContext = request.getServletContext();
```

方案2，在当前controller类中声明一个ServletContext的Field，然后利用@AutoWired注入
![[Spring FrameWork#^d50302]]
![[JavaWeb#^a706e3]]
```
@Controller
public void ApiController{
	@AutoWired
	private ServletContext servletContext;  //自动装配

	servletContext.XXX 方法就能使用了
}
```


### 共享域
有三个级别

|     | 共享域                            | 范围              |
| --- | ------------------------------ | --------------- |
| 请求域 | request                        | 一次请求            |
| 会话域 | session                        | 一次会话，一个浏览器的多个请求 |
| 应用域 | [[JavaWeb#【接口】ServletContext]] | 整个项目            |

^b7b2e1

| 共享域的操作方法                               | 备注         |
| -------------------------------------- | ---------- |
| setAttribute(String key, Object value) | 向域中存储/修改数据 |
| getAttribute(String key)               | 获得域中的数据    |
| removeAttribute(String key)            | 移除域中的数据    |

^061d4e

获取共享域的方式：
1、原生对象上获取
	request级别：HttpServletRequest
	session级别：HttpSession
	ServletContext级别：@AutoWired private ServletContext servletContext;    自动装配

2、springMVC也提供了几种方式（太复杂，先不记了！）


## SpringMVC 响应数据
Spring MVC如何![[#^0040ff]]
开发模式：

1、混合开发模式：Java三层架构 + 动态页面（js）  ->   最后响应一个组合的html文件。
适用场景：只需要浏览器打开的，例如后台管理系统、CRM、OA等；
主流互联网App不适合，因为返回的html，手机打开不了！ ^5705e2

2、前后端分离模式：前端针对不同设备独立开发（PC端、iOS端、微信小程序端等），与后端交互！
适用场景：主流互联网！
### 页面跳转控制
返回一个页面  --> 混合开发模式

1、配一个试图解析器
2、handler方法返回“index"。这个文件名
3、不要加@ResponseBody，这个注解表示不走视图解析器，直接在页面显示内容！

例如applicationContext设置成 `/springmvc` 映射当前项目
转发（项目下）：返回的字符串前：forward:/jsp/index   可以忽略applicationContext
重定向（项目外）： redirect:/springmvc/jsp/index   必须写成全地址
	Springmvc优化了，帮我们自动加，我们也不用写了，写成 redirect:/jsp/index

### 返回JSON数据
返回一个JSON数据
![image.png](https://raw.githubusercontent.com/rockyshen/blog-img/master/20240310234906.png)

@ResponseBody  加到handler方法，或类上，表示直接响应JSON，不要走视图解析器

@RestController = @Controller + @ResponseBody

将result转为JSON，写会response的示例代码！
![[JavaWeb#^a733e5]]

### 返回静态资源处理
返回一个静态资源，直接用路径找是不行的，因为CEO都是通过秘书找的，静态资源不在其中！

由于SpringMVC重写了Servlet，导致处理静态资源的[[JavaWeb#DefaultServlet]]失效，所以就无法访问静态资源，需要以下方式手动配置：

静态资源处理，需要在MvcConfig配置类中实现WebMvcConfiturer类！并重写configureDefaultServletHandling方法！
```
public class MvcConfig implements WebMvcCOnfiguer{
	@override
	public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
		configurer.enable();    // 开启静态资源查找！
	}
}
```

DefaultServletHandler  二秘书！专门查项目中的静态资源

视图解析器
```
@Override  
public void configureViewResolvers(ViewResolverRegistry registry) {  
    registry.jsp();  
}
```

## RESTful风格设计
[[Django-2#RESTful - 核心概念]]

2024.4.5 最近Twitter上提出GraphQL作为RESTful替代方案，记录之！
[前后端协作关系：引入GraphQL概念](https://www.yuque.com/i5ting/yes0cm/hr4dg1plbdg4l7g1?singleDoc#)

### 接口文档
雷神的谷粒商城项目，就有一个很好的接口文档示例：
[谷粒商城 - 接口文档](https://easydoc.net/s/78237135/ZUqEdvA4/HqQGp9TI)

| 需求   | 接口和请求方式           | 请求参数                       | 返回值        |
| ---- | ----------------- | -------------------------- | ---------- |
| 分页查询 | /user        GET  | page=1&size=10 param       | {JSON响应数据} |
| 用户添加 | /user        POST | {user数据}  是User实体类         | {JSON响应数据} |
| 用户详情 | /user/id    GET   | 路径参数                       | {JSON响应数据} |
| 用户更新 | /user         PUT | {user更新数据}                 | {JSON响应数据} |
| 用户删除 | /user/id   DELETE | 路径参数                       | {JSON响应数据} |
| 条件模糊 | /user/search  GET | page=1&size=10&keyword=关键字 | {JSON响应数据} |

^22caa9

哪些情况下用路径传参，哪些情况下用param传参？
A：对应资源是唯一的，用路径传参；对应资源是多个的话，用param传参！
请求参数限制在10个以内，过多的请求参数导致API难以维护
==敏感信息用POST和请求体来传递参数==

![[Django-2#^f0e920]]




## 全局异常处理
网络连接异常、数据格式异常、空指针异常。我们需要捕捉这些异常。
声明式异常，将异常提取出来！统一处理！  AOP思想啊！
一般情况下，在Common-utils中创建一个异常处理类。
有了这个全局异常处理类，当程序内部发生异常时，不会在页面返回乱七八糟的代码，而是干干净净的JSON！
### 实现步骤
#### Step 1：异常处理类  -->  @ControllerAdvice
步骤1：声明一个异常处理类，加上注解：@ControllerAdvice
@RestControllerAdvice = @ResponseBody + @ControllerAdvice，定义一个异常处理handler方法,返回JSON  

**注意，去配置类中加上扫描@ComponentScan**

全局异常处理类，示例代码
```
@RestControllerAdvice  
public class GlobalExceptionHandler {  
    //1 全局异常处理  
    @ExceptionHandler(Exception.class)  
    public Result error(Exception e) {  
        e.printStackTrace();  
        return Result.fail(null).message("执行了全局异常处理");  
    }  
  
    //2 特定异常处理，传入指定的Exception  
  
    //3 自定义异常处理  
    @ExceptionHandler(GgktException.class)  
    public Result error(GgktException e) {  
        e.printStackTrace();  
        return Result.fail(null).code(e.getCode()).message(e.getMessage());  
    }  
}
```

```
// 自定义异常
@Data  
@AllArgsConstructor  
@NoArgsConstructor  
public class GgktException extends RuntimeException{  
    private Integer code;  
    private String message;  
}
```

#### Step 2：异常处理方法  -->  @ExceptionHandler
步骤2：异常处理类下的方法上，加上注解：@ExceptionHandler，发生什么异常，你怎么处理！
注意：不加@RequestMapping，不用设置访问URL路径

@ExceptionHandler(异常的类)
handler(Exception e)    形参中可以接到这个异常

示例代码
```
@RestControllerAdvice
public class GlobalExceptionHandler {  
      
    @ExceptionHandler(Exception.class)  
    public Object GeneralExceptionHandler(Exception e){  
        // 父类，通用异常处理，当没有找到子类异常时，走父类  
        String msg = "通用异常处理机制响应";  
        System.out.println(msg);  
        return msg;  
    }  
      
    @ExceptionHandler(ArithmeticException.class)  
    public Object arithmeticExceptionHanderler(ArithmeticException e){  
        //自定义处理异常的逻辑即可  
        String msg = "除零异常处理机制响应";  
        System.out.println(msg);  
        return msg;  
    }  
}
```

2、特定异常处理：将Exception 改为 具体的异常，优先处理特定的异常

3、自定义异常处理：先写一个异常类，然后再到try catch的地方抛出自定义的异常，然后写异常处理方法

### 分层异常处理规约

#Java编程规约 
在DAO层（Mapper层）产生异常的类型非常多，无法用细粒度的异常进行catch，使用catch(Exception e)方式，并throw new DAOException(e)，不需要打印日志。

在Service层出现异常时，必须记录日志到磁盘，尽可能带上参数信息，相当于保护案发现场

在Web层绝不继续往上抛异常，当意识到出现错误时，应该跳转到友好错误界面即可。

## 拦截器 HandlerInterceptor 
`org.springframework.web.servlet.HandlerInterceptor`
必须实现HandlerInterceptor接口
只有这三个方法：
	preHandle：在执行完时实际程序之前调用
	postHandle：在执行完实际程序之后调用
	afterCompletion：在完成请求之后调用



![](https://raw.githubusercontent.com/rockyshen/blog-img/master/adfbf0741d1b1ff67fdff43407760acb.PNG)
### 实现步骤
#### Step 1 实现HandlerInterceptor接口
1、加入ioc容器，@Component  （这一步容易遗忘）
2、重写方法，@Override
```
public class MyInterceptor implements HandlerInterceptor {  
    @Override  
    public boolean preHandle
    
	@Override  
	public void postHandle

	@Override  
	public void afterCompletion
}
```
##### preHandle()
在处理handler方法之前执行。适用场景：编码格式设置，登录保护，权限处理
形参：
	request     请求对象，转发
	response   响应对象，重定向
	handler     handler方法

return true  表示放行， false  表示拦截
##### postHandle()
handler方法执行完毕后触发。适用场景：对结果处理，例如敏感词检查！

形参：
	request     请求对象，转发
	response   响应对象，重定向
	handler     handler方法

return modeAndView 表示返回的视图和共享域数据！

只有preHandle返回true，才会触发！
##### afterCompletion()
整体都处理完毕了，会执行此方法

#### Step 2 拦截器加入ioc容器
在MvcConfig配置类中实现WebMvcConfiturer类，并重写==addInterceptors==方法
```
@Configuration  
@ComponentScan({"com.atguigu.controller","com.atguigu.exceptions"})  
@EnableWebMvc  
public class MvcConfig implements WebMvcConfigurer {  
    @Override  
    public void addInterceptors(InterceptorRegistry registry) {  
        // 拦截全部请求  
        registry.addInterceptor(new MyInterceptor());  
    }  
}
```

指定地址拦截  .addPathPatterns("url")
```
registry.addInterceptor(new MyInterceptor()).addPathPatterns("/user/**");

只拦截user下的所有路径
```

.excludePathPatterns()     排除指定URL

多个拦截器的执行顺序：
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B43%E6%9C%8811%E6%97%A5%2016_17_31.jpg)


#### 总结各种过滤/拦截方案
在Spring工程中，可以用来作为统一业务处理的方法有好几种：

1、原生Servlet下的Filter  [[JavaWeb#Filter 过滤器]]
![[JavaWeb#^afe1d0]]

2、Spring MVC中的拦截器[[#HandlerInterceptor 拦截器]]
Filter只能拦截请求发起与Servlet之间
HandlerInterceptor可以在任何位置（CEO-秘书-经理-handler）进行拦截！

3、Spring Cloud Gateway自定义全局过滤器    [[SpringCloud#GlobalFilter 全局过滤器]]

![[JavaWeb#^fde18a]]

## 参数校验
-> JSR303
对前端传过来的实体类的Field值进行参数校验！

主要有两种：
1、hibernate-validator 是 Hibernate 提供的 Bean Validation 的实现库
由Hibernate提供的注解，需要导这个包！
```
<!-- 校验注解，依赖导入 -->

<dependency>  
    <groupId>org.hibernate.validator</groupId>  
    <artifactId>hibernate-validator</artifactId>  
    <version>8.0.0.Final</version>  
</dependency>  
  
<dependency>  
    <groupId>org.hibernate</groupId>  
    <artifactId>hibernate-validator-annotation-processor</artifactId>  
    <version>8.0.0.Final</version>  
</dependency>
```

2、spring-boot-starter-validation：spring-boot-starter-validation 是 Spring Boot 提供的依赖库，它集成了 hibernate-validator 并提供了更方便的使用方式。
雷神在谷粒商城用的依赖是:
```
<!--        后端实体类字段的校验注解相关依赖-->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-validation</artifactId>  
        </dependency>
```


**步骤1：==在实体类的Filed上加注解==，接值时，Spring会帮我们校验！**
```
@Data  
public class User {  
    @NotNull  
    @NotBlank    private String name;  
  
    @Length(min = 6)  
    private String password;  
  
    @Min(1)  
    private int age;  
  
    @Email  
    private String email;  
  
    @Past  
    private Date birthday;  
}
```


**步骤2：handler方法的形参前，必须加@==Validated==注解，才能执行校验规则！**
	@Validated({UpdateGroup.class})   分组校验，需要手从见几个接口，里面不用写任何内容！
	接收的实体JSON数据的话，再加一个@RequestBody

@Valid注解  是老版本，不能分组


| 注解                         | 规则                    | 使用场景               |
| -------------------------- | --------------------- | ------------------ |
| @Null                      | 标注值必须为null            |                    |
| @NotNull                   | 标注值必须非空               |                    |
| @AssertFalse               | 标注值必须为False           |                    |
| @AssertTrue                | 标注值必须为True            |                    |
| @Min(val)                  | 标注值必须大于等于val          |                    |
| @Max(val)                  | 标注值必须小于等于val          |                    |
| @DecimalMin(val)           | 标注值必须大于等于val          |                    |
| @DecimalMax(val)           | 标注值必须小于等于val          |                    |
| @Size(max,min)             | 标注值必须在max-min之间       | 一般指长度大小，例如字符串或集合长度 |
| @Digits(interger,fraction) | 标注值必须是一个数字，且必须在可接受范围内 | 校验整数及浮点类           |
| @Past                      | 标注值只能日期型，且必须是过去日期     | 生日                 |
| @Future                    | 标注值只能日期型，且必须是将来日期     | 预约时间               |
| ==@Pattern(value)==        | 标注值必须符合[[re.正则表达式]]   |                    |
| @Email                     | 格式正确的email            |                    |
| @Length                    | 标注值字符串大小必须在指定范围内      |                    |
| @NotEmpty                  | 标注值字符串不能是空字符串         | 集合长度不为空            |
| @Range                     | 标注值必须在指定范围内           |                    |
|                            |                       |                    |

易混点：
@NotNull  ，对象不为空，包装类型
@NotEmpty   ==集合==不为空，且长度大于0
@NotBlank    ==字符串==不为空，且不为空字符串


**步骤3：接收错误绑定信息 BindingResult**
1、步骤1：handler(校验对象，BindingResult result)，必须紧挨着校验对象！
2、步骤2：利用BindingResult对象result，调用方法进行后续处理

BindingResult类的对象里存储了校验结果信息。
```
@PostMapping("register")  
public Object register(@Validated @RequestParam User user, BindingResult result){  
    if (result.hasErrors()) {   //当碰到错误，不抛异常，走这里的方法  
        Map data = new HashMap();  
        data.put("code","400");  
        data.put("msg","参数异常了"); 
        return data;              //将这个map返回给前端，而不是抛出异常  
    }
    return user;
```

### 自定义校验器
ref 雷神 Ep69，待补充


## 跨域访问 CORS
同源策略：协议相同、地址相同、端口相同！
![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202408231129641.png)
### 1、注解方式：@CrossOrigin
允许前端程序的node服务器（端口号：5173）访问我，浏览器你别拦截他！

需要加到Controller的类上，或加到GET/POST等方法上！
@CrossOrigin  需要在每一个handler方法上，逐个去加，效率太低

示例：
```
@RestController
@RequestMapping("/user")
@CrossOrigin(origins = {"http.user.code-nav.cn","域名2"}, methods = { RequestMethod.GET})
public class UserController {
	//略
}

上述配置中，只允许user.code-nav.cn这个域名发过来的请求允许跨域访问本后端服务，其他都不允许！
```

### 2、配置类方式：CorsConfig implements WebMvcConfigurer
写一个配置类CorsConfig，再Response响应体中加入允许跨域访问的字段。这个更统一！

联想到，利用Nginx中的配置中，也是在响应中加上允许跨域的字段，原理相同：[[Nginx#利用Nginx配置解决跨域]]

本质上就是用了下面的方法：
![[网络#^a0ac4f]]


跨域配置类示例代码：
```
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")    // 覆盖所有请求
		        //重要！（必须用 patterns，否则 * 会和 allowCredentials 冲突）
                .allowedOriginPatterns("*")   // 放行哪些域名
                .allowCredentials(true)    // 允许发送 Cookie
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .allowedHeaders("*")
                .exposedHeaders("*");
                .maxAge(3600);    // 跨域允许时间
    }
}
```

注意：`allowedOrigins("*")`和`allowCredentials(true)`为`true`时候会出现错误！需要改成`allowedOriginPatterns("*")`或者单独指定接口`allowedOrigins("http//www.baidu.com")`

注意：不同版本的Spring Boot，对于跨域配置类的设置上有区别哦！
因为用的是2.3.2.RELEASE  低版本的Springboot 用这个  allowedOrigins
2.6.6 高版本的  用这个   allowedOriginPatterns ^bde019
```
@Configuration  
public class CorsConfig implements WebMvcConfigurer {  
    @Override  
    public void addCorsMappings(CorsRegistry registry) {  
        registry.addMapping("/**")  
	        // 新版SpringBoot下，allowedOrigins替换为llowedOriginPatterns
            .allowedOrigins("*")               
            .allowCredentials(true)
            .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")  
            .maxAge(3600);
    }  
}
```


思考：@CrossOrigin和跨域配置类有什么区别？
回答：@CrossOrigin加到Controller方法上，比较麻烦。如果想要整个微服务系统都实现跨域，就需要用到config/CorsConfig配置类！

联想到Django框架中的跨域解决方案：[[Django-3#CORS 跨域请求]]

### 3、[[SpringCloud#网关：Gateway]]
在Spring Cloud GateWay中配置，以实现全局或按路由控制 CORS 行为
![[SpringCloud#配置跨域]]

## 统一返回结果类 Result

### 领域模型规约
POJO是DO/DTO/VO/BO的统称，禁止命名成 xxxPOJO

```
├── model
	├── pojo             # 也就是do，User类等
	├── request          # 封装request请求对象
		├── UserRegisterRequest    # 用于请求注册的用户对象（可能只包含用户名、密码）
```

#### DO
Data Object
此对象与数据库结构一一对应，通过Mapper层向上传输数据源对象
数据对象，xxxDO 其中的xxx即为数据库表名
实体类包！
#### DTO
Data Transfer Object
数据传输对象，Service层向外传输的对象。业务层！

BeanUtils.copyProperties(数据源，目标)，前提是字段名Fields必须保持一致。
例如将Entity的值，宝贝到VO中

如果是两两对拷，用BeanUtils.copyProperties方法最合适；属性两两对拷！
假如B对象的Field值，来自A、C不同类，那就不能用这个方法了，老老实实get、set方法！
```
// subjectEevo --> subject
BeanUtils.copyProperties(subjectEeVo,subject);   将subjectEeVo中Field值，拷贝到subject中
```
![[Java_01_基础#^c62eaf]]


#### Query
数据查询对象，各层接收上一层的请求，注意：超过两个参数的查询封装，禁止使用Map类型来传输。应该利用Query封装。
UserRegisterRequest    就是用户注册请求的对象，封装了：用户名、密码、确认密码

前端传过来的JSON，我们的DO字段比它更多（有些不需要用户传递的信息：updateTime等）
例如：QueryVo  封装查询条件的VO
![[Django-2#^9c09ae]]
#### VO
View Object
展示对象，xxxVO   xxx就是网页名称

通常是Web向模板渲染层传输的对象。


### Result类
统一返回结果，后端所有的接口handler方法，==统一返回值都固定是Result数据类型==，固定类型返回给前端。

搭配[[#全局异常处理]]一起使用。

![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202409111110057.png)
OneDrive-个人/文档/02-学习/Learn_Java_Mosh/统一返回结果Result类.excalidraw

4大标配
	code 状态码
	message 描述
	data  数据（一个合格的后端，返回的数据一定要带唯一标识，前端的key要用！）[[Vue_01_基础#key的原理]]
	timestamp 时间戳

固定下来之后，我就用固定的！不用一直改，用一套自己熟悉的最重要！

参考我维护的template-utils中的Result类，模版参考！
https://github.com/rockyshen/template-utils

### ReturnCodeEnum类
状态码枚举
枚举类的步骤：举值-构造-遍历
