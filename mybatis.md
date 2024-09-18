![](https://raw.githubusercontent.com/rockyshen/blog-img/master/20210411175012.jpg)
持久层的框架，提高持久层数据处理效率。是基于Jdbc、JdbcTemplate(Spring旗下的)、DbUtils的进一步优化。
mybatis几乎免除了所有JDBC代码及其设置参数和获取结果集的工作。
mybatis可以通过xml或注解来配置和映射：==原始类型、接口和Pojo==为数据库中的记录
持久层框架都是给予[[ORM]]思维的框架
![[ORM#^2fc863]]
https://mybatis.org/mybatis-3/zh_CN/index.html

曾用名：ibatis
使用版本：3.5.11

- jdbc在多表查询时非常麻烦！几乎不可用！
- hibernate封装过于严重，学习曲线陡增，自定义空间小（被架构师淘汰了）
- mybatis封装比较轻量级，自定义空间大

## 各种依赖
现在开发中，一般不会单独引入mybatis的依赖包，如果要用的话，固定这个版本，本地maven仓库就是这个！可以复用，不要重复下载不同版本！
```
<dependency>
   <groupId>org.mybatis</groupId>
   <artifactId>mybatis</artifactId>
   <version>3.5.11</version>
</dependency>
```

mysql-connector-java ，必须的MySQL驱动
依赖包，固定用这个版本，本地maven仓库已有！
```
<dependency>  
    <groupId>mysql</groupId>  
    <artifactId>mysql-connector-java</artifactId>  
    <version>8.0.25</version>  
    <scope>runtime</scope>  
</dependency>
```

mybatis整合spring-boot的依赖；
依赖包，固定也用这个版本，本地maven仓库已有！
![[SpringBoot#^d8c52e]]

mybatis-plus（中包含了mybatis）整合spring-boot的依赖

## 基本使用

```
├── java.com.atguigu
	├── mapper
		├── EmployeeMapper         # 接口
	├── pojo
		├── Employee               # 实体类
├── resources.com.atguigu
	├── mapper
		├── EmployeeMapper.xml        # mapper约束的xml
	mybatis-config.xml                # mybatis配置文件
```

### 操作步骤
#### EmployeeMapper接口 + EmployeeMapper.xml
准备XxxMapper接口和XxxMapper.xml文件
	XxxMapper接口：规约的方法，==Mapper接口中不允许重载==！不然找不到实例
	XxxMapper.xml：映射文件，专注写sql语句，理解为对接口的实现

| XX                          | XX                                                                                               |
| --------------------------- | ------------------------------------------------------------------------------------------------ |
| @Mapper                     | 标识为Mybatis接口，映射mybaits.xml文件；<br>等价于 [[#映射文件中的`<mapper namespace>`]]<br>mybatis V3.3之后，自动扫描，可以不加 |
| @MapperScan(basepackage="") | 扫描相同包下，添加了@Mapper注解的Mapper接口，注册为bean，将其加入ioc容器管理<br>等价于 [[#配置文件中的`<mapper resource>`]]           |

  @MapperScan 注解可以加到主启动类（*或配置类上也可以*），总之在运行启动时就会加载就行了。
指定扫描`XxxMapper.java`接口的范围 ^4a52a7

注意：后期Spring-Boot-Starter有自动配置类，会自动扫描Mapper接口，不需要我们手动添加扫描了。

思考：@Mapper 和 @Repository 有什么区别？
@Repository注解是把Mapper层的类加入ioc容器中；
@Mapper注解是建立接口类与映射文件XxxMapper.xml的关联，以便执行sql语句。

#### 测试环境MybatisTest
在测试环境下
3、使用mybatis的api进行数据库查询即可。
	全局：SqlSessionFactory
	一次性：SqlSession.getMapper(接口.class)

测试场景下，使用mybatis的5个步骤，固定操作 ^cd1d90
```
@Test  
public void test_02() throws IOException {  
    //1.读mybatis-config.xml配置文件
    // 别导错包！Resources是apache那个包  
    InputStream ips = Resources.getResourceAsStream("mybatis-config.xml");  
  
    //2.创建sqlSessionFactory（一次连接就只有一个）  
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(ips);  
  
    //3.根据sqlSessionFactory创建sqlSession(每次业务创建一个，用完就释放)
    //自动开启，后续需要手动commit，否则不更新数据库
    SqlSession sqlSession = sqlSessionFactory.openSession();
```

**ibatis方式**
```
	//4.直接在sqlSession上执行crud的方法进行数据库查询即可  
	Student student = sqlSession.selectOne("xx.jj.kkk", 1);  
	System.out.println("student = " + student);
```
缺点：
1、selectOne方法的String容易写错
2、Object只有一个，多参数不行
3、返回值是Object，需要强转！

**mybatis方式**
```
//4.获取接口的代理对象（代理技术，调用代理对象的方法，就会查找Mapper接口的方法）  
EmployeeMapper mapper = sqlSession.getMapper(EmployeeMapper.class);  
Employee employee = mapper.queryId(1);  
System.out.println("employee = " + employee);

//5.提交并关闭
sqlSession.commit();    //上面如果是DML语句，必须commit,否则数据库不更新
sqlSession.close();
```

注意：sqlSessionFactory.openSession()方法只会自动开启事务，但是不会自动提交！
==如果是DML语句，必须sqlSession.commit,否则数据库不更新。==
除非sqlSessionFactory.openSession(autoCommit=true)，才会自动提交

**简化**：利用@BeforeEach 和 @AfterEach注解，在每个@Test方法之前和之后执行
```
public class MybatisTest {  
    private SqlSession sqlSession;  

	// test方法之前，初始化一个sqlSession
    @BeforeEach  
    public void init() throws IOException {  
        sqlSession = new SqlSessionFactoryBuilder()  
                .build(Resources.getResourceAsStream("mybatis-config.xml"))  
                .openSession();  
    }  
      
    @Test  
    public void test01() throws IOException {  
        OrderMapper mapper = sqlSession.getMapper(OrderMapper.class);  
        Order order1 = mapper.queryOrderById(1);  
        System.out.println("order1 = " + order1);  
    }  
      
    @AfterEach  
    public void clean() {  
        sqlSession.close();  
    }  
}
```

#### mybatis原理图
演进过程：
ibatis的步骤
1、写一个pojo实体类、Mapper.xml（没有接口）
2、写mybatis-config.xml
3、基于SqlSession直接允许selectOne()等一系列方法

ibatis的三个缺点
	1、sql语句是字符串容易出错，
	2、selectOne方法的形参就只能传一个参数，
	3、返回值默认是Object类需要[[Java_01_基础#强制类型转换]]

mybatis的改进
1、利用接口，规定方法！[[Java_02_面向对象#接口 - interface 关键字]]
2、利用动态代理，将接口和xml（getMapper方法）生成代理类，来执行sql  ([[Spring FrameWork#动态代理：JDK动态代理 --接口--> 拜把子]])   ^0352fd

就能彻底改进ibtis的三个缺点！
mybatis用接口类的全限定符.方法名（namespace.id）作为参数，传递给SqlSession的方法的参数1（String s）
见下图，虚线框出来的部分，就是mybatis基于ibatis的进一步封装！

JDK动态代理，就是用来处理：接口上的动态代理！
	还有一种是CGLIB动态代理，是用来处理extends继承的方式的动态代理，在mybatis中没有用到这个！
	
![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202407280049321.png)

### 配置文件 mybatis-config.xml
主要干两方面的事情
1、mybatis设置`<settings>`、别名`<typeAliases>`、插件`<Plugins>`
2、druid连接池数据库信息`<environments>`、mapper.xml的位置指定`<mapper resource="">`

后期被完全抛弃！
1  -->  写入config/MapperJavaConfig配置类中的SqlSessionFactoryBean加入ioc容器的过程中
2  -->   mapper.xml指定写入config/MapperJavaConfig配置类中的MapperScannerConfigurer加入ioc容器的过程中； druid连接池写入config/DataSourceJavaConfig配置类中

全局配置mybatis的配置文件！（例如连接数据库啊，驼峰啊、别名啊！）
[mybatis官网-XML配置](https://mybatis.org/mybatis-3/zh_CN/configuration.html)
一般项目命名为：mybatis-config.xml
标签添加位置，严格按照官网文档的先后顺序！

#### 标签顺序
```
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE configuration  
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  
        "http://mybatis.org/dtd/mybatis-3-config.dtd">  
<configuration>  
	<settings>
		<setting name="logImpl" value="STDOUT_LOGGING"/>
	</settings>

	<typeAliases>

	<plugins>
	
    <environments default="development">             默认使用哪个环境
        <environment id="development">               可以有开发环境，测试环境
            <transactionManager type="JDBC" />       自动开启事务
            <dataSource type="POOLED">               mybatis提供的连接池
                <!-- 建立数据库连接的具体信息 -->  
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>  
                <property name="url" value="jdbc:mysql://118.25.46.207:3306/mybatis-example"/>  
                <property name="username" value="root"/>  
                <property name="password" value="p@ssw0rd"/>  
            </dataSource>
        </environment>
    </environments>  

	<mapper>
```

^77c997
#### `<mapper resource>`
`<mappers>`下包含多个`<mapper>`
[mybatis 映射器 ](https://mybatis.org/mybatis-3/zh_CN/configuration.html#mappers)

每个mapper标签，定位一个xml映射文件的路径。（路径的起始点是从Resources文件夹开始！）
等于注解@MapperScan(basepackage) = resource ^c2d820

```
├── src/main/java
	├── com.atguigu.mapper
		├── XxxMapper        # Mapper接口

├── resources
	├── mappers
		├── XxxMapper.xml    # xml文件
```

例如下例表示：resources文件夹下的，mappers文件夹下的，EmployeeMapper.xml映射文件
```
# 配置文件 mybatis-config.xml

<configuration>
	<mappers>
		<mapper resource="mappers/XxxMapper.xml">
```


mybatis-config.xml 中只保留 settings、typeAliases、plugins。

数据库连接池、qlSessionFactory、Mapper代理对象都放到 MapperJavaConfig配置类中定义

**常用设置**
`<setting name="logImpl" value="STDOUT_LOGGING"/>`      添加控制台日志输出
`<setting name="mapUnderscoreToCamelCase" value="True"/>`    实体类的Field与mysql的column自动蛇形与驼峰映射，只在resultType生效！
`<setting name="autoMappingBehavior" value="FULL">`      resultMap深层结构，自动映射非主键列，默认值：PARTIAL，只映射单层

==重要，这个情况发生`<select>`中间的在**sql语句**中==
1、如果实体类与sql的column名完全一样，那resultType填一样的那个就行；
2、如果实体类的Field是（aColumn）,sql的column名是（a_column）,那resultType填aColumn=a_column
3、如果实体类的Field和sql的column完全不一致，那就用resultMap，手写映射关系 ^9f53de

#### [[SpringBoot#整合mybatis]]
[mybatis-spring-starter参考文档](https://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/zh/index.html)

mybatis-starter自动配置
mybatis-spring-boot-starter自动配置（约定大于配置），然后额外需要自定义的配置信息，写在application.yml中

约定：
1、默认会搜寻带有@Mapper注解的mapper接口
2、自定义扫描 @MapperScan

【主流】方式3：application.properties配置
自定义：XML 映射文件的路径。

classpath：类路径，是指Java应用程序在运行时查找类文件src和资源Resources的位置集合。
classpath的起始点是Resources文件夹!
```
mybatis:  
  mapper-locations: 
	  classpath: mapper/*.xml   # Resources/mapper/*.xml
```

mybatis-plus的配置也相同
```
mybatis-plus.mapper-locations=classpath:com/atguigu/ggkt/vod/mapper/xml/*.xml
```
### 映射文件 mapper.xml
[Mybatis官网-XML映射文件](https://mybatis.org/mybatis-3/zh_CN/sqlmap-xml.html)
一般项目命名为：XxxEmployeeMapper.xml（例如EmployeeMapper.xml）
EmployeeMapper.java    接口，约定方法
EmployeeMapper.xml     xml文件，实现接口里的方法，sql语句！

映射文件 mapper.xml中，只关注在写sql语句！
```
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE mapper  
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">  
        
<mapper namespace="com.atguigu.mapper.EmployeeMapper">  
    <!-- 声明标签写sql crud: select insert update delete -->  
    <!-- 每个标签对应一个Mapper接口的实现 -->  
    <!-- mapper接口不能重载，因为不能识别唯一方法名 -->  
    <select id="queryById" resultType="com.atguigu.pojo.Employee">  
        // sql语句
        select emp_id empId, emp_name empName, emp_salary empSalary from t_emp where emp_id = #(id)  
    </select>  
</mapper>
```

注意：接口类和Mapper.xml都必须都被Spring读到，否则就会抛异常：`BindingException: Invaid bound Statment`。这个异常就是说，两者没有对应，找不到xml中的sql语句
核心原因是Maven构建的项目，只会编译.java文件，.xml不会加入编译，导致缺失

```
├── src/main/java
	├── com.atguigu.mapper
		├── XxxMapper        # Mapper接口

├── resources
	├── mappers
		├── XxxMapper.xml    # xml文件
```

#### Mapper接口类与映射文件.xml关联的方式
思考，最终mybatis-config.xml会被注解替换掉！但是XxxMapper.xml还是会保留下来写复杂的sql语句，如何确保xml与Mapper类合并在一起生效，思考🤔？
最终答案：resources下与src下包名结构保持一致！就能打包在一起！或者在application.yml中配置mapper查询路径

方式1：
直接在XxxMapper.xml的`<mapper namespace>`中指定接口的类全限定符！保证接口和xml一对一的关系！
```
<mapper namespace="com.atguigu.mapper.XxxMapper">
```

方式2：接口类的路径和映射文件的路径，保持一模一样，这样编译的是否就会把接口类和映射文件放在一个目录里面（一家人就齐齐整整），如下图
```
├── src/main/java
	├── com.atguigu.mapper
		├── XxxMapper        # Mapper接口

├── resources
	├── com.atguigu.mapper
		├── XxxMapper.xml    # xml文件

├── target
	├── com.atguigu.mapper
		├── XxxMapper.java
		├── XxxMapper.xml
```

然后配置文件mybatis-config.xml中进行如下配置
在config中用`<package name="com.atguigu.mapper">` 就能自动读！ ^ec06c7
```
<mappers>
	<package name="com.atguigu.mapper">  # 将包内的映射器接口全部注册为映射器
```
思考：Java的Mapper文件和mybatis的Mapper文件，是否一定要保持相同的目录层级（例如：java/org.rockyshen.lottery.mapper 和 resources/mapper
回答：只要在application.xml的mybatis配置环节！如果你Java和Resources保持一致，这个可以不设置，就能读到；如果不一致，就要设置一下classpath定位mybatis的Mapper文件

方式3：这种方法也比较冷门！不要这么弄！
pom.xml的`<build>/<Resource>`中添加，指定Resources下的指定后缀名的文件，输出到src目录中。这也太奇怪了！ ^d9a47e
```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.yml</include>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
            <includes> 
	            <include>**/*.yml</include>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
    </resources>
</build>
```

^a12f3a

这样maven编译时也会包含mapper.xml了！

java文件夹下可以用点分割！
resources文件夹下创建多层，用斜杠/分割
![[Java_01_基础#^0b9096]]


### 映射文件的标签总结
#### 标签结构
```
<mapper namespace="">  
    <resultMap id="" type="">  
        <id column="" property="" />  
        <result column="" property="" />  
        <result column="" property="" />  
        <association property="" javaType="">  
            <id column="" property="" />  
            <result column="" property="" />  
        </association>  
    </resultMap>  
  
    <select id="" resultMap="">  
	    sql语句
	</select>  
</mapper>
```

#### `<mapper namespace>`
参数 namespace，指定接口类的全限定符

映射文件的`<mapper namespace>`指定接口类的全限定符！
必须保证一个namespace对应一个接口类！

注意：==mybatis是不支持重载方法的！==Mapper层的接口方法名，一个是一个，不能重名！

底层是因为动态代理时，namespace会作为key写入SqlSessionFactory中，key=namespace，不允许重名的！

注解 @Mapper = namespace 。namespace定位到接口类，<select id定位到接口类中的方法名

```
├── src/main/java
	├── com.atguigu.mapper
		├── XxxMapper        # Mapper接口

├── resources
	├── mappers
		├── XxxMapper.xml    # xml文件
```

```
# 映射文件 XxxMapper.java接口类的namespace

<mapper namespace="com.atguigu.mapper.XxxMapper">
```
#### `<resultMap>`
![[#^b4786c]]

resultMap可以处理多层级的映射，和resultType二选一！ ^a27c5e

需要填写的属性：
- id           这个resultMap的标识符
- type        被映射的对象类型

##### `<id>`
`<id column="tId" property="t_id">`
指定主键映射关系
- column   数据库中主键列名
- property    实体类的field名
##### `<result>`
`<result column="tName" property="t_name">`
普通列的映射关系

示例代码
```
<resultMap id="tMap" type="teacher">
	这里自定义
	<id column="tId" property="t_id">
	<result column="tName" property="t_name">
</resultMap>

<select id="queryById" resultMap="tMap">
	sql查询语句
</select>
```

##### `<association>`
多表查询时，给单个对象进行映射
- property      实体对象属性的名字
- javaType      这个对象的数据类型的全限定符，或别名

###### `<id>`
被插进来的对象的主键映射

###### `<result>`
被插进来的对象的其他列映射

##### `<collection>`
多表查询时，给多个对象的集合进行映射
- property         集合属性的名字
- ofType            集合的泛型的类型

###### `<id>`
被插进来的对象的主键映射

###### `<result>`
被插进来的对象的其他列映射

#### `<select>`
需要填写的属性
- id：在命名空间namespace中唯一标识，可以被用来引用这句sql语句，一般写方法名
- resultType：期望这条语句的结果映射到类的全限定符或[[#类型别名]]。注意，如果返回的是集合，应该写**集合的泛型**`List<Student>`，而不是集合本身的类型。与resultMap二选一
- resultMap：二选一，自定义映射，多表查询，如果用resultMap，`resultMap=tMap`
#### `<insert>`
注意：返回值类型默认就是int，所以不用写resultType，因为你向数据库发起更新操作，数据库不会返给你什么东西吧。它就给你返回一个row（受影响的行数）。也可以[[#主键回显]]
需要填写的属性：
- id：在命名空间namespace中唯一标识，可以被用来引用这句sql语句，一般写：方法名
- useGeneratedKeys：我想要数据库自增长的主键值
- keyProperty：将主键值赋值给实例类的那个Field
- keyColumn：数据库中那列是主键列

#### `<update>`
注意：返回值类型默认就是int，所以不用写resultType，因为你向数据库发起更新操作，数据库不会返给你什么东西吧。它就给你返回一个row（受影响的行数），也可以[[#主键回显]]
需要填写的属性：
- id：在命名空间namespace中唯一标识，可以被用来引用这句sql语句，一般写：方法名

#### `<delete>`
注意：返回值类型默认就是int，所以不用写resultType，因为你向数据库发起删除操作，数据库不会返给你什么东西吧。它就给你返回一个row（受影响的行数），也可以[[#主键回显]]
需要填写的属性：
- id：在命名空间namespace中唯一标识，可以被用来引用这句sql语句，一般写：方法名

#### SQL语句
包裹在标签中间的SQL语句，涉及到实体类（Filed）与数据库column的映射
```
没开启蛇形与驼峰映射
<select>
	select emp_id empId, emp_name empName, emp_salary empSalary from t_emp where emp_id = #{id}
</select>

开启蛇形驼峰映射
<select>
	select emp_id, emp_name, emp_salary from t_emp where emp_id = #{id}
</select>
```
![[#^9f53de]]

### 配置文件与映射文件中`<mapper>`的区别
思考：mybatis-config.xml 与 映射文件 mapper.xml中`<mapper>`标签的区别：
反思：第一次的理解是完全错误的🙅 -->~~其实mybatis-config.xml中的`<mapper>`指定resources/mappers/OrderMapper.xml的路径位置。和XML映射文件中`<namespace>`指定接口类的全限定符，效果是一样的~~

正确的思考：

| XX                                  | XX                                                                                                                         |
| ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| 配置文件 mybatis-config.xml中的`<mapper>` | ![[#^c2d820]]                                                                                                              |
| 映射文件 XxxMapper.xml中的`<mapper>`      | 映射文件的`<mapper namespace>`指定接口类的全限定符！保证一个namespace对应一个接口类！<br>注解 @Mapper = namespace 。namespace定位到接口类，<select id定位到接口类中的方法名 |
|                                     |                                                                                                                            |

### 配置文件`<mappers>`中的resource、class和package
[mybatis-config中的映射器（`<mappers>`标签）](https://mybatis.org/mybatis-3/zh_CN/configuration.html#mappers)

mappers主要职责是找到.xml文件，resource就是指定这个路径的，只要设置正确，就一定会生效

而class和package都是关联的是XxxMapper接口类！只有当输出target的接口类和.xml文件在一个文件夹下，才会生效。
这下彻底明白了！
```
    <mappers>  
        <!-- 指定xml映射器的位置 -->  
  
        <!-- 使用相对于类路径的资源引用 -->  
<!--        <mapper resource="mapper/RoleMapper.xml"/>-->  
  
        <!-- 使用映射器接口类（也即RoleMapper类）的完全限定类名 -->  
        <mapper class="org.rockyshen.mapper.RoleMapper"/>  
  
        <!-- 将包内的映射器接口全部注册为映射器 -->  
<!--        <package name="org.rockyshen.mapper"/>-->  
    </mappers>

```

### 向sql语句传参
功能开关：打开日志输出到Terminal

`#{ key }`     占位符+赋值（推荐）               动态值

`${ key }`     字符串拼接   "emp_id =" + id    动态列名、容器名  用这个

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B45%E6%9C%8816%E6%97%A5%2013_37_10.jpg)
### 参数输入
这里的数据输入指的是，调用代理了EmployeeMapper接口的代理类实例mapper调用方法时传递的值。`Employee employee = mapper.queryById(1);`

#### 简单类型 #{id} 和 ${id}
包含一个值的数据类型：[[Java_01_基础#基元类型]] 、 [[标准包.类#基元包装类]]  、字符串String
`#{int}`  `#{string}`
注意DML操作的返回值类型，规定是int！所以resultType不用写！

#{id}   底层就是  [[JDBC#prepareStatement]]
${id}   底层就是  [[JDBC#createStatement()]] ^05dd8a

Q：#{id}和${id}的区别？ #Java八股/SSM
A：#{id}对应在mapper.xml中的sql是预编译好的（不是拼接），参数位置用？占位，然后将变量占位替换；
${id}则是直接在mapper.xml中sql语句的声明变量位置进行拼接，有sql注入风险（or 1=1）。 ^1b273b

预编译的意思是，表名已经在编译时确定了，无法再修改了！

思考：#{id}更安全，也更常用。但是当需要插入动态表名、列名或其他Sql片段（动态排序条件）时，${id}就更好

联想到AsyncFlow异步任务框架中，分表策略，当第一张任务表满容量时，触发扩容，就是创建新表，就需要用到${newTabalName}
```
<select id="selectCount" resultType="int">  
    SELECT COUNT(*) from ${tableName}
</select>

<!--    当旧表满时，创建新表 t_lark_task_2-->    
<update id="createNewTable">  
    CREATE TABLE IF NOT EXISTS `${newTableName}` (
```

#### 复杂类型  @Param
包含多个值的数据类型：
	实体类：Employee类（包含多个Field）
	集合类：List、Set、Map
	数组类：`int[]` 、`String[]`
	复合：`List<Employee>`

注意区分[[SpringMVC#利用param接收参数 --> @RequestParam]]，这个是用来从URL的param中提取参数提供个SpringMVC的。我们这里是提供给Mybatis的！
##### 实体类
实体类的属性 ->  参数名
也可以加个@Param注解
```
Employee实体类的属性：empName、empSalary

<insert id="insertEmployee">  
    insert into t_emp(emp_name empName, emp_salary empSalary) values (#{empName},#{empSalary});</insert>
```

##### 多个简单类型的组合 @Param("id") id
多个简单类型的参数组成的数组或集合，==用@Param注解，相当于让多个参数一一对应起来。==

加到：Mapper层的方法的形参`mapper.selectXX(@Param("id") Integer id)`上，声明这个参数对应到xml中的`#{id}`这个变量对应！

【常用】方案1：@Param("形参名")注解，放在接口方法的参数上
方法2：mybatis提供的机制 arg0 、arg1
```
mapper.queryByName(@Param("name")name, XXX)

<!--  
传递参数为零散的简单类型  
List类：根据员工姓名和工资查询员工信息  
-->  
<select id="queryByNameAndSalary" resultType="com.atguigu.pojo.Employee">  
	方案1
    SELECT emp_id empId, emp_name empName, emp_salary empSalary    FROM t_emp    WHERE emp_name = #{name} and emp_salary = #{salary};

	方案2
    SELECT emp_id empId, emp_name empName, emp_salary empSalary    FROM t_emp    WHERE emp_name = #{arg0} and emp_salary = #{arg1};
</select>

```
##### Map类
填map的key！
```
<!--  
传递参数为Map类型  
Map类：根据map{name=员工名字， salary=员工工资}插入该员工  
-->
<insert id="insertEmpByMap">  
    insert into t_emp(emp_name empName, emp_salary empSalary) values (#{name},#{salary});</insert>
```
### 结果输出
DML操作型语句，默认返回int类型（返回的是rows，受影响的行号）

联想到：[[Django-2#序列化器]]

查询语句指定返回值
#### 单个类型
resultType，指定返回值的数据类型！按照规则自动映射，resultType只能映射一层结果！ ^b4786c

![[#^a27c5e]]

resultType语法：
	类的全限定符（java.lang.String，自己的类com.atguigu.pojo.Employee）
	别名简称（例如：string、double）
基元类型：`_int` 、`_double`  (加个下划线)
包装类：`int` `double` ^f100be
```
<!--  
接收数据  
根据员工ID查询员工姓名  
-->  
<select id="queryNameById" resultType="string">  
    select emp_name from t_emp where emp_id = #{id};</select>
```

##### 类型别名
[mybatis提供72种别名简称，用于返回类型指定](https://mybatis.org/mybatis-3/zh_CN/configuration.html#typeAliases)
写在写在mybatis-config.xml，environment前面
重要，在`<select id="" resultType="">`的resultType属性上生效！
把自定义的类，也取个别名  例如：com.atguigu.pojo.Employee

```
每个类，单独定义
<configuration>
	<typeAliases>
	  <typeAlias alias="Author" type="domain.blog.Author"/>
	  <typeAlias alias="Blog" type="domain.blog.Blog"/>
	</typeAliases>

	<settings>

	<enviroments>

</configuration>
```

【常用】按一个包，批量定义，pojo包下所有实体类，都自动有别名，首字母小写
com.atguigu.pojo.Employee 这个类自动别名为  employee
`<relect resultType="com.atguigu.pojo.Employee">`  -->  `<select resultType="employee">`
```
<typeAliases>
  <package name="com.atguigu.pojo"/>
</typeAliases>
```
#### Map类型
当需要接收多个数据时，有没有一个合适的类承载时，可以存在Map类中。
把多个结果存放在map字典中，{key=列名，val=结果}

例如：查询部门的最高工资和平均工资，平均工资并不是employee类的一个Field！
```
Map<String,Object> selectEmpNameAndMaxSalary();

<select id="selectEmpNameAndMaxSalary" resultType="map">  
    SELECT        emp_name 员工姓名,  
        emp_salary 员工工资,  
    (SELECT AVG(emp_salary) FROM t_emp) 部门平均工资,  
    FROM t_emp WHERE emp_salary (SELECT MAX(emp_salary) FROM t_emp)</select>
```

#### List类型
如果返回值是集合，必须指定泛型！
ibatis有两个查询方法：selectOne 和 selectList，selectOne底层也是调selectList，加了一个大于1的判断。所以只需要写泛型，不要写集合类型！

#### 主键回显
获取插入数据的主键 PRIMARY KEY
```
<insert id="insertEmpByMap" useGeneratedKeys="true" keyColumn="emp_id" keyProperty="empId">  
    insert into t_emp(emp_name empName, emp_salary empSalary) values (#{name},#{salary});</insert>

测试类就能读取到！
employee.getEmpId()
```
useGeneratedKeys="true"     我想要数据库自增长的主键值
keyColumn="emp_id"           数据库中那列是主键列
keyProperty="empId"            将主键值赋值给实例类的那个Field

如果非自增长的话，一般用UUID
这是我们自己写：
```
Teacher teacher = new Teacher();
teacher.setName("张老师")；

String id = UUID.randomUUID().toString().replaceAll("-","");
teacher.setID(id);

// 这个id就能用了
```

交给mybatis
```
<select>
	<selectKey order="before" resultType="string" keyProperty="tID">            
		SELECT REPLACE(UUID(), '-','')    //在执行sql之前，先执行生成主键的操作
	</selectKey>
	INSERT XXX
</select>
```
order="before"                           在执行insert之前还是之后执行
resultType="string"                     返回值类型
keyProperty="tID"                       赋值给Field

## 多表映射
建表角度，是双向的，表与表之间，有一对多和多对一的关系。
==**查询角度，是单向的，只有对一关系，或对多关系！**==

客户表与订单表
如果从订单角度查询，一个订单只对一个客户，是对一映射
如果从客户角度查询，一个客户对多个订单，是对多映射

```
public class Customer{
	private Integer customerId;
	private String customerName;
	private List<Order> orderList;   // 站在基于客户查询的角度，体现对多的关系
}

public class Order{
	private Integer orderId;
	private String orderName;
	private Customer customer;       // 站在基于订单查询的角度，体现对一的关系
}
```

**忠告：**
1、只有真实发生多表查询时，才去修改实体类，否则不提前设计和修改实体类！
2、无论有多少张表，实体类设计都是两两考虑（客户、订单、订单项）
3、在多表查询时，只关注本次查询的属性。例如：查询订单和对应客户，就不要关注客户中订单的集合了（死锁，死循环）
	上述例子中，站在订单角度，查询客户，就不要关注`List<Order>`，赋空值！

### 对一映射
如果从订单角度查询，一个订单只对一个客户，是对一映射
只需要在订单类的Field中增加一个==客户类==即可，订单类就能装查询结果！
sql语句：`select * from t_order join t_customer on t_order.customer_id = t_customer.customer_id where t_order.order_id = 1;`

OrderMapper.xml
```
    <resultMap id="orderMap" type="order">  
        <id column="order_id" property="orderId" />  
        <result column="order_name" property="orderName" />  
        <result column="customer_id" property="customerId" />  
  
<!--        Customer实体类的映射-->  
        <association property="customer" javaType="customer">  
            <id column="customer_id" property="customerId" />  
            <result column="customer_name" property="customerName" />  
        </association>  
    </resultMap>
```

### 对多映射
如果从客户角度查询，一个客户对多个订单，是对多映射
只需要在订单类的Field中增加一个==客户类的集合==即可

sql语句：`select * from t_order join t_customer on t_customer.customer_id = t_order.customer_id;`


## 动态语句
`SELECT * FROM job 【WHERE location = ‘北京‘ and type = 'XXX' and salary = 'XXX'】`
where后面的条件判断，会动态变化！

mybatis框架提供：if where set trim等标签，可以写在XxxMapper.xml文件中，提供条件的动态判断。

### if和where标签
#### `<if test="">`
- test      内部作比较运行，成立则拼接

| 符号 | 含义 |  |  |
| ---- | ---- | ---- | ---- |
| != | 不等于 |  |  |
| > | 大于 | `&gt;` | 避免和`<if>` 标签符号混淆 |
| < | 小于 | `&lt;` |  |
`<where>`   当条件不满足是，会多出 where或and关键字，用where标签就可以解决！
```
<select id="query" resultType="employee">  
    select * from t_emp        
	    <where>  
            <if test="name != null">  
            emp_name = #{name}            
            </if> 
            <if test="salary!=null and salary &gt; 100">  
            and emp_salary = #{salary}            
            </if>  
        </where>  
</select>
```

### set标签
同where的作用，用在`<update>`下的条件不满足时，去除关键字
```
<update id="update">
	update t_emp
		<set>
			<if test = "empName != null">
				emp_name = #{empName} ,
			</if>
			<if test = "empSalary != null">
				emp_salary = #{empSalary}
			</if>
		</set>
		where emp_id = #{empId}
</update>
```

### trim标签
自定义规则的标签
`<trim prefix="where" preOverrides="and|or">`
替换where标签，并去掉and或or
### choose/when/otherwise标签
类似switch……case

根据两个条件查询
	1、姓名不为空就用姓名
	2、姓名为空但薪水不为空就用薪水查询
	3、都为空则查询全部）
```
<select id="query" resultType="employee">
	select * from t_emp
		where
			<choose>
				<when test="name != null">
					emp_name = #{name}
				</when>
				<when test="salary != null">
					emp_salary = #{salary}
				</when>
				<otherwise>
					1=1          <!-- 确保最后一定能满足！ -->
				</otherwise>
			</choose>

```
### foreach标签
前提条件：传入集合数据
- collection        你要遍历的集合
- open               遍历之前要追加的字符串
- close                遍历结束要追加的字符串
- seperator         分割符号
- item=“id”                 获取每个遍历项
```
<foreach collection="ids" open="(" seperator="," close=")" item="id">
	#{id}
</foreach>
```

### sql片段
没啥卵用！把问题复杂化了！
```
<sql id="selectSql">
	select * from t_emp
</sql>

下方可以直接引用
〈select id="query" resultType="employee">
	<include refid="selectSql" />
		<where>
			<if>
			</if>
		</where>
</select>
```

## 整合Spring / SpringBoot
将mybatis对象加入ioc容器
整个mybatis创建过程，我们思考一下哪些要加入ioc容器，加入的原则是：全局复用的实例化对象才加入ioc容器，那种用一次就丢弃的对象不要加入ioc容器

| XX                       | XX                                  |
| ------------------------ | ----------------------------------- |
| SqlSessionFactoryBuilder | 不加入，只用一次                            |
| SqlSessionFactory        | 一直存在，加入ioc容器                        |
| SqlSession               | 每次都创建一个，但是mybatis现在调用代理对象mapper，不加入 |
| Mapper映射实例               | 每次都创建一个，但是我们需要用啊，得加                 |

mybatis实现了==**FactoryBean接口**==，将繁琐的读取mybatis-config等操作封装在mybatis对FactoryBean的实现类中，叫做：==**SqlSessionFactoryBean**==
![[Spring FrameWork#^152080]]


mybatis整合spring所需要的依赖；
就用这个版本，本地maven仓库已有，避免重复下载！
```
<dependency>
   <groupId>org.mybatis</groupId>
   <artifactId>mybatis-spring</artifactId>
   <version>3.0.2</version>
</dependency>
```

### ~~方式1：整合Spring~~【❄️经典白雪】
保留mybatis-config.xml

将Druid连接池加入ioc容器
```
@Configuration  
@PropertySource("classpath:jdbc.properties")  
public class DataSourceConfig {  
    @Value("${jdbc.user}")  
    private String user;  
    @Value("${jdbc.password}")  
    private String password;  
    @Value("${jdbc.url}")  
    private String url;  
    @Value("${jdbc.driver}")  
    private String driver;  
    
    @Bean  
    public DataSource dataSource(){  
        DruidDataSource dataSource = new DruidDataSource();  
        dataSource.setUsername(user);  
        dataSource.setPassword(password);  
        dataSource.setUrl(url);  
        dataSource.setDriverClassName(driver);  
        return dataSource;  
    }  
}
```
注意：存在BUG，读不进去。所以DataSource单独一个配置类！！

利用SqlSessionFactoryBean将SqlSessionFactory加入ioc容器
```
@Bean
public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource){
	SqlSwssionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();

	//指定连接池
	sqlSessionFactoryBean.setDataSource(dataSource);

	//指定mybatis-config.xml
	Resource resource = new ClassPathResource("mybatis-config.xml");
	sqlSessionFactoryBean.setConfigLocation(resource);

	return sqlSessionFactoryBean
}
```

利用==MapperScannerConfigurer==（就是Mapper代理对象的FactoryBean）将Mapper代理对象加入ioc容器
```
@Bean
public MapperScannerConfigurer mapperScannerConfigurer(){
	MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();

	mapperScannerConfigurer.setBasePackage("com.atguigu.mappper");

	return mapperScannerConfigurer
}
```

### 方式2：整合SpringBoot
[[SpringBoot#整合mybatis]]
完全抛弃mybatis-config.xml

mybatis-config.xml中的配置会被mybatis-spring-boot-starter自动配置（约定大于配置），然后额外需要自定义的配置信息，写在application.yml中

将mybatis-config.xml残留的settings、TypeAlias、Plugins都在SqlSessionFactoryBean的过程中整合进去。
	原理：将所有设置都写入Configuration类的一个实例中
	然后调用SqlSessionFactoryBean.setConfiguration(configuration)传参就行了

将SqlSessionFactory加入ioc容器，基础上增加settings、TypeAlias、Plugins等
```
@Bean
public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource){
	SqlSwssionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();

	//指定连接池
	sqlSessionFactoryBean.setDataSource(dataSource);

	//指定mybatis-config.xml
	// Resource resource = new ClassPathResource("mybatis-config.xml");
	// sqlSessionFactoryBean.setConfigLocation(resource);

	//使用代码方式替换功能mybatis-config中的功能
	org.apache.ibatis.session.Configuration configuration = new org.apache.ibatis.session.Configuration();  
	configuration.setMapUnderscoreToCamelCase(true);  
	configuration.setLogImpl(Slf4jImpl.class);  
	configuration.setAutoMappingBehavior(AutoMappingBehavior.FULL);  
	sqlSessionFactoryBean.setConfiguration(configuration);  
	sqlSessionFactoryBean.setTypeAliasesPackage("com.atguigu.pojo");
	PageInterceptor pageInterceptor = new PageInterceptor();  
	Properties properties = new Properties();  
	properties.setProperty("helperDialect","mysql");  
	pageInterceptor.setProperties(properties);  
	sqlSessionFactoryBean.addPlugins(pageInterceptor);
	
	return sqlSessionFactoryBean
}
```

**********
## 高级拓展
### PageHelper插件
1、导入依赖：com.github.pagehelper 5.1.11
固定用这个版本，本地maven仓库有，避免重复下载！
```
<dependency>
   <groupId>com.github.pagehelper</groupId>
   <artifactId>pagehelper</artifactId>
   <version>5.1.11</version>
</dependency>
```

2、在mybatis-config.xml中添加插件
```
<plugins>
	<plugin interceptor="com.github.pagehelper.PageInterceptor">
		<property name="helpDialect" value="mysql" />
	</plugin>
</plugins>
```
[PageHelper属性大全](https://blog.csdn.net/jie_7012/article/details/106774283?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522170973755016800213075093%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=170973755016800213075093&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~sobaiduend~default-1-106774283-null-null.nonecase&utm_term=pagehelper%E5%B1%9E%E6%80%A7&spm=1018.2226.3001.4450)

**Mybatis中PageHelper插件的使用步骤**：
	1、pom中导依赖
	2、mybatis-config.xml中引入插件 `<plugin>`
	3、在具体业务类，==调用查询全部数据之前，使用静态方法==PageHelper.startPage()
	4、查询结果封装在PageInfo对象中。 ^d06214

```
PageHelper.startPage(pageNum,pageSize);    // 查询全部数据前，使用PageHelper

List<Employee> list = mapper.queryList();  // 执行mapper查询

PageInfo<Employee> pageInfo = new PageInfo(list);   // 结果封装在PageInfo实体类中

//pageInfo获取分页数据
当前页数据、获取当前页总条数、获取当前总页数……
```
#### startPage()
PageHelper.startPage(currentPage, pageSize)   静态方法
将查询结果传递到PageInfo( )方法中。


#### pageInfo
基于pageInfo对象调用属性：
	1、pageNum       当前页码
	2、pageSize         每页显示数据量
	3、size                 当前页的数据量
	4、total                总记录数
	5、pages              总页数
	6、`List<T>`          当前页的数据集合


==**工作原理**==：基于当前页(currentPage) 和 这一页显示的数量(pageSize)，自动在查询语句上拼接limit语句
```
sql limit x,y
	x  偏移量
	y  查询数量

page   第几页
pageSize  页容量

x = (page - 1) * pageSize
y = pageSize
```


#### PageBean结果类
基于PageInfo，将页码信息和查询数据封装到一个统一实体类中，便于JSON返回。
联想到统一返回结果R类  [[SpringBoot#2.R类]]

```
@Data  
@NoArgsConstructor  
@AllArgsConstructor  
public class PageBean<T> {  
    private int currentPage;  
    private int pageSize;  
    private long total;  
    private List<T> data;  
}
```

```
PageHelper.startPage(currentPage, pageSize)
List<Schedule> scheduleList = scheduleMapper.queryList();

// 查询到的结果注入PageInfo方法中
PageInfo<Schedule> info = new PageInfo<>(scheduleList);    

//基于info实例对象的6大属性，赋值仅PageBean实例
PageBean<Schedule> pageBean = new PageBean<>(currentPage, pageSize, info.getTotal(),info.getList());
```

R结果类示例代码
```
public class R {  
    // Field，初始化  
    private int code = 200;        //返回200状态码  
    private boolean flag = true;   //返回状态  
    private Object data;          //返回具体数据  
  
    // 完全没问题，把具体数据赋给data就行，其他两个属性就是默认值  
    public static R ok(Object data){  
        R r = new R();  
        r.data = data;  
        return r;  
    }  
  
    // 失败的情况  
    public static R fail(Object data) {  
        R r = new R();  
        r.code = 500;   //未获取到数据的状态码 500        r.flag = false;  
        r.data = data;  
        return r;  
    }  
  
    // getter和setter方法  
    public int getCode() {  
        return code;  
    }  
  
    public void setCode(int code) {  
        this.code = code;  
    }  
  
    public boolean isFlag() {  
        return flag;  
    }  
  
    public void setFlag(boolean flag) {  
        this.flag = flag;  
    }  
  
    public Object getData() {  
        return data;  
    }  
  
    public void setData(Object data) {  
        this.data = data;  
    }  
}
```

### MybatisX插件
MybatisX插件，基于数据库表，逆向生成Model、Mapper、Service+impl的工具，Controller不生成！

步骤1：IDEA中连接到DataSource，选中需要逆向的Table，按下图勾选。
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-05-31%20at%2023.10.48%402x.png)
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-05-31%20at%2023.13.59%402x.png)

步骤2：生成的代码会放到`generator`文件夹，逐个拷贝到对应本项目中去，然后删掉`generator`文件夹即可。

**********************
结语
mybatis是半自动，封装简单CRUD方法，复杂方法还需要mapper.xml手动写Sql语句
[[Mybatis_Plus]]是半自动向全自动迈进的体现

## mybatis源码阅读
mybatis 3.3.0
mybatis-parent 32

### mybatis执行过程
![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202407280051854.png)
