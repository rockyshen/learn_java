狭义上简称为：Spring，其实指的是Spring FrameWork，底层基础框架！
使用版本：Spring 6.0.6
当前最新是Spring FrameWork 6，要求Java 17
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/spring-overview.png)

| 重要功能模块         | 功能介绍                                      |
| -------------- | ----------------------------------------- |
| Spring Core    | 提供框架基础部份，控制反转Ioc和依赖注入DI                   |
| Spring Beans   | 提供BeanFactory，工厂模式的经典实现，Spring将管理对象成为Bean |
| Spring Context | 基于Core的封装，提供框架式访问对象的方法                    |
| Spring JDBC    | 提供了JDBC的抽象层，提供一套规范给各数据库厂商，简化连接、操作数据库的难度   |
| Spring AOP     | 面向切面编程，自定义拦截器                             |
| Spring Web     | 提供面向Web应用程序的集成功能                          |
| Spring Test    | 为测试提供支持，支持JUnit进行单元测试                     |
|                |                                           |
## 基本概念
### 组件 bean
组件：==**可以复用的**==java对象！（有些对象实例化一次之后，就不用了，就不能称之为组件）

组件交给Spring来管理的优势：
1、组件之间解耦合，因为组件A与组件B不实际交叉，交叉关系在Spring配置文件中记录
2、可复用和可维护性，只要改配置文件修改组件之间的关系即可，不用重写业务代码
3、可享受[[Spring家族]]中其他功能，例如AOP、事务管理

思考：实体类不用交给ioc容器进行管理，因为ioc容器主要管理和注入带有业务逻辑的组件（controller、repository、service），实体类可以通过mybatis持久层框架与数据库交互，不用ioc容器去管理。
### 容器 context
- 普通容器：数组、列表、集合等都是普通容器，只具备存储数据的功能，没有周期管理等复杂功能
- 复杂容器：Servlet容器能够管理Servlet（init、service、destroy）、Filter、Listener。这样的组件的一生，==强调生命周期==！是复杂容器
		IoC容器，不仅存储组件，也可以管理组件之间的依赖关系，并创建和销毁组件，所以是复杂容器。

## IoC 容器
Spring框架替代了程序员原有的new对象、对象属性赋值等动作。我们只需要编写配置文件，告诉Spring组件之间的关系即可。

ioc:基于[[Java_01_基础]] EE的框架，针对bean的生命周期进行管理的轻量级容器。

### 核心功能
IOC和DI都是发生在Spring容器内部，是Spring容器的核心功能！
#### IoC
==Inversion of Control  控制反转，对应的就是创建实例化对象的功能==
传统JavaEE中，当我们需要一个对象时，需要程序直接创建对象（new,实例化），现在控制反转了，==创建和管理权移交给IoC容器==！

将创建管理对象的权限交给Spring Ioc容器管理，开发者只需要将对象的依赖关系告诉 （@Autowired）Spring 容器。当对象被创建时，容器会自动为其注入所需的依赖对象，而不需要对象自己去查找依赖或者创建依赖对象。解藕！

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B42%E6%9C%8828%E6%97%A5%2010_05_49.jpg)
我的理解：
结合领域驱动设计的思路[[Java_02_面向对象#领域驱动设计]]，领域类主要构建场景，包括field和method的编写，服务类（测试类）主要是实例化对象，调用方法。控制反转就是将服务类的实例化对象这些操作移交给Spring核心容器来处理，程序员只需要关系领域类的场景构建即可！
#### DI
==Dependency Injection 依赖注入，对应的就是对象之间依赖关系管理的功能==
在IoC容器内部，实例化了两个对象，我们可以通过配置文件告诉容器，将对象2注入到对象1中（方式有：*构造函数传参、setter方法*） ^1dddef

下面这段代码来自[[Java_03_高级特性#共享数据]]，就是通过构造器传参的方式，将clerk对象注入Producer中！
![[Java_03_高级特性#^63b4a7]]
### IoC容器的接口和实现类
#### Ioc容器接口
![[Java_02_面向对象#^c0b956]]

###### BeanFactory接口
接口，是IoC容器的超级接口，父级接口！
约定基本的核心容器的操作：创建组件、存储组件、对外提供组件 ^c3f35c

易混淆
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-07-25%20at%2023.53.50%402x.png)
**BeanFactory**   是所有Bean的工厂，是ioc的顶层接口，本质上是ioc容器
**FactoryBean**   是提供给第三方依赖加入ioc的工厂模式，方便第三方依赖加入的，本质上是Bean! ^8c3ae3

@马士兵：
BeanFactory：是Bean工厂，或者说是IoC容器的入口（源码中的原话），他的子类DefaultListableBeanFactory类就是IoC容器本身，所以BeanFactory也可以理解为容器。里面的bean对象的生产遵循一套标准化的流程（[[#【重要】Bean的生命周期]]

FactoryBean：该接口主要有：isSingleton、getObjectType、getObject三个规约方法，相当于提供了私人订制的生产bean对象的方法。多用于第三方框架整合到Spring时使用！

###### ApplicationContext接口
继承自BeanFactory接口，子接口
在BeanFactory接口基础上，提供更完善的功能
拓展功能有：事件发布和监听功能、国际化支持、多种资源访问方式、添加事务、实现AOP、


所有我们后续使用的都是，ApplicationContext接口的实现类！
###### 4种实现类
下表为基于ApplicationContext接口的四种容器实现类，**选一种即可**。每个项目new实例化一个即可！

| 实现类                                 | 简介                                          |        |
| ----------------------------------- | ------------------------------------------- | ------ |
| ClassPathXmlApplicationContext      | 读取编译后target文件夹下的xml                         | 读取xml  |
| ~~FileSystemXmlApplicationContext~~ | ~~通过文件系统路径(D盘、E盘等)的xml配置文件创建IoC容器对象~~       |        |
| AnnotationConfigApplicationContext  | 通过读取Java类（作为配置文件）创建IoC容器对象                  | 注解+配置类 |
| WebApplicationContext               | 专门为Web准备，[[SpringMVC]]，隐藏在@Configuration注解中 |        |

### 基本步骤
#### Step 1：编写配置文件
编写配置文件，主要是记录：组件类信息、组件之间的引用关系

主要有三种形式：
	1、[[#Spring配置文件 - xml]]
	2、[[#Spring配置文件 - 注解]]  
	3、 [[#Spring配置文件 - 配置类]]
#### Step 2：实例化IoC容器
java代码（程序主入口，测试环境）中实例化ioc容器对象，启动ioc容器，引用配置文件（ioc容器内部将发生ioc和di）
#### Step 3：获取Bean组件
在java代码（程序主入口，测试环境）中，通过ioc容器对象，获取ioc容器的组件bean，并调用不同的组件的方法

### Spring配置文件 - xml
xml是最原始的方式，通过xml文件定义Bean及其依赖关系；
之后逐渐过渡到后两种方式：注解、java配置类（适用于SpringBoot）
#### `<bean 属性>`标签
需要加入IoC容器的对象，称为Bean

[[Java_02_面向对象#实例化对象的两种方式]]：
	1、构造器(有参、无参)
	2、(静态、非静态)工厂模式
注意：==不同的实例化对象的方式，在ioc容器的配置文件中是不同的！==

| ClassPathXmlApplicationContext      | 通过类路径下(/src或/resources文件夹下)的xml配置文件创建IoC容器对象， |
| ----------------------------------- | --------------------------------------------- |
IDEA中，我已经将spring-xml作为模版，加好了，新建时直接选就行了！自动生成带Spring-xml约束标签名的xml配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>  

<!--    spring指定的约束文件，约定了可以在spring框架下使用的标签名-->
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```

##### 常用属性
###### id
也就是beanName，唯一标识，不允许重复

重要：同一个Spring配置文件xml中，bean的id必须唯一，否则报DuplicateBeanDefinition。 ^8ada82

背后的原理，因为基于BeanDefiniton创建Bean时，是基于beanName进行for循环的！ ^16fbb7

###### class
指定这个bean的类路径，类的全限定符
###### scope
bean的作用于，默认就是singleton
###### consrtrucor-arg
设置bean的构造函数的参数
[[#有参构造器 = 依赖注入]]
###### property
设置bean的属性值，ref表示引用类型，ref=类全限定符
[[#利用setter方法向Field注入值 `<property>`]]
###### init-method
指定bean初始化时调用的方法，比较少用到。不指定就用默认的方法。
###### depends-on
指定当前bean依赖的其他bean
[[#循环依赖]]
###### parent
指定当前bean的父类

##### `<bean>`标签与@Bean属性对照表
1、在xml配置文件中使用`<bean>`标签的类，将会被注册进IoC容器中
2、@Bean注解通常用在@Configuration类的方法上，通常需要自定义bean实例化的逻辑，@Bean注解的方法返回对象会加入IoC容器。（一般用在配置类上）
3、@Component注解，通常用于标注普通bean，不需要特殊的初始化逻辑，所以属性都是默认的！

| `<bean>`                                        | @Bean                                                                        |
| ----------------------------------------------- | ---------------------------------------------------------------------------- |
| <bean id="beanName" ...>                        | @Bean(name="beanName")                                                       |
| <bean class="com.example.MyBean" ...>           | @Bean <br>public MyBean myBean() { ... }                                     |
| <bean scope="singleton" ...>                    | @Scope("singleton")                                                          |
| `<constructor-arg name="name" value="value" />` | @Bean <br>public MyBean(String name) { ... }                                 |
| `<property name="name" value="value" />`        | @Bean <br>public MyBean myBean() { bean.setName("value"); <br>return bean; } |
| `<bean depends-on="bean1,bean2" ...>`           | @DependsOn({"bean1", "bean2"})                                               |

##### @Bean属性与@Component属性对照表

| @Bean                                                                        | @Component                                     |
| ---------------------------------------------------------------------------- | ---------------------------------------------- |
| @Bean(name="beanName")                                                       | @Component("myBean")<br>默认类名第一个字母小写作为 bean 的名称 |
| @Bean <br>public MyBean myBean() { ... }                                     | @Component(scope="singleton")<br>默认就是单例模式      |
| @Scope("singleton")                                                          |                                                |
| @Bean <br>public MyBean(String name) { ... }                                 |                                                |
| @Bean <br>public MyBean myBean() { bean.setName("value"); <br>return bean; } |                                                |
| @DependsOn({"bean1", "bean2"})                                               |                                                |
##### 构造函数
###### 无参构造器
```
//示例类，使用无参构造器
public class HappyComponent {  
    // 无参构造器，编写ioc容器的xml配置文件  
    public void doWork() {  
        System.out.println("HappyComponent.doWork");  
    }  
}
```

```xml
<!--    无参构造器的xml配置信息-->  
    <bean id="happyComponent1" class="com.atguigu.ioc_01.HappyComponent" />
    <bean id="happyComponent2" class="com.atguigu.ioc_01.HappyComponent" />
```

id：唯一标识符
class：类的全限定名
默认是单例模式，如果一个class，写了两个bean，就是new两个实例化对象。


###### 有参构造器 = 依赖注入

^ce877d

因为给当前类的构造器里传东西，就是依赖注入的过程！
其实就是依赖注入 [[#构造函数注入 `<constructor-arg>`]]

##### 工厂方法
![[Java_02_面向对象#^e14ede]]
###### 静态工厂
```
//示例类，使用静态工厂
// 静态工厂模式，且为单例模式  
public class ClientService {  
    // Field，类的内部创建当前类的实例  
    private static ClientService clientService = new ClientService();  // 私有静态Field，单例模式  
  
    // 私有化构造器，单例模式  
    private ClientService(){}      // 将构造器设置为private，外部无法通过new实例化，毕竟是工厂模式！  
  
    // 静态工厂方法调用，获取实例  
    public static ClientService createInstance(){   // 公共静态方法，利用无参静态方法来实例化  
        return clientService;  
    }  
}
```

```xml
<!--    静态工厂的xml配置信息-->  
    <bean id="clientService" class="com.atguigu.ioc_01.ClientService" factory-method="createInstance" />
```
id
class
factory-method   必须是静态方法，否则报错！

###### 实例工厂
```
//示例类，使用实例工厂
// 实例工厂  
public class DefaultServiceLocator {  
    private static ClientServiceImpl clientService = new ClientServiceImpl();  
  
    public ClientServiceImpl createClientServiceInstance(){  
        return clientService;  
    }  
}

如何使用实例工厂进行实例化
var defaultServiceLocator = new DefaultServiceLocator();  //实例化工厂实例

ClientServiceImpl clientService = defaultServiceLocator.createClientServiceInstance();   //基于工厂实例调用实例化方法

```

```xml
<!--    实例工厂的xml配置信息-->  
    <bean id="defaultServiceLocator" class="com.atguigu.ioc_01.DefaultServiceLocator" />  
    <bean id="clientService2" factory-bean="defaultServiceLocator" factory-method="createClientServiceInstance"/>
```

##### 【❄️经典白雪】
-> [[#@ComponentScan("类的全限定符")]]
【经典白雪】xml方式将类声明为Bean组件，太麻烦了，了解其原理即可。
未来会在自定义类上，加@Component ，@Controller ， @Service等方式，自动声明为这是需要被加入ioc容器的Bean！
[[#使用注解加入ioc容器 --> @Component]]    [[#@Bean]]

#### DI的配置：依赖注入
Spring是依赖注入框架 [[Java_02_面向对象#依赖注入]]

依赖注入的配置，也就是==声明Bean与Bean之间的关系==
![[#^1dddef]]

##### 向构造函数中注入参数 `<constructor-arg>` 
![[#^ce877d]]
###### 传递一个参数
```xml
    <bean id="userService" class="com.atguigu.ioc_02.UserService">  
<!--            指定构造器传参，依赖注入，DI-->  
        <constructor-arg ref="userDao"/>  
    </bean>
```

^7517f2

在bean内部的constr-arg标签注入构造器参数
	value表示参数是基元类型
	ref表示参数引用类型。
###### 传递多个参数
```xml
方式1
    <bean id="userService" class="com.atguigu.ioc_02.UserService">  
<!--            指定构造器传参，依赖注入，DI-->  
        <constructor-arg value="18"/>  
        <constructor-arg value="张三"/>  
        <constructor-arg ref="userDao"/>  
    </bean>

方式2，用constructor-arg标签的name值指定每个属性的名字，就不用管顺序了！
    <bean id="userService" class="com.atguigu.ioc_02.UserService">  
<!--            指定构造器传参，依赖注入，DI-->  
        <constructor-arg name="age" value="18"/>  
        <constructor-arg name="name" value="李四"/>  
        <constructor-arg name="userDao" ref="userDao"/>  
    </bean>
```
必须按参数顺序，value直接填值。
constructor-arg   name  参数的名称！
constructor-arg   index  参数的序号，0、1、2

##### 利用setter方法向Field注入值 `<property>`
在bean内部的property标签注入setter()参数：
	**==name表示set方法名字，去掉set关键字，首字母变小写==**！
	value表示参数是基元类型，ref表示参数引用类型。
```
public class MovieFinder{}

public class SimpleMovieLister{  
    // Field  
    private MovieFinder movieFinder;  
    private String movieName;  
      
    //setter方法注入
    public void setMovieFinder(MovieFinder movieFinder) {  
        this.movieFinder = movieFinder;  
    }  
  
    public void setMovieName(String movieName) {  
        this.movieName = movieName;  
    }  
}
```

setMovieFinder()    ---->    movieFinder
```xml
<bean id="movieFinder" class="com.atguigu.ioc_02.MovieFinder"/>  
<bean id="simpleMovieLister" class="com.atguigu.ioc_02.SimpleMovieLister">  
    <property name="movieFinder" ref="movieFinder"/>  
    <property name="movieName" value="肖申克的救赎"/>  
</bean>
```

缺点：必须要有setter方法才行，没有的话要手写
##### 【❄️经典白雪】
之后给Fiedl依赖注入值，可以用[[#基元类型DI --> @value("key")]]  或  [[#引用类型DI --> @Autowired]]

![[#^458366]]

#### 创建ioc容器
[[#ApplicationContext接口]]

方式1：创建ioc容器实例，并指定配置文件即可 ^caa717
```
public class SpringIoCTest {  
    public void creatIoC(){  
        // 四种ioc容器接口实现类，四选一
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-03.xml");  
    }
}
```

^d56778

方式2：先创建容器实例，再指定配置文件，再refresh()（源码底层的实现）
```
1、ClassPathXmlApplicationContext applicationContext1 = new ClassPathXmlApplicationContext();  

2、applicationContext1.setConfigLocations("spring-03.xml");  

3、applicationContext1.refresh();
```

##### 【❄️经典白雪】
-> [[#@Configuration]] 
**【经典白雪】**：这种方式只是原理了解！未来利用注解+配置类的方式会很简单！
在配置类上，@Configuration 指定配置类，@ComponentScan扫描组件加入ioc
#### ioc容器中获取bean
方式1：调用==getBean(ID)==，根据BeanID，返回值类型是Object，==需要强转==，不推荐。
```
Object happyComponent = (HappyComponent)applicationContext1.getBean("happyComponent");
```

方式2：调用==getBean(ID, class)==，根据BeanID，并指定class（返回值类型）
```
HappyComponent happyComponent1 = applicationContext1.getBean("happyComponent", HappyComponent.class);
```

方式3：调用==getBean(class)==，根据class，直接获取
注意1：同一个类型，在ioc容器中只能有一个bean，否则会出现NoUnique异常！单例模式！
用BeanID则不会存在这个问题。
```
HappyComponent happyComponent2 = applicationContext1.getBean(HappyComponent.class);
```

注意2：ioc的配置bean必须是实现类（因为要实例化，接口不能实例化），但是可以在场景中根据接口获取bean实例（实现类只能有一个，不然也报不NoUnique异常）！
示例：
```
public class HappyComponent implements Happyable{       // 接口：Happyable
    // 实现Happyable接口的方法  
    @Override  
    public void doWork() {  
        System.out.println("HappyComponent.doWork");  
    }  
}  
```

ioc配置bean的时候，一定是实现类：com.atguigu.ioc_03.HappyComponent
```xml
<bean id="happyComponent" class="com.atguigu.ioc_03.HappyComponent"/>
```

可以根据接口类型获取bean实例！
```
Happyable happyComponent3 = applicationContext1.getBean(Happyable.class);  
happyComponent3.doWork();
```

##### 【❄️经典白雪 】
-> [[#引用类型DI --> @Autowired]]
【经典白雪】利用实例化applicationContext的ioc容器，然后基于容器实例调用getBean方法，是底层原理，了解即可。后续，我们会使用@Autowired注解将组件注入Field直接调用，不需要我们手动写getBean()去手动从ioc容器中获取组件的！注意！
[[#引用类型的依赖注入 @Autowired -> Field]]
#### Bean的作用域 scope
利用`<bean>`标签实例化对象，在Spring内部是将它转换为`BeanDefination`类的一个实例化对象，该对象内部包含id、class等信息。
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B42%E6%9C%8829%E6%97%A5%2011_31_24.jpg)

scope指定基于一个bean，创建几个实例化对象。默认是单例模式，只创建一个实例。
	singleton：【默认】单例模式，只实例化一个，高并发时，用多线程而不是多对象！
	prototype：多例模式，每调用一次，实例化一次
	request：Web项目，请求范围内
	session：Web项目，会话范围内 ^9f1716

思考：单例模式和多例模式的Bean，在生命周期上有什么区别？ #Java八股/SSM 
单例模式的Bean，由ioc容器管理，bean的生命周期等于容器的生命周期！
多例模式的Bean，不由ioc容器管理，由Spring为我们创建，每次调用getBean()就会返回一个Bean对象，生命周期不归ioc容器管理，主要在使用就活着，不使用了就会被GC。

#### 【重要】Bean的生命周期
单例模式下，Bean的生命周期和容器相同
多例模式下：
	出生：使用对象时由Spring框架为我们创建；
	活着：对象只要在使用过程中，就一直活着
	死亡：当对象长时间不被引用，就会被GC

![[#^ad8461]]

展开来说：
1、实例化一个Bean，也就是我们常说的new
2、按照Spring上下文对实例化的bean进行配置，也就是Ioc注入
3、检测Aware接口
4、（如有）初始化前置任务触发（BeanPostProcessor接口的postProcessBeforeInitialzation方法）
5、（如有）触发init-method方法
6、如有）初始化后置任务触发（BeanPostProcessor接口的postProcessAfternitialzation方法）
7、完成以上工作，我们代码中就能用这个bean了。
8、当bean不需要时，会执行destory方法
##### 实例化
##### 属性赋值
##### 初始化 Initialization

init-method
执行初始化调用方法

联想到python中的：[[类#`__new__`和`__init__`]]   一个是实例化对象，一个是属性赋值
但是Java中，属性赋值已经在前面完成了，这个方法就没啥作用了！

在xml中指定init-method 和 destory-method
```
<bean id="javaBean" class="com.atguigu.ioc_04.JavaBean" init-method="init" destroy-method="clear"/>
```

我们还可以用它做一个其他用处

##### 销毁



周期方法，需要你手写具体逻辑，然后在xml中关联声明！

bean的周期方法负责管理bean在ioc容器内部的初始化和销毁:
	当代码中创建ioc容器时，就会触发init方法
	当正常结束ioc容器时，就会触发destroy方法（如果不调close()，则不会触发destroy)
联想到servlet的init、destroy和tomcat的关系

bean的周期方法，==必须是public、void、无参，命名随意==！
```
public class JavaBean {  
    //初始化  
    public void init() {}  
    public void clear() {}  
}
```

注意：多例模式下，不会调用周期方法的：销毁方法，太多了，不想管了！

##### 循环依赖问题
A对象中有个B类型的属性，B对象中有一个A类型的属性，就构成了循环依赖问题。

Spring容器在创建bean实例化以后，要给bean的属性自动赋值，要全部赋值之后才能交给用户使用。循环依赖就无法使Spring容器启动。

三级缓存解决循环依赖问题
就是三个Map存储不同阶段的Bean对象。
基本思路是：在Bean创建过程中，将正在创建的Bean对象让入一个专门用于缓存创建中Bean的缓存池中，当后续创建其他Bean对象时，如果依赖于该缓存池中正在创建的Bean时，则直接使用缓存池中Bean对象，而不是重新创建一个新的Bean对象。

| XX   | XX                    |                                                          |
| ---- | --------------------- | -------------------------------------------------------- |
| 一级缓存 | singletonObjects      | 主要存放，已经完成实例化、属性填充、初始化所有步骤的单例Bean实例。成品Bean                |
| 二级缓存 | earlySingletonObjects | 主要存放，已经完成初始化，但是属性还没赋值的Bean，还不能提供给用户使用，半成品Bean（或称：早期Bean） |
| 三级缓存 | singletonFactories    | 主要存放，提前暴露的单例bean引用，lamda表达式                              |

具体步骤：
1、首先从一级缓存（SingletonObjects)中查找是否存在已经创建过的Bean对象，若有，直接返回；
2、如果一级缓存没找到，则从二级缓存（earlySingletonObjects）中查找该Bean对象的早期对象（还没进行属性填充和初始化的半成品），若存在，则返回早期对象
3、若耳机缓存中不存在该Bean对象早期对象，则将正在创建的Bean对象放入三级缓存（singletonFactories)中，并在创建过程中进行依赖注入，即为该Bean对象注入依赖的其他Bean对象。此时如果其他Bean对象中依赖了正在创建的Bean对象，SPring直接从三级缓存中获取此正在创建的Bean对象，而不是重新创建一个新的。
4、当Bean对象完成创建后，Spring将其从三级缓存中移除，并将其加入一级缓存中，以便下次获取该Bean对象时直接从一级缓存中获取。

[[#循环依赖]]

##### 【高级特性】Bean生命周期相关接口
先Bean的实例化和属性赋值，再initMethod方法
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Bean%E5%88%9B%E5%BB%BA%E6%B5%81%E7%A8%8B.png)
###### BeanPostProcessor接口
是Spring为修改Bean提供的强大扩展点，可作用于容器中所有bean！

下面有两个实现类：
- postProcessBeforeInitialization()：在Bean调用初始化方法initMethod()前调用，可以对Bean进行一些预处理操作。
- postProcessAfterInitialization()：在Beand调用初始化方法initMethod()后调用，可以对Bean进行一些后处理操作。

###### InitializingBean接口
org.springframework.beans.factory.InitializingBean

InitialiZingBean接口是Spring为bean初始化提供的扩展点。提供最后一次机会，给bean对象的属性进行赋值。

![image.png](https://raw.githubusercontent.com/rockyshen/blog-img/master/20240529222424.png)

- **接口方法】afterPropertiesSet()**
适用情况：一个ConstantPropertiedUtil类，加入ioc容器，创建时就调用afterPropertiesSet()方法，读取数据库中的值，赋给对象的Field上，这样对象在完成实例化，加入ioc容器时，就有这些数据了！

```
@Component  
@Data  
public class ConstantPropertiesUtil implements InitializingBean {  
  
    @Value("${spring.cloud.alicloud.oss.endpoint}")  
    private String aliyunOssEndpoint;
    
	public static String ACCESS_KEY;  
  
	@Override  
	public void afterPropertiesSet() throws Exception {  
	    ACCESS_KEY = aliyunOssAccessKey;
	    }
```

联想到[[SpringBoot#@ConfigurationProperties（prefix = "druid"）]]
```
@Component
@Data
@ConfigurationProperties(prefix = "spring.cloud.alicloud.oss")
public class ConstantPropertiesUtil implements InitializingBean {
    private String endpoint;
    private String accessKey;

    public static String ACCESS_KEY;

    @Override
    public void afterPropertiesSet() throws Exception {
        ACCESS_KEY = accessKey;
    }

}
```

[[#作用域及周期方法]]


#### 【高级特性】FactoryBean接口`<T>`
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-07-25%20at%2023.53.50%402x.png)
- 如果不特殊声明，就是走BeanFactory的标准化流程：refresh()的13个方法
- 如果声明了实现`FactoryBean<T>`，就是走私人定制的方向！
	在接口方法getObject这个方法中，你可以用new的方式、动态代理的方式、反射的方式，创建对象，随便你！
	实现了FactoryBean的类，getObject方法返回的对象，自动会加入IoC容器 ^152080

所谓自动，就是在实例化Bean的流程中，第一步就是先判断一下有没有实现FactoryBean接口，私人定制的Bean的步骤
```
	if (isFactoryBean(beanName)){}     # 私人定制 bean！
	else { getBean }                   # 标准化流程 13步refresh()
```

一般用于==配置第三方框架的bean对象==，将创建过程的具体实现逻辑放在getObject()中。


你只要按照我接口定义的三个方法，override重写之后，就能将自己业务逻辑实现在创建Bean的过程中。
![[#^8c3ae3]]


使用场景：
1、代理类的创建
2、==第三方框架整合到spring框架中。最喜欢使用！==
	mybatis中SqlSessionFactoryBean就是FactoryBean接口的实现类
3、复杂对象的实例化

需要一个具体的案例，来辅助理解FactoryBean的使用
	举例：mybatis的使用，最后我们需要一个SqlSession的实例化对象，基于此来进行数据库操作，但是获取SqlSession实例化对象需要非常复杂的4-5条语句[[Mybatis#^cd1d90]]，我们就可以利用SqlSessionFactoryBuilder，将创建SqlSession实例化对象的逻辑封装在其中！

联想：[[#静态工厂]]和[[#实例工厂]]。
##### 步骤
0、先写一个需要加入Ioc容器的第三方类
1、实现FactoryBean接口，指定泛型为这个类！
2、重写getObject()，写具体逻辑，return的对象就会自动加入IoC容器
3、重写getObjectType()，指定返回值类.class
4、配到xml配置文件中

先写一个自己需要使用的类
```
public class Student {  
    private String name;  
    private Integer age;
}
```

再把这个类，实现为StudentFactoryBean
```
public class StudentFactoryBean implements FactoryBean<Student> {  
  
    @Override  
    public Student getObject() throws Exception {  
       // new的方式  
//     Student student = new Student();  
  
       // 代理的方式  
       Object o = Proxy.newProxyInstance(Student.class.getClassLoader(), Student.class.getInterfaces(), new InvocationHandler() {  
          @Override  
          public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {  
             // 这里写增强！  
             return null;  
          }  
       });  
       Student student = (Student) o;  
  
       return student;  
    }  
  
    @Override  
    public Class<?> getObjectType() {  
       return Student.class;  
    }  
  
    @Override  
    public boolean isSingleton() {  
       return true;  
    }  
}
```

```
<bean id="javaBean" class="XX.XX.JavaBeanFactoryBean" />
```

id是getObject方法返回的对象的标识
	特殊：工厂bean的ID 是    `&id`    记忆就行！
class是FactoryBean标准化工作类的全限定符

小技巧：这里bean标签下，不能用property标签给对象赋值，因为这个bean标签是工厂类的。要想赋值，要在Factory中也同样声明一个，作为桥梁进行连接

##### 接口方法
##### isSingleton()
返回true false
是否是单例模式
###### getObject()
复杂的实例化过程写在这里！
###### getObjectType()
返回类型

**重点关注**：java中new对象+调用方法的思维方式转变为去Spring的xml配置文件中写bean的方式。

#### 综合案例-总结步骤
1、手写pojo实体类、
2、基于MVC模型，手写dao层、service层、controller层的业务
	思维方式转换：你只管要就行（声明field+setter <-> 读取xml，获取bean）
3、配置xml
	a.druid+jdbcTemplate连接
	b.ioc容器和依赖注入
4、编写Test测试环境
	a. 实例化ioc容器
	b. 获取bean
	c. 调用bean的方法
	d. 关闭ioc容器
==**【经典白学】**==xml方式已经被淘汰了，主要是帮助我们从底层理解ioc容器的使用。

### Spring配置文件 - 注解

#### 加入ioc容器   -->  @Component
==@Component注解适用于类级别；@Bean注解适用于方法级别==
@Component 将自定义类中，如果放在类上，就把这个自定义类的所有方法产生的实例标记为组件bean。 ^68dd31

| 注解          | 备注              | 总结、思考                                                                                                                 |
| ----------- | --------------- | --------------------------------------------------------------------------------------------------------------------- |
| @Component  | 通用组件            |                                                                                                                       |
| @Controller | 表述层的类，用这个注解     | 一般用@RestController = @Controller + @ReponseBody返回JSON数据，不走视图解析器                                                       |
| @Service    | 业务层的类，用这个注解     | 注解加到XxxServiceImpl实现类上，不是加到接口上；                                                                                       |
| @Repository | Mapper层的类，用这个注解 | ==后期用不上==，通过MapperScannerConfigurer工厂方法加入ioc容器中，不需要用@Repository手动加入<br>继承了`extends BaseMapper`的Mapper接口，不用@Repository |
注意：其实这几个都和@Component是一样的，只是区分三层结构而已。

1、在需要的类前面用装饰器加上@Component
	就像当于在xml中建立一个`<bean>`
	id是类名，首字母变小写；class是类的全限定符
	@Component(value="自定义bean id")

2、配置指定包（告诉Spring要去哪里扫描注解
```
<context:component-scan base-package="包1, 包2">
	// 排除特定的包
	<context:exclude-filter type="annotation" expression="类的全限定符"/>

	// 指定包含特定的包
	〈context:exclude-filter ……〉
</context:component-scan>
```

#### 作用域及周期方法
生命周期方法，写在class类文件内部。

##### [[Java_02_面向对象#构造器前置方法 @PostConstruct]]
联想使用场景
![[RabbitMQ#^1f4b7a]]
![[RabbitMQ#^6a5dd3]]

##### @PreDestroy
用于标记在Bean实例销毁之前执行的方法。当容器管理的Bean实例要被销毁时，容器会自动调用被@PreDestroy注解标记的方法。

可以用于关闭数据库连接、释放文件句柄、取消订阅事件等

`@Scope(scopeName = ConfigurableBeanFactory.SCOPE_PROTOTYPE)`          // 加载类前面，或工厂方法前面

#### 依赖注入
##### 引用类型DI  -->  @Autowired 
可以用于构造函数注入、Setter方法注入、==最常用的是加到成员变量上进行注入==。
```
@Component  
public class UserService {  
    @Autowired  
    private UserDao userDao;    //简化理解：把UserDao类的实例拿过来，下面可以用了
}
```

我的理解：@Autowired很重要！当我们需要组件上调用组件的方法时，就需要利用@Autowired注入这个组件。我们就能直接调用对应的API方法了！(***前提，自己的类必须也加入ioc容器***)。
>修正：当我们把组件作为参数传递时，不需要注入，ioc容器会自动帮我们根据数据类型查找。但是

~~因为我们会生成两个ioc容器：WEB容器（子）、root容器（父）。~~
~~1、容器内部的组件可以直接使用。~~
~~2、跨越两个容器时，需要利用@Autowired进行依赖注入，否则获取不到~~
~~3、子容器可以利用@Autowired依赖注入root容器中的组件。root容器不能利用@Autowired依赖注入web容器的组件~~
~~4、mybatis动态代理技术，mapper代理类存放在root容器的子容器中，所以service层和mapper层虽然在同一个root容器，但是也需要@Autowired依赖注入！~~
【更正】上面的总结是🙅错误的，只要加入了容器，都需要依赖注入来使用。除非是形参！

【经典白雪】xml进行依赖注入，利用setter方法名的转换，前提必须要写setter方法，太麻烦了！用@Autowired注解的方式，就不需要特地去写setter()，直接在类的Field字段上注解`@Autowired`就完成了依赖注入！ ^458366

==@Autowired 的意思就是：从ioc容器中找到对应类型实体对象（组件bean），然后自动装配到添加此注解的Field上，你就可以直接在自己的类中调用装配的组件的各种方法了== ^d50302

思考：同一个微服务模块内部，不同层之间（controller -> service -> mapper）使用@Autowired注解注入类实例。不同微服务模块之间（例如service_course的service需要调用service_user的service就不能用@Autowired了，因为他们根本就不在一个ioc容器中，必须使用OpenFeign进行远程调用

**【总结】
1、@Service加到接口实现类上（踏踏实实，有一个加一个）
2、@Autowired 装配接口，不要用接口对应实现类（格局小了！）**
>ChatGPT：一般来说，在引用这个实现类的类中，你可以直接注入这个实现类，而不必非要以接口的形式注入。
>不过，为了更好的可维护性和松耦合性，通常会==建议使用接口来声明依赖，并且使用接口类型来接收实例对象。====这样做可以使代码更易于维护和修改，同时也能够更好地符合面向接口编程的原则。接口的应用使得程序更加灵活，方便进行解耦，特别是在单元测试的时候可以使用mock对象代替实际实现类，提高了代码的可测试性。

一般情况下，ioc容器中的组件都是[[设计模式#Singleton 单例模式]]
特殊情况下，当一个类存在多个实例化对象时，需要配合@Qualifier指定特定的一个

##### @Autowired对比@Resource
1、@Autowired是Spring框架提供；@Resource是JDK提供的；
2、@Autowired默认按类型注入，当一个接口存在多个实现类，需要通过qualifier指定；@Resource（name=""）默认按名称beanName注入；
3、@Autowired可以用在字段、setter方法、构造函数；
@Resource只能用在字段、setter方法上，不能用在构造函数上
4、@Autowired的参数：required和qualifier
@Resource的参数：name、type、shareable等

有倚老卖老的互联网老兵，声称：要用构造方法注入，不要用@Autowired注入，就因为一点小小的特性，Fuck，为了讲而讲吧！
理由是什么呢？
1、这种方法可以声明为final               日常开发谁会用final，用的多吗？
而且必须是一个构造函数，不然spring就不知道用哪个了！还是用@Autowired吧，别听这些老兵混淆视听
```
public class OrderService {
	// 声明一个需要注入类型的Field
	private UserService userService;
	
	// 指定构造器，必须传入userService，完成构造的同时，完成了注入
	public OrderService(UserService userService){
		this.userService = userService;
	}

}
```


#### 懒加载 @Lazy
有些Bean对象，不需要一开始就加载，可以指定它们为懒加载，等需要的时候再实例化！

##### 多例指定  --> @Qualifier("多例类名")
@Autowired是Spring框架提供的，默认是根据类型从容器中寻找（byType)，当一个接口存在多个实现类时（我们又注入的是接口），则需要配合@Qualifier显式指定实现类的名称，才能正确匹配到对应的bean。

**==@Autowired + @Quilfied("多例类名")   =   @Resource(name = "多例类名")==** ^7ed582

假设有一个接口MyInterface，有两个实现类MyInterfaceImpl1和MyInterfaceImpl2，需要在另一个类中注入MyInterface接口的实例。 ^06f0fe
```
public class MyCustomerClass {
	@Autowired
	@Quilfied("myInterfaceImpl1")    //指定注入MyInterface接口的1号实现类！
	private MyInterface myInterface;
}
```
##### 基元类型DI  -->  @value("key")  
一般用来：配置文件中读值。

注意：别导错包了：`org.springframework.beans.factory.annotation`， 不是lombak下的包！
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-05-28%20at%2014.11.47.png)

可以作用到：1、形参列表上、2、Field上
```
1、Field上
public class JavaBean {
	@value(“${jdbc.userpassword}”)     // 用@value注解，用于敏感信息引入
	private int password;
}

2、形参列表上
public DataSource createDatasource(@value("${jdbc.user}") String username, @value("${jdbc.password}") String password,……){

	DruidDataSource dataSource = new DruidDataSource();

	dataSource.setUsername(username);
	dataSource.setPassword(password);
}
```

`@value(“${key:value}”)`  如果没有查到，设置value默认值

注意：第三方类，不能用@Component来注解，因为jar包是只读。
	但是，你可以手写一个类来继承第三方包的类，并在手写的类上加上 @Component 注解来声明为 Spring 的 bean。这种方式通常被称为“包装”或者“适配器”模式，通过这种方式可以将第三方包的类以 Spring bean 的形式来管理和使用。

##### 多例指定 -->  @Resource
来自这个提案：JSR-250
![[#^7ed582]]

如果要用@Resource注解，需要导一个依赖，不在jdk中，需要额外导包！
```
<dependency>
   <groupId>jakarta.annotation</groupId>
   <artifactId>jakarta.annotation-api</artifactId>
   <version>2.1.1</version>
</dependency>
```

多例指定！
```
public class MyCustomerClass {
	@Resource(name = "myInterfaceImpl2")    //指定注入MyInterface接口的1号实现类！
	private MyInterface myInterface;
}
```

### Spring配置文件 - 配置类
全注解！
配置类，可以解决第三方包依赖xml的问题（逐步递进）
#### @Configuration
将本文件声明为配置类！也就是用来创建ioc容器的原料！
##### @ComponentScan("类的全限定符")
Step 1：在类中用全注解来指定：
	1、包扫描注解配置
	2、引用外部的配置文件(jdbc.properties)
	3、声明第三方依赖的bean组件
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/%E6%88%AA%E5%B1%8F2024-03-02%2014.57.26.png)

扫描多个包：`@ComponentScan({"com.atguigu.path1", "com.atguigu.path2", "com.atguigu.path3"})`

标准配置类的示例代码
```
@Configuration()  
@ComponentScan("com.atguigu")  
@PropertySource("classpath:jdbc.properties")  
public class JavaConfig {  
    @Value("${atguigu.url}")  
    private String url;  
    @Value("${atguigu.driver}")  
    private String driver;  
    @Value("${atguigu.username}")  
    private String username;  
    @Value("${atguigu.password}")  
    private String password;  
      
    //druid连接池  
    @Bean  
    public DataSource dataSource(){  
        DruidDataSource dataSource = new DruidDataSource();  
        dataSource.setUrl(url);              // mysql数据库路径  
        dataSource.setDriverClassName(driver);   //驱动，jar包的全限定符  
        dataSource.setUsername(username);                          // 账号  
        dataSource.setPassword(password);                      // 密码  
        return dataSource;  
    }  
      
    //jdbcTemplate  
    @Bean  
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {  
        JdbcTemplate jdbcTemplate = new JdbcTemplate();  
        jdbcTemplate.setDataSource(dataSource);  
        return jdbcTemplate;  
    }  
}
```

^0b84ec

Step 2：在测试类中，创建ioc容器；并获取bean实例
（注意不是用之前那个：ClassPathXmlApplicationContext，这是读取xml文件的），配置类用：AnnotationConfigApplicationContext(配置类.class)

| AnnotationConfigApplicationContext | 通过读取Java类（作为配置文件）创建IoC容器对象 |
| ---- | ---- |
方式1：全自动
```
AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(JavaConfiguration.class);
```

方式2:手动
```
AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();

applicationContext.register(JavaConfiguration.class);

applicationContext.refresh();
```


思考：@ComponentScan相当于是扫描器，@Component相当于是标记！
扫描器  --->  Java配置类中
标记     ---->  实现类中

#### @Bean
==@Component注解适用于类级别；@Bean注解适用于（配置类的）方法级别==

@Bean 注解通常用于标记一个方法，该方法返回一个由 Spring 容器托管的 bean 实例。
写在配置类中，其他地方用不到！（不是写在主启动类中！） ^2f2f58

有些情况下，我们需要从第三方模块调用方法，获取到一个对象，将这个对象加入我们需要使用的ioc容器中，就需要使用@Bean，和我们自定义类的实例化对象加入ioc容器是不同的。类模版在人家第三方库那边。
>例如：在配置类中实例化第三方依赖的对象加到ioc容器中。（例如Druid连接池）
>例如：SpringCloud中，将RestTemplate服务调用和负载均衡，加入ioc容器，也是创建一个TestTemplateConfig配置类，然后return new loadbalancer() @Bean和@LoadBalancer

@Bean(initMethod = "init")
	参数1:name = "重命名bean id"
	参数2：initMethod、destroyMethod指定周期方法

##### 初始方法 initMethod
在xml中指定在==Bean实例化完成==后，调用的initMethod方法，留给我们最后的机会去给属性赋值（initMethod、destroyMethod）
一般用于：
	1、第三方依赖包加入ioc容器！（最常用）
	2、需要进行特殊的定制化配置（initMethod、destroyMethod）
	3、根据特定条件创建多个实例（scope）

本方法中会判断是否实现了 [[#InitializingBean接口]]，如果实现了的话，先走接口中的方法，再走initMethod方法

##### 作用域 scope
作用域 @Scope(scopeName = ConfigurableBeanFactory.SCOPE_SINGLETON)

第三方依赖声明为bean组件：
```
@Bean
public RequestMappingHandleMapping handlerMapping() {
	return new RequestMappingHandleMapping();
}
```

第三方包的依赖注入方式（dataSource --注入--> jdbcTemplate）
```
@Bean  
public JdbcTemplate jdbcTemplate() {  
    JdbcTemplate jdbcTemplate = new JdbcTemplate();  
    //方案1:如果所需的组件也是@Bean方法，可以直接调用  
    jdbcTemplate.setDataSource(dataSource());  //dataSource()看上去是方法，其实就是那个Bean  
    return jdbcTemplate;  
}  
  
@Bean  
public JdbcTemplate jdbcTemplate1(DataSource dataSource) {  
    JdbcTemplate jdbcTemplate1 = new JdbcTemplate();  
    //方案2:形参列表想要声明的组件类型，可以多个，ioc容器也会注入  
    //如果同一个类生成多个Bean实例，需要形参=bean id  
    jdbcTemplate1.setDataSource(dataSource());  
    return jdbcTemplate1;  
}
```

#### @import
将多个配置类，汇总到一个中，之后在实例化ioc容器时，读一个就行了！
```
@Configuration
public class ConfigA {
	@Bean
	public A a() {
		return new A();
	}
}

@Configuration
@Import(ConfigA.class)   //将配置类B导入配置类A
public class ConfigB {
	@Bean
	public B b() {
		return new B();
	}
}
```

#### 项目文件结构
```
├── Module
	├── config
		├── JavaConfig         # 配置类
	├── pojo
		├── Student               # 实体类
	├── dao
		├── StudentDao            # Dao接口
		├── impl
			├── StudentDaoImpl    # 实现类
	├── service
		├── StudentService        # Service接口
		├── impl
			├── StudentDaoImpl    # 实现类
	├── controller
		├── StudentController    # Controller类	
├── resources
	├── jdbc.properties          # mysql账号密码
```

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B43%E6%9C%885%E6%97%A5%2010_14_47.jpg)
### Spring 单元测试
简化了，我们每一次在测试类中，启动实例化一个ioc容器，获取bean，使用bean的过程。

之前在测试类中，我们都要new ApplicationContext，并把配置类传进去，来实例化一个容器。

导入依赖包:spring-test
```
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-test</artifactId>  
    <version>6.0.6</version>  
    <scope>test</scope>
</dependency>
```

#### @SpringJUnitConfig()
利用注解来简化new ApplicationContext来创建ioc容器！
注意：@SpringBootTest 只在SpringBoot环境下生效；
但是@SpringJUnitConfig适用于所有Spring FrameWork框架的单元测试环境！

属性
	1、locations = 指定xml配置文件
	2、value = 指定配置类    JavaConfig.class

```
@SpringJUnitConfig(value = JavaConfiguration.class)  
public class IocAnnotationTest {  
  
    @Autowired  
    private StudentController controller;

	@Test  
    public void show2() {  
        controller.findAll();  
    }
```

#### JUnit
![[标准包.类#^28cb08]]

junit5 的测试方法
```
@SpringBootTest
public class JwtTest {
    // Field，依赖注入
    @Autowired
    private JwtHelper jwtHelper;

    @Test
    public void test01(){
```

junit 4的测试方法
```
@SpringBootTest
@RunWith(SpringRunner.class)
public class GulimallProductApplicationTests {
	//测试方法
}
```

### IoC源码分析
`OneDrive\文档\02-学习\Learn_Java_Mosh\java_第三方库_关系图.excalidraw`

深入思考这些问题：
Q：说明Bean的生命周期
A：首先实例化一个BeanFactory的实现类，作为IoC容器，一般是DefaultListableBeanFactory，然后基于BeanFactory，从xml配置文件或注解中利用ResourceReader读取到信息，生成BeanDefinition对象（定义Bean的对象），完成BeanDefinition的实例化和初始化工作。然后从BD中获取到所有beanName的列表，利用for循环每一个beanName，执行基于BeanDefiniton生成Bean对象的操作（主要是利用反射的方式），每个生成的Bean对象就都加入IoC容器中提供使用，接着销毁。
从源码角度，主要是BeanFactory中调用refresh()方法开始，执行一条总共6个主要方法的调用链完成Bean对象的创建工作：getBean(beanName) -> doGetBean() -> createBean() -> doCreateBean() -> createBeanInstance -> populateBean() ^ad8461

Q：说一下循环依赖和三级缓存
A：循环依赖是指A对象中有一个B类型的属性，B对象有一个A类型的属性。这种情况下，就形成个了循环依赖。
Spring框架中利用三级缓存机制解决循环依赖问题。总的来说，就是利用实例化 和 初始化分离的思想，当A对象已实例化，但是未进行初始化时的半成品A放入缓存中，并继续指向B对象的实例化，B对象也同样将半成品放入缓存中。然后提取缓存中的A赋给B对象，同时将B也赋给A对象，就能解决循环依赖的问题。
从源码角度，三个级别的缓存都是Map集合，一级缓存叫SingletonObjects、二级缓存叫earlySingletonObjects、三级缓存在singletonFactories，key是BeanName，value分别是：一级缓存中是成品对象、二级缓存中是半成品对象、三级缓存中是lambda表达式

Q：FactoryBean和BeanFactory的区别
A：BeanFactory是IoC容器的顶层接口，在Spring源码注释中有一句话很好地解释了这一点“The root interface for access the bean container。可以理解为是IoC容器的入口，另外，它下面有一个实现类DefaultListableBeanFactory就是我们常说的IoC容器本身。

`FactoryBean<T>`：如果说BeanFactory接口作为IoC容器，定义了一套标准的创建Bean的流程（这部分我阅读过Spring的源码，主要是refresh()方法）；那么FactoryBean可以理解为是提供给第三方框架加入IoC容器的一个定制化的接口。第三方框架通过需要这三个的方法：isSingleton、getObjectType、getObject。geyObject()方法返回的对象就会加入IoC容器（背后的逻辑是：在标准化流程创建bean之前，有一个isFactoryBean的判断，如果实现了FactoryBean会走单独的一套创建Bean的流程，将getObject()返回的对象加入IoC容器！

ApplicationContext和BeanFactory的区别
BeanFactory是父接口，ApplicationContext是其子接口。
BeanFactory接口是IoC容器的顶层接口，也是进入容器的入口。其子类DefaultListableBeanFactory实现类就是IoC容器！
ApplicationContext是在BeanFactory接口基础上进行了功能的完善，支持更多调用方法。

设计模式

Aware接口的作用：
当我们在自定义的类中，需要引用IoC容器中自带的一些Bean时（例如ApplicationContext)，由于Spring体现了控制反转，所有我们不用实例化和调用，那如何才能让IoC容器帮我们设置applicationContext这个属性值呢？通过实现Aware接口的子接口，就能自动重写对应setApplicationContext方法，来实现容器对象的属性赋值！

Aware接口和Synchronized接口一样，只有一个空方法！起一个标识作用！
```
public interface Aware {  

}
```
#### BeanDefinition和Bean
![CleanShot 2024-07-25 at 23.57.42@2x.png](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-07-25%20at%2023.57.42%402x.png)

不管是：
	xml方式（[[#使用xml加入ioc容器 --> `<bean>`]]）
	还是注解方式（[[#加入ioc容器 --> @Component]]）
	都是Bean的定义信息，通过工具类（[[JavaWeb#DOM4J解析xml]]）解析就能对应到BeanDefination对象！最终都会转成一个beanDefination对象
	利用BeanDefination对象，创建出具体对象bean！

- BeanDefination：从xml或注解中读取配置信息，创建出beanDefination对象（有点类似于类模板，这是豆模板）
	使用了其中的[[Java_02_面向对象#4、反射 -> getInstance()]]

- [[#BeanFactory接口]]：可以理解为访问IoC容器对象的入口，或者IoC容器对象。
- BeanFactory -> ==DefaultListableBeanFactory==实现类：就是实际存放Bean的容器本器。

利用BeanFactory接口中规约的方法就可以拿到其中的Bean对象
```
源码的注释原文
The root interface for accessing a Spring bean container.
```

方式1：利用beanDefination对象，获取指定一个类型的Bean
```
BeanDefination peron = beanFactory.getBeanDefination(beanName:"person");

person.setName = xxx;
```

- [[#ApplicationContext接口]] ：继承了BeanFactory接口，提供一系列更加完善的功能。

方式2：利用applicationContext对象，获取指定类型的Bean
```
ApplicationContext person = context.getBean(Person.class);
```

#### BeanFactoryPostProcessor 和 BeanPostProcessor
BeanFactory对象完成实例化之后、【初始化阶段】完成了属性赋值，在调用初始化方法之前，进行增强的类！

Bean对象完成实例化之后、【初始化阶段】完成了属性赋值，在调用初始化方法之前，进行增强的类！


#### 依赖注入

 #### Aware接口
 为 bean 提供了访问 Spring 容器内部服务的能力,使 bean 能够更好地集成到 Spring 应用程序中。
 一般用在二次开发！
 
  作用
 * 当我们在自定义的类中，需要引用IoC容器中自带的一些Bean时（例如ApplicationContext)，  
 * 由于Spring体现了控制反转，所有我们不用实例化和调用，  
 * 那如何才能让IoC容器帮我们设置applicationContext这个属性值呢？  
 * 通过实现Aware接口的子接口，就能自动重写对应setApplicationContext方法，来实现容器对象的属性赋值！  
是空的，起一个标识作用！联想到Synchronized

常用的子类
1. **BeanNameAware**：实现这个接口的 bean 可以获取到自己在 Spring 容器中的 bean 名称。
2. **BeanFactoryAware**：实现这个接口的 bean 可以获取到创建自己的 BeanFactory 实例。
3. **ApplicationContextAware**：实现这个接口的 bean 可以获取到创建自己的 ApplicationContext 实例。
6. **ResourceLoaderAware**：实现这个接口的 bean 可以获取到 ApplicationContext 中的 ResourceLoader 实例,用于加载资源文件。
7. **EnvironmentAware**：实现这个接口的 bean 可以获取到 ApplicationContext 中的 Environment 实例,用于获取环境变量或系统属性。

#### 循环依赖
三级缓存 + 提前暴露对象 （实例化与初始化分开操作）解决循环依赖！
![CleanShot 2024-07-25 at 23.51.57@2x.png](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-07-25%20at%2023.51.57%402x.png)

![CleanShot 2024-07-25 at 23.53.11@2x.png](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-07-25%20at%2023.53.11%402x.png)

[[Java_02_面向对象#实例化与初始化的区别]]

源码中是三个Map
```
一级缓存：Map<String, Object> singletonObjects     ConcurrentHashMap<>(256)

二级缓存：Map<String, Object> earlySingletonObjects   ConcurrentHashMap<>(16)

三级缓存：Map<String, ObjectFactory<?>> singletonFactories    ConcurrentHashMap<>(16)
```

Q：三个缓存对象，在获取数据的时候，顺序是什么？
A：一级缓存（成品） -> 二级缓存（半成品） -> 三级缓存（lambda)
![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202407261536729.png)

Q：如果只有一级缓存，能解决循环依赖问题吗？
A：不能，有成品和半成品对象，beanName会冲突，所以需要两个缓存结构存放他们：一级缓存放成品；二级缓存放半成品

Q：如果只有一级和二级缓存，不用三级缓存可以吗？
A：当存在AOP时，就会出现问题，必须使用三级缓存。
```
This means that said other beans do not use the final verison of the bean.

# 其实就是 成品A 和 半成品A 的区别！
```
那为什么AOP，为什么就必须使用三级缓存呢？
一个beanName对应两个对象（原始对象，代理对象），
![[#^8ada82]]



## AOP 面向切面编程
### AOP概念
[[编程范式#面向切面编程 AOP]]
![[编程范式#8个核心概念]]
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/fa46cd18c71cca029dee3a465c249448.JPG)

![[编程范式#Interceptor 拦截器]]

```
try{
	前置
	目标方法执行代码
	后置
}catch{
	异常
}finally{
	最后
}
```
### 代理
[[编程范式#代理]]

### Spring整合AOP
[Spring FrameWork中AOP的参考文档](https://docs.spring.io/spring-framework/reference/core/aop.html)
Spring容器中使用AOP非常简单，只需要定义执行的Advice增强方法，并用@Aspect注解标注应该在何处触发执行。Spring通过CGLIB动态代理创建子类等当时实现AOP代理模式；通过JDK动态带来里创建接口类的方式实现AOP代理模式。

==注意：实际加入Spring IOC容器的是代理类的对象，不是目标类的对象！==

Spring AOP= AspectJ注解层 + JDK + CGLIB
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/8019f05e5b615a2fa247930f112c0099.JPG)


1、不需要导`spring-aop`的包，已经含在Spring-context包里面！
```
<dependency>
   <groupId>org.springframework</groupId>
   <artifactId>spring-aop</artifactId>
   <version>6.1.9</version>
</dependency>
```

2、==需要导`spring-aspectj`这个包==，由它提供Aspectj注解：@Before、@After、@Around都出自这个包
```
<dependency>
   <groupId>org.springframework</groupId>
   <artifactId>spring-aspects</artifactId>
   <version>6.1.9</version>
</dependency>
```

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/%E6%88%AA%E5%B1%8F2024-03-03%2017.34.15.png)
#### 使用步骤
##### 1、定义XxxAspect类
定义XxxAspect类及其中的Advice增强方法（写代码），具体定义几个方法，根据插入的位置决定！
并加注解：@Component  @Aspect

##### 2、Advice增强方法上加@Before
在其中的增强方法上使用[[编程范式#Advice]]指定插入目标方法的位置（@Before、@After……）

##### 3、@Before(execution=“”)中加入切点表达式
在@Before等注解中：配置切点表达式。选中插入的类和方法【也就是：切点】
相当于告诉这个Advice增强方法作用在哪里
	`@Before("execution(* com.atguigu.service.impl.*.*(..))")``
	==*小技巧：先写@Aspect，再写execution，会自动提示*==
```
@Aspect  
@Component  
public class LoggingAspect {
	// 在执行UserService的每个方法前执行:  
	@Before("execution(public * com.itranswarp.learnjava.service.UserService.*(..))")  
	public void doAccessCheck() {  
	System.err.println("[Before] do access check...");  
	}  
  
	// 在执行MailService的每个方法前后执行:  
	@Around("execution(public * com.itranswarp.learnjava.service.MailService.*(..))")  
	public Object doLogging(ProceedingJoinPoint pjp) throws Throwable {  
	System.err.println("[Around] start " + pjp.getSignature());  
	Object retVal = pjp.proceed();  
	System.err.println("[Around] done " + pjp.getSignature());  
	return retVal;  
	}  
}
```

##### 4、主启动或配置类上开启@EnableAspectJAutoProxy
4、主启动或配置类上开启Aspect，配置类开启：`@EnableAspectJAutoProxy` ^e51f84
```
@Configuration  
@ComponentScan  
@EnableAspectJAutoProxy   # 开启AOP
public class AppConfig {  
	...  
}
```
#### 接值
在测试场景中，利用Field接值的注意点！
AOP检测到接口，默认使用jdk动态代理（拜把子），代理对象和目标对象是拜把子兄弟，必须用接口接值！
如果没有接口，默认使用CGLIB动态带来（认干爹），代理对象继承自目标对象，接值！
```
@SpringJUnitConfig(value = JavaConfiguration.class)  
public class SpringAopTest {  
    @Autowired  
    private Calculator calculator;  
    /*  
    
    如果有了AOP技术，在ioc容器中存的其实是代理对象，而不是目标对象！  
     */  
    @Test  
    public void test() {  
        int add = calculator.add(1, 2);  
        System.out.println("add = " + add);  
    }  
}
```

#### 获取切点详细信息 JointPoint
在增强方法中获取目标方法信息:
##### 目标方法的类的信息
方法名、参数、访问修饰符、所属类的信息

超级简单，在增强方法的形参加入`JointPoint jointPoint`，这个对象的实例`jointPoint`就包含目标方法的所有信息!
```
@Component  
@Aspect  
public class MyAdvice {  
    @Before("execution(* com.atguigu.service.impl.*.*(..))")  
    public void before(JoinPoint joinPoint) {
		//1.获取方法所属类的信息  
		String simpleName = joinPoint.getTarget().getClass().getSimpleName(); 
  
		//2.获取方法名  
		String name = joinPoint.getSignature().getName();  
  
		//3.获取参数列表  
		Object[] args = joinPoint.getArgs();
    }
```
##### 目标方法的返回值 returning
当我在返回增强的代码中想要那个返回值时，需要用到！只在@AfterReturning生效
1、在增强方法的形参里添加一个Object result，代表要用result这个变量来接收从目标方法传过来的返回值。
2、在@AfterReturning注解中加一个returning属性，和第一步的result变量关联起来
```
@AfterReturning(value = "execution(* com.atguigu.service.impl.*.*(..))",returning = "result")   // 正常结束会走的  
public void afterReturning(Object result) {}
```
##### 异常对象的信息 throwing
只在@AfterThrowing生效，操作同上
1、形参加Throwable throwable
2、注解加Throwing属性，throwable
```
@AfterThrowing(value = "execution(* com.atguigu.service.impl.*.*(..))", throwing = "throwable")   // 异常时走的  
public void afterThrowing(Throwable throwable) {}
```

#### 切点表达式
作用是定义增强方法，应该插入到哪个类的哪个方法中！是一种类似正则表达式的查询定位语句

execution(1 2 3.4.5(6))
1、访问修饰符
2、返回值类型
3、包的位置
	具体包：com.atguigu.service.impl
	单层模糊：com.atguigu.service.*
	多层模糊：com..impl
	细节  ..不能开头
4、类的名称
	具体类名：CalculatorPureImpl
	模糊：*
	部分模糊：`*Impl`
5、方法名
6、形参列表（同一个方法的重载）

切点表达式的提取和复用
方式1：提取到`public void pointcut()`方法中
1、提取切点表达式，用一个随意命名的方法承载，加上注解@PointCut(提取的切点表达)
2、在增强注解中引用切点表达式的方法
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/%E6%88%AA%E5%B1%8F2024-03-03%2021.50.07.png)

【推荐】方式2:提取到存储切点表达式的增强类，单独维护
注意：一定也要@Component，一定要被扫描到
```
/*  
统一管理切点表达式的增强类  
 */  
 
@Component  
public class MyPointCut {  
    @Pointcut("execution(* com.atguigu.service.impl.*.*(..))")  
    public void pc1(){}  
  
    @Pointcut("execution(* com..impl.*.*(..))")  
    public void pc2(){}  
}
```

#### 环绕增强
自定义的所有的增强方法！方法里维护一个try catch finally结构！
```
@Component  
public class TxAdvice {  
    //ProceedingJointPoint类型，除了获取目标类的全部信息，还多了一个执行方法  
  
    @Around("com.atguigu.pointcut.MyPointCut.pc1()")  
    public Object transaction(ProceedingJoinPoint joinPoint) {  
  
        // 首先保证目标方法被执行  
        Object[] args = joinPoint.getArgs();  
        Object result = null;  
  
        try {  
            // 执行前，前置增强  @Before            System.out.println("开启事务");  
            result = joinPoint.proceed(args);  
            // 成功执行后， @AfterReutrning            System.out.println("结束事务");  
        } catch (Throwable e) {  
            // 异常增强  @AfterThrowing            System.out.println("事务回滚");  
            throw new RuntimeException(e);  
        }finally {  
            // @After  
        }  
        return result;  
    }  
}
```


#### 切面优先级 @Order(value)
多个增强类（事务、日志等），哪个先执行，需要通过切面优先级来定义！
- 值越小，优先级越高，优先级越高  ->  前置越先执行，后置越后执行！


### AOP源码分析

#### AOP实现原理



#### JDK动态代理的实现原理



#### Spring-AOP如何生效


## tx 声明式事务
事务的概念来自MySQL：
![[mysql.DML#^87e58f]]

**==一个方法中同时修改两个表时，需要开启事务==** ^3821da

联想到：MySQL中的事务要求：![[mysql.DML#^a6e81f]

联想到[[mysql.DML#MySql -事务]] 、 [[redis#事务]]  、 [[redis#redis事务与mysql事务对比]]

Q：Spring中哪些实现事务的方式？
A：编程式事务、声明式事务

编程式事务，需要手动在每个mysql操作前后加这样结构的代码，大量冗余 ^8ae14e
```
Connection conn = ……;

try {
	conn.setAutoCommit(false);
	// 核心操作
	// 业务代码
	// 提交事务
	conn.commit();
}catch(Exception e) {
	// 回滚事务
	conn.rollback();
}finally {
	conn.close();
}
```

^01e2e3

[[#TransactionTemplate]]
### 事务管理器 - TransactionManagement
事务管理器，主要接收dataSource，控制住数据库连接的！

声明式事务，就是将事务处理逻辑封装到一个事务框架中，我们直接利用注解使用即可。
	联想到[[编程范式#声明式编程]]  -> 只关注我们想要得到什么结果，不关注怎么做

spring-tx  这个第三方包，就是用来使用事务。
	这个第三方包，使用了AOP进一步封装！
![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202408050938976.png)

### 编程式事务
#### 基本步骤
1、配置类中，生成DataSourceTransactionManageer对象，注入IoC容器
一定要用PlatformTransactionManager接口类型，而不要用TransactionManager接口类型！
```
/**  
 * 编程式事务的配置类，一定要用PlatformTransactionManager类型，而不要用TransactionManager  
 * TransactionManager接口是荣誉接口，根本没有任何方法，得用PlatformTransactionManager接口，里面才有getTransaction(new DefaultTransacitonDefinition())  
 */

@Configuration
public class TransactionConfig {
	// dataSource对象略

	@Bean  
	public PlatformTransactionManager transactionManager(DataSource dataSource) {  
	    // 内部要基于连接池进行mysql的操作，所以需要连接池对象  
	    DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager();  
	    dataSourceTransactionManager.setDataSource(dataSource);  
	    return dataSourceTransactionManager;  
	}
```

2、业务类中注入PlatformTransactionManager对象，并生成transaction对象（事务对象，基于此对象操作事务的提交和回滚）
```
    @Autowired  
    private PlatformTransactionManager dataSourceTransactionManager;  
      
    public void performTransaction(String data){  
        TransactionStatus transaction = dataSourceTransactionManager.getTransaction(new DefaultTransactionDefinition());  
        try {  
            //执行事务的地方，一组mysql  
//             jdbcTemplate.update("sql",data);  
            // 提交事务  
            dataSourceTransactionManager.commit(transaction);  
        }catch(RuntimeException e) {  
            //失败  
            dataSourceTransactionManager.rollback(transaction);  
            throw e;  
        }  
    }
```

#### 利用[[#编程式事务 TransactionTemplate]]工具类实现编程式事务
相见对应章节

### 声明式事务 @Transactional
#### 基本步骤
Step 1：在配置类中：事务管理器加入ioc容器
	1、选择对应事务管理器TransactionManager实现加入到ioc容器；
	2、开启事务的注解    `@EnableTransactionManagement //开启tx`
```
@Configuration  
@ComponentScan("com.atguigu.service")  
@EnableTransactionManagement  
public class ServiceJavaConfig {  
  
    @Bean  
    public TransactionManager transactionManager(DataSource dataSource) {  
        DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager();  
        dataSourceTransactionManager.setDataSource(dataSource);  
        return dataSourceTransactionManager;  
    }  
}
```

^d607a3

Step 2：使用注解指定哪些方法需要添加事务即可  --> 声明式事务 @Transactional
	1、方法上：当前方法会有事务
	2、类上：类下所有方法都有事务  【常用】 ^151dab
	
这个注解，相当于标记哪些方法需要使用事务！
利用AOP面向切面编程 + 动态代理技术，在标注了@Transactional注解的方法前后执行事务！

只能加在public修饰的方法上，其余protect、private都不行。因为动态代理技术只能代理公开的方法！

#### 常用参数
##### readonly
只读模式
可以提升查询效率，默认是false    `@Transactional(readonly = true)`

Q：查询语句又不需要添加事务，这个有什么用？
A：一般我们都是在类上添加事务，类下所有方法都生效，当某个特定方法是查询相关操作时，可以写设置为readonly=true，覆盖父类上的设置！（秒！

##### timeout
给事务设置一个超时时间，到时间未完成，事务回滚，并放出异常！  单位是秒！
`@Transactional(timeout = 60)`      超过60秒，就会回滚事务

异常为：TransactionTimeoutException

注意：在类上加了timeout，方法上默认timeout=-1，方法是不生效的！

##### rollbackFor
指定异常回滚 
`@Transactional(rollbackFor = Exception.class)`          建议改为这个！避免漏网之鱼

事务异常 
默认情况下，指定发生运行时异常（Runtime Exception）才会回滚。我们想要修改为所有Exception异常时都回滚，避免发生IOException时，数据出错！
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B43%E6%9C%884%E6%97%A5%2015_07_47.jpg)

##### ~~noRollbackFor~~
指定异常不回滚 
排除声明的那个异常    没什么应用场景！

##### isolation
事务隔离级别
[[mysql.DML#事务隔离级别]]

多个事务并发执行时，避免事务之间相互影响。

| 隔离级别                |                                          |                          |
| ------------------- | ---------------------------------------- | ------------------------ |
| DEFAULT             | 默认使用mysql的隔离级别；<br>mysql是可重复读；oracle是读已条 |                          |
| READ_UNCOMMITTED    | 读未提交     可以读到未被提交的数据，最低级别                | 很糟糕<br>会发生脏读、不可重复读、幻读的问题 |
| READ_COMMITED       | 读已提交      能读到已被提交的数据                     |                          |
| ==REPEATABLE_READ== | 可重复读                                     | mysql默认隔离级别，☑️           |
| SERIALIZABLE        | 所有事务串行执行                                 |                          |
 
推荐事务隔离级别为： 可重复读

`@Transactional(isolation = Isolation.REPEATABLE_READ)`  设置为：读已提交

##### propagation
设置到==子方法上==

事务的传播行为，是指：事务所在方法之间调用，如何影响子事务！

注意：方法1和方法2一定是在==不同的类中才生效==（同一个类中的父子方法调用，传播行为不生效），方法1所在的类，被注入到方法2所在类，并在方法2的类中：类1.方法1调用。

![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202408051031911.png)

推荐默认！因为既然被调用了，证明我们是一起的。
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/34d0894020915eaf2faddfe58457598e.PNG)
这些事务传播机制是借助[[标准包.类#ThreadLocal类]]实现的。

#TODO 事务的传播级别，理解的不是特别好，先记录并搁置。
2024.8.5  回炉从造，对@Transactional事务有了比较好的理解，也补完了之前@阳哥直播时指出的一个点：@Transactional注解存在的坑！

| propagation      | XX                                                         |
| ---------------- | ---------------------------------------------------------- |
| ==REQUIRED(默认)== | 如果父方法有事务，就加入，变成一个独立的事务；<br>如果父方法没有事务，子方法就新建一个事务；           |
| REQUIRES_NEW     | 不管父方法是否有事务，本方法都新建一个独立的事务                                   |
| NESTED           | 如果当前父方法中存在事务，则在该事务中嵌套一个新事务；<br>如果没有事务，则新建一个事务（和required一样） |
| SUPPORTS         | 如果当前存在事务，则加入事务；<br>如果当前不存在事务，以非事务方式执行                      |
| NOT_SUPPORTED    | 以非事务的方式执行，若当前父方法存在事务，则把当前事务挂起                              |
| MANDATORY        | n. 强制性<br>必须在一个已有的事务中执行，否则抛异常                              |
| NEVER            | 本方法必须没有事务的情况下执行，否则报错                                       |

### 【大坑🌋】声明式事务失效场景
[【spring事务失效有哪些场景啊-哔哩哔哩】]( https://b23.tv/Buz9zCt)   这位up主讲的很好！

注意：@Transactional注解有一个大坑，一定要注意：
1、同一个类中不同方法上，加@Transactional是失效的！因为不走AOP代理增强
解决思路，将xxxService注入后，用这个service调用，而避免用this调用，会发现cglib对象（是继承的动态代理对象）

2、private、final修饰的方法，加上@Transactional注解也会失效，因为是基于CGLIB继承的方式动态代理，private方法无法被增强！

3、异步线程中，事务与连接绑定，与主线程对数据库的连接不是同一个，也不能保证事务！

4、异常被吃掉！ 主要是嵌套事务的问题，内层子方法如果异常被try…catch捕获了，外层以为岁月静好，就把事务给提交了，那不是完犊子了！

5、[在@Transactional注解修饰的方法中使用线程锁，可能遇到锁实效问题。](https://mp.weixin.qq.com/s/Fp0T5s8s2MFfuqQbDii5bw)
在下例中：在方法内部加锁，但锁的释放是在事务提交之前。如果在锁释放后，事务提交前，有其他线程进入尝试更新同一个数据，就可能发生[[mysql.DML#Dirty Reads 脏数据]]。
注意：事务管理和锁操作分离！
```
@Service  
public class UpdateService {  
   private final Lock lock = new ReentrantLock();  

   @Transactional  
   public void updateData() {  
       lock.lock();  
       try {  
           // 模拟数据库更新操作，耗时操作
      } catch (Exception e) {  
           Thread.currentThread().interrupt();  
      } finally {  
           lock.unlock();  
      }  
  }  
}
```

加了@Transactional注解的方法中一定要避免RPC等耗时操作，导致大事务长时间占据数据库连接，大坑！！

【强制】事务场景中，抛出异常被catch后，如果需要回滚，一定要注意手动回顾事务。 #Java编程规约  ^7052d2

【参考】@Transactional事务不要滥用，事务会印象数据库的QPS，另外使用事务的地方需要考虑各方面的回滚方案，包括缓存回滚、搜索引擎回滚、消息补偿、统计修正等。 #Java编程规约 

声明式事务只能加到方法上（颗粒度大），如果一个方法包含读和写操作，读操作用不到事务，但是包含在事务中，就会造成长事务的问题，导致数据库连接无法释放，影响性能。


*************************************
补充记录
## Template类
### JdbcTemplate
由Spring Framework提供的模板类，简化JDBC操作
联想到 [[JDBC#Druid连接池 + JdbcTemplate类]]
Spring提供的“写好的DAO接口的实现类”，直接实例化一个对象，调用方法即可。
同类的：DbUtils （Apache Commons依赖包中）

Q：JDBC和JDBCTemplate的关联和区别是什么？
A：
- JDBC是Sun公司定义的一套规范，是接口，各数据库尝试提供的驱动就是接口的实现类。 ^6a3f1c
- JdbcTemplate是接口的实现类！

使用步骤：
![[JDBC#^4b07ab]]

```
JdbcTemplate jdbcTemplate = new JdbcTemplate();

//Druid连接池建立连接
jdbdTemplate.setDataSource(一个Druid连接池对象)

jdbcTemplate.update();            // 更新数据

jdbcTemplate.queryforObject();    // 查询单个对象
jdbcTemplate.query;               // 查询多个对象，集合
```

### RedisTemplate
`org.springframework.data.redis.core.RedisTemplate`
Java中操作[[redis]]，主要就是用它。与Spring高度整合，是线程安全的，支持多种序列化策略
	也可以用：[[标准包.类#Jedis]]，更加轻量级，简单
	也可以用：[[标准包.类#Redisson]]

RedisTemplate.opsForValue() 一般都返回ValueOperations接口，在这个接口上执行操作redis的方法。

后端训练营-用户抽奖项目，将RedisTemplate常用的方法封装进了RedisUtil类中，增加了异常处理。更方便使用。我上传到了我的github上，需要的话就下载下来，放入自己项目中！

```
@Autowired  
private RedisTemplate<String, Object> redisTemplate;

redisTemplate.opsForValue().XXX
```
##### redisTemplate.execute(lua脚本)
```
List<Object> result = redisTemplate.execute(new DefaultRedisScript<>(redisScript, List.class), keys, args);
```


##### redisTemplate.opsForValue().常用方法
opsForValue主要是普通key-value的操作方法

###### set(String key, Object value)
普通缓存放入，key-value
###### get(String key)
普通缓存获取

###### 【重载】set(String key, long time, TimeUnit.SECONDS)
普通缓存并设置过期时间
time，单位是秒，小于等于0，表示无限期

###### expire(String key, long time, TimeUnit.SECONDS)
指定这个key的过期失效时间，单位秒

###### hasKey(String key)
判断key是否存在

###### delete(String key)
可以传一个或多个key

###### setIfAbsent(key, value)
就是setnx这个命令   [[redis#SETNX key value]]
仅在键不存在时，设置键的值。
redis分布式锁

###### 【重载】setnx(String key, Object value, long time, TimeUnit.SECONDS)
指定过期时间
###### increment(key, delta)
对存储在指定键的数值执行递增操作

##### redisTemplate.opsForHash().常用方法
opsFoeHash主要是Hash这种数据结构的操作方法
###### get(key, item)
获取指定key的Hash表中，item这一行的value
###### entries(String key)
获取该key的HashTable中所有的field-value对
###### putAll(String key, Map<String, Object> map)
指定key，批量将Map中多个field-value加入
###### put(String key, String item, Object value)
指定key的HashTable，加入一条数据

###### delete(String key, Object item)
删除key对应的Hash表中，item所在行

###### hasKey(String key, String item)
判断key对应Hash表中，item这一行，有没有value

###### increment(String key, String item, double num)
key对应的Hash表中，item所在行的value，递增！

##### redisTemplate.opsForSet().常用方法
针对redis中Set数据结构的操作方法
###### members(String key)
因为key是不能重复的，返回redis的Hash中的所有key，装在set中，返回一个set

###### isMember(String key, Object value)
根据value，查询一下有没有这个value对应的key

###### add(String key, Objects values)
key对应一个Set集合（不允许重复）将多个value加入其中（重复的将被移除），返回成功加入set的个数
###### size(String key)
返回key对应的Set集合中的个数

###### remove(String key, Object... values)
key对应的这个Set集合，删除多个value

###### pop(key)
key对应的这个Set集合，随机移除一个元素

##### redisTemplate.opsForList().常用方法
针对redis中List数据结构的操作方法

###### size(String key)
获取该key对应的列表的长度

###### index(String key, long index)
通过索引，获取list中的值

###### rightPush(String key, Object value)
加入列表的队尾

###### remove(String key, long count, Object value)
这个key对应的列表，移除count个数的值

*********************
###### multiSet(Map<String, String> map)
设置多个键值对
###### multiGet(`Collection<String>` keys)
传一个字符串类型的集合，基于这个集合中的key,获取对应value

###### getAndSet(key, value)
设置指定key的值，并返回之前的旧值

### ~~RestTemplate~~
由Spring Framework提供的模板类，REST请求（GET、POST、DELETE）时，两个微服务之间相互调用的工具类！ 

重要的总结和思考：两个模块之间，A想要使用B中的某个方法，应该怎么办？这种场景很常见，例如Gateway模块中一般是很纯净的，不引入Mybatis等数据库操作的依赖的，但是这时如果需要在Gateway层面进行统一业务处理，例如接口调用次数+1，就涉及数据库写入，有哪些方案？

| 方式                                  | 弊端                                    |                                                                           |
| ----------------------------------- | ------------------------------------- | ------------------------------------------------------------------------- |
| A项目打成Jar包，引入B项目                     | 代码复制两份，万一要改，两头都要改                     |                                                                           |
| Http方式：A项目暴露一个接口，<br>B项目利用Http请求去调用 | 请求方式需要封装header，响应结果封装在response中，需要拆出来 | [[#~~RestTemplate~~]]<br>[[SpringCloud#服务调用：OpenFeign]]<br>这两个本质都是Http方式！ |
| RPC方法：想调用本地方法一样，<br>调用远程方法          | 传递参数，拿到结果，简单直接<br>性能更高                | Dubbo框架（RPC实现）就是存储提供方的地址和方法名，消费者方需要的时候去那里找                                |

==**现在都用[[SpringCloud#服务调用：OpenFeign]]**==

(url, requestMap, ResponseBean.class)
	url    请求的微服务地址
	requestMap    封装成Map的请求参数
	ResponseBean.class    HTTP响应转换成的对象类型

Spring FrameWork内置的Http响应码（枚举类）

#### 常用方法
##### getForObject()
发送GET请求，直接将==响应体==中数据转化成的对象，基本就是JSON
参数：
	1、url：要访问的资源URL地址
	2、responseType：希望将响应体中数据转化成什么类型的对象 Result.class
	~~urlVariable：可选参数，如果使用了占位符，可以使用这个参数传递实际值~~
	3、uriVariable / requestMap：可选参数，请求参数
##### getForEntity()
发送GET请求，将响应结果包装在==ResponseEntity==中，里面除了响应体数据，还有响应头、响应状态码等信息，更加丰富。
##### postForObject
发送POST请求，直接将==响应体==中数据转化成的对象，基本就是JSON
参数：
	1、url
	2、request对象     需要写入的对象实例，例如：payDTO
	3、responseType    响应体的数据转化成什么对象，例如Result.class
	4、Map
##### postForEntity
发送POST请求，将响应结果包装在==ResponseEntity==中，里面除了响应体数据，还有响应头、响应状态码等信息，更加丰富。

##### delete()
发送DELETE请求，无返回值

##### put()
发送PUT请求，更新资源。

### RabbitTemplate
`import org.springframework.amqp.rabbit.core.RabbitTemplate`

![[RabbitMQ#^366d02]]

| 方法名               | 方法功能                        | 所属接口            | 接口所属类          |
| ----------------- | --------------------------- | --------------- | -------------- |
| confirm()         | 确认消息是否发送到==交换机==            | ConfirmCallback | RabbitTemplate |
| returnedMessage() | 确认消息是否发送到==队列==<br>注意：失败才调用 | ReturnsCallback | RabbitTemplate |
#### 接口
##### ConfirmCallback
发送到交换机上
[[RabbitMQ#confirm() -> @override ConfirmCallback]]

##### ReturnsCallback
发送到消息队列上
[[RabbitMQ#returnedMessage() -> @override ReturnsCallback]]
#### 常用方法
##### convertAndSend()
在生产者端，将java对象转换为消息，发生给RabbitMQ服务器发送消息
	参数1：指定交换机
	参数2：指定路由键
	参数3：要发的消息
	参数4：其他（例如==消息后置处理器== ->接口匿名实现类）

```
@SpringBootTest  
public class RabbitMQTest {  
  
    public static final String EXCHANGE_DIRECT = "exchange.direct.order";  
    public static final String ROUTING_KEY = "order";  
    public static final String QUEUE_NAME  = "queue.order";  
  
    @Autowired  
    private RabbitTemplate rabbitTemplate;   //注意测试类路径的问题  
  
  
    // 模拟生产者向RabbitMQ服务器发送一条消息！  
    @Test  
    public void test01SendMessage() {  
        rabbitTemplate.convertAndSend(EXCHANGE_DIRECT,ROUTING_KEY,"Hello Rabbit! I am SpringBoot");  
    }  
}
```

##### receiveAndConvert()
接收消息并将其转换为Java对象


### TransactionTemplate
编程式事务

TransactionTemplate的示例代码。
思路：execute内部传递匿名实现类TransactionCallback -> @Override doInTransaction方法 ^5d3d7f
```
public void performTransactionalOperation(String data){  
    transactionTemplate.execute(new TransactionCallback<Void>() {  
        @Override  
        public Void doInTransaction(TransactionStatus status) {  
            try {  
                //执行事务操作，可以包含一组数据库读写操作  
                jdbcTemplate.update("sql",data);  
                throw new RuntimeException("an error");  
            }catch(Exception e) {  
                //失败,事务回滚  
                status.setRollbackOnly();  
                throw new RuntimeException(e);  
            }finally {  
                return null;  
            }  
        }  
    });  
}
```

lambda表达式简化版！
```
public void performTransactionUseTemplate(){  
    transactionTemplate.execute((status)->{  
        //jdbcTemplate执行语句  
        return Boolean.TRUE;  
    });  
}
```



## 其他特性
这里记录在日常开发过程中，学习到的新的Spring知识点。
是之前SSM课程没有涵盖的，都记录在这里！

### 定时任务
#### @Scheduled(参数)
在具体执行定时任务的方法上加@Scheduled()注解，并指定时间间隔
以下几个参数：

| XX                                | 单位：毫秒                           |
| --------------------------------- | ------------------------------- |
| cron                              | 设置定时执行表达式 [[linux#cron表达式]]     |
| zone                              | 执行时间的时区                         |
| fixedDelay 和 ==fixedDelayString== | 一个固定延迟时间执行<br>上一个任务完成后，延迟多长时间执行 |
| fixedRate 和 ==fixedRateString==   | 一个固定频率执行<br>上一个任务开始后，多长时间后开始执行  |
| initialDelay 和 initialDelayString | 初始延迟时间<br>第一次被调用前延迟的时间          |
```
@Service
public class TestService2 {

    private static SimpleDateFormat format = new SimpleDateFormat("HH:mm:ss");

    // 初始延迟1秒，每隔2秒
    // 第一次被调用前延迟1秒，上一个任务开始后，2秒执行
    @Scheduled(fixedRateString = "2000",initialDelay = 1000)
    public void testFixedRate(){
        System.out.println("fixedRateString,当前时间：" +format.format(new Date()));
    }

	// 每次执行完延迟2秒
	// 上一个任务完成后，延迟2秒执行
    @Scheduled(fixedDelayString= "2000")
    public void testFixedDelay(){
        System.out.println("fixedDelayString,当前时间：" +format.format(new Date()));
    }

    // 每隔3秒执行一次  0/3表示0开始，步长为3，也即每隔3秒
    @Scheduled(cron="0/3 * * * * ?")
    public void testCron(){
        System.out.println("cron,当前时间：" +format.format(new Date()));
    }

```

#### @EnableScheduling
在主启动上，加上@EnableScheduling注解，会自动扫描带有@Scheduled注解的方法，并按时间间隔执行。

配置类上开启自动任务
```
@Configuration
@EnableScheduling 
public class AopConfig{ 
	……
}
```

#### TaskScheduler接口
帮助我们实现定时任务、周期性任务和异步任务的接口
##### 接口方法
###### schedule(Runnable task, Trigger trigger)

##### 【实现类】ThreadPoolTaskScheduler
TaskScheduler接口的实现类
Spring框架中，创建一个具有线程池功能的任务调度器，可以灵活配置线程池的大小，任务执行的延迟、间隔等参数

- setPoolSize(int num)        设置线程池大小
- setThreadNamePrefix("str")   设置线程的名字前缀

1、构造一个taskScheduler对象，注入ioc容器
```
@Configuration
public class TaskConfig {

	@Bean
	public TaskScheduler taskScheduler() {
		ThreadPoolTaskScheduler threadPoolTaskScheduler = new ThreadPoolTaskScheduler();
		threadPoolTaskScheduler.setPoolSize(5);
		threadPoolTaskScheduler.setThreadNamePrefix("ThreadPoolTaskScheduler")
		return threadPoolTaskScheduler
			}
}
```

2、在需要使用任务调度器的类注入，并重写schedule（接口的匿名实现

```
@Autowired
private ThreadPoolScheduler taskScheduler;

taskScheduler.schedule( () -> {……}, newCronTrigger(""));
```

### 异步
@EnableAsync
@Async
可以在应用程序中开启多线程执行异步任务，提高应用程序的性能和响应速度

#### @EnableAsync
主启动或配置类上开启异步功能
```
@EnableAsync
@SpringBootApplication
public class AysncDemoApplication {
	public static void main(String[] args) {
		SpringApplication.run(AysncDemoApplication.class, args);
	}
}
```

#### @Async
可以加到方法上，也可以加到类上（对类里面所有方法都起作用）

注意：1、加了@Async注解的类，一定要加入IOC容器
2、一定是其他类调用这个方法才生效，本类调用不生效（因为本类就是在一个线程中）

```
@Service
public class AsynchronousService{
	@Async 
	public Future springAsynchronousMethod(){
		Interger result = longTimeMethod(); 
		return result;
	}
}
```

其他类中注入并使用，返回值是Future对象封装，用get()获取
```
@Autowired
private AsynchronousService asynchronousService;

public void useAsynchronousMethod(){
    Future future = asynchronousService.springAsynchronousMethod();
    future.get(1000, TimeUnit.MILLISECONDS);    // 等待1秒
}
```

### Spring事件驱动模型
待了解
[[设计模式#Observer 观察者模式]]
