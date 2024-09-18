[尚硅谷JDBC实战教程（一套掌握jdbc，JDK17+MySQL8）](https://www.bilibili.com/video/BV1sK411B71e/?vd_source=72e3f029ff16aef4d56c5c73e18a983f)

[伟风老师的笔记](https://www.wolai.com/4SyaBmWpcQhVKae3oKjuNC)

| XX           | 版本     |                    |
| ------------ | ------ | ------------------ |
| jdk          | 1.8    |                    |
| JDBC         | 4.2    | 与JDK 1.8同步，JSR 221 |
| mysql        | 8.0.25 |                    |
| mysql-jdbc驱动 | 8.0.27 |                    |
| druid        | 1.1.21 |                    |

java与数据库连接的必要纽带！
Java  <---->  JDBC  <---->  数据库 ^bedd3c

JDBC是Java连接数据库的技术统称。是规范，是接口（各个数据库厂商提供实现类（jar包驱动），各种数据库驱动就是JDBC的实现类。

可以看到，Java程序与数据库连接时，发送连接时，协议就是jdbc
```
jdbc://118.50.24.90/ggkt
```

JDBC作为接口规范，存放在java.sql和javax.sql包中，只规范了方法命名、参数、返回值，不做具体的业务逻辑代码！
	java.sql 是Java SE标准库的一部分，用于基本的数据库访问。
	javax.sql是Java EE扩展API的一部分，提供了更高级别和更复杂的数据库访问功能。

[[Mybatis]](半自动)、Hibernate(全自动)、Spring Data JPA(@注解)、DbUtils(Apache Commons依赖包中)、JdbcTemplate都是JDBC的实现类！ ^ec256a

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/IMG_1249.jpg)
## 基本概念
Java Database Connection
在java代码中，使用JDBC提供的方法，**==发生字符串类型的SQL语句到数据库==**，并获取语句执行结果，实现对数据库的增删改查操作的技术。

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%202%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B42%E6%9C%8829%E6%97%A5%2016_51_33.jpg) ^6eac4b


## 核心类和接口
以下都是基于==**mysql-jdbc驱动**==针对JDBC的实现！

DriverManager
将厂商的jar包注册到java程序中（读取配置信息等）,可以获取connection

Connection实现类
与数据库建立连接，在Connection类的实例化对象上就可以执行数据库语句了，可以获取statment | preparedStatment | callableStatment

Statment
静态SQL

PreparedStatment
动态值SQL查询

CallableStatment
标准存储过程SQL

具体发送SQL语句到数据库软件中，PreparedStatment主要用这个
statment  静态语句，没有外部变量
preparedstatment   动态语句，有外部变量。

Result实现类
获取查询到的结果，除了上述三种statement会返回查询结果，其余都返回int

常用路线：DriverManager -->  Connection -->  PreparedStatment  -->  Result

## 基本步骤
### 1、注册驱动
DriverManager()
：依赖的jar包，进行安装
	注意：驱动版本 8+   com.mysql.cj.jdbc.Driver
	      驱动版本 5+   com.mysql.jdbc.Driver
```
DriverManager.registerDriver(new Driver());
```

问题，会注册两次驱动。最佳实践，就是利用反射机制来触发Driver类的静态代码块！
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-05-22%20at%2023.00.10%402x.png) ^48c513
```
// 注册驱动，避免两次注册问题，避免调用registerDriver方法，只new Driver触发内部static代码块  
// 静态代码块，会在类加载时触发  

// 方案1：触发两次驱动加载，浪费资源  
// DriverManager.registerDriver(new Driver());  
  
// 方案2:mysql驱动默认，写死了，不方便换驱动  
// new Driver();  
  
// 方案3:通过反射触发，将类模版信息提取到外部配置文件中，方便切换  
Class.forName("com.mysql.cj.jdbc.Driver");
```

^36259c

### 2、建立连接
Connection
参数1：URL     jdbc:数据库厂商名://ip地址:端口/数据库名
参数2：username
参数3：password
```
Connection connection = DriverManager.getConnection(
	"jdbc:mysql://118.25.46.207:3306/atguigu_jdbc",
	"root",
	"AmRI3@n2");
```

### 3、创建statement的对象
基于connection连接对象
#### createStatement()
```
Statement statement = connection.createStatement();
```

字符串拼接Sql，容易造成注入攻击！
```
String sql = "select * from t_user where account = '"+account+"' and password = '"+password+"' ;";

password处输入：or 1 = 1;  就能判断通过
```
#### prepareStatement
固定SQL语句结构，用？占位符，提前确定好语句结构
setObject的1、2指的是，占位符从左向右的顺序👉
然后setObject()方法进行映射

联想到[[Mybatis#简单类型 {id} 和 ${id}]]
![[Mybatis#^05dd8a]]

```
//4 创建prepareStatement  
String sql = "select * from t_user where account = ? and password = ? ;";  
PreparedStatement preparedStatement = connection.prepareStatement(sql);  
preparedStatement.setObject(1,account);  
preparedStatement.setObject(2,password);  
ResultSet resultSet = preparedStatement.executeQuery();
```

getMetaData()
可以获取列的元数据（包括列数，列名）
```
ResultSetMetaData metaData = resultSet.getMetaData(); //可以获取列的数量
```

metaData.getColumnLabel(i)
优先获取列的别名，没有别名才会获取列名。不建议用getColumnName

### 4、发送sql
让statment发送sql语句到数据库，并获取返回结果
```
String sql = "select * from t_user;";  
ResultSet resultSet = statement.executeQuery(sql);
```

#### executeUpadate()  
返回int，影响行数（非DQL的查询都行）

#### executeQuery()
返回ResultSet对象（DQL查询）

### 5、解析结果集
resultSet结果对象

下文是源码注释
```
Moves the cursor forward one row from its current position. A ResultSet cursor is initially positioned before the first row; the first call to the method next makes the first row the current row; the second call makes the second row the current row, and so on.
```
#### next()
resultSet.next()来移动光标，获取指定数据行，对应一个对象。配合while循环
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B45%E6%9C%8822%E6%97%A5%2023_56_05.jpg)

#### getString()
String clomnName   根据列名查询，或别名
int columnIndex     根据列ID查询，第一列是1、2、3

```
while (resultSet.next()){  
    int id = resultSet.getInt("id");  
    String account = resultSet.getString("account");  
    String password = resultSet.getString("password");  
    String nickname = resultSet.getString("nickname");  
    System.out.println(id+"--"+account+"--"+password+"--"+nickname);  
}
```


ResultSet存放在Map还是实体对象中？
实体对象更好，Map不好，因为Map没有数据校验、不支持反射；

### 6、销毁资源
关连接、释放statement对象、释放resultSet对象
由内到外关闭。
```
resultSet.close();  
statement.close();  
connection.close();
```

## JDBC --> Mybatis
mybatis内部封装了JDBC，那么Mybtis解决了JDBC哪些问题？
1、mybatis中集成数据库连接池，避免了频繁创建断开连接
2、解决了sql语句在java代码中，分离到mapper.xml中，解藕和
3、mybatis自动将java对象映射至sql语句
4、mybatis自动将sql执行结果映射至java对象


## 拓展
### 主键自增长回显
Order和OrderItem表，当Order插入一条数据后，需要获取自增长的id，用于OrderItem表中关联记录

查询的时候，才会返回resultSet，插入数据只会返回rows，插入成功，我想要自增长id，就得用此方法！

第二个参数，Statement.RETURN_GENERATED_KEYS = 1
```
PreparedStatement preparedStatement = connection.prepareStatement(sql,Statement.RETURN_GENERATED_KEYS);

// 这个结果集就是主键，一行一列， id=值  
ResultSet resultSet = preparedStatement.getGeneratedKeys();  
resultSet.next();  
int id = resultSet.getInt(1);
```

### 批量插入优化
1、mysql连接路径后添加参数
```
jdbc:mysql://118.25.46.207:3306/atguigu_jdbc?rewriteBatchedStatements=true
```

2、insert into values【必须带s】，语句不能添加分号，因为会自动追加
```
String sql = "insert into t_user (account,password,nickname) values (?,?,?)";
```

3、不是执行每条语句，而是addBatch()，执行追加；然后批量执行
```
//批量添加，批量执行  
preparedStatement.addBatch();  
preparedStatement.executeBatch();
```

### JDBC添加事务
1、前提条件，必须在一个connection中
2、事务的开启，在service层开启
3、connection的关闭，也在service层
利用try_catch实现事务
![[Spring FrameWork#^8ae14e]]
![[Spring FrameWork#^01e2e3]]
```
public void transfer(String addAccount, String subAccount, int money) throws SQLException, ClassNotFoundException {  
    BankDao bankDao = new BankDao();  
  
    Class.forName("com.mysql.cj.jdbc.Driver");  
    Connection connection = DriverManager.getConnection("jdbc:mysql://118.25.46.207:3306/atguigu_jdbc", "root", "AmRI3@n2");  
  
    try{  
        //开启事务  
        connection.setAutoCommit(false);   //关闭自动提交  
  
        //执行数据库动作  
        bankDao.addMoney(addAccount,money,connection);  
        System.out.println("------------");  
        bankDao.subMoney(subAccount,money,connection);  
  
        // 事务提交  
        connection.commit();  
    }catch(Exception e){  
        //事务回滚  
        connection.rollback();  
  
        throw e;  
    }finally {  
        connection.close();  
    }  
}
```


## 连接池
之前每次用完connection就销毁，浪费，引入连接池复用！提高使用效率
java.sql.DataSource接口，规范了连接池获取连接的方法；规范了回收连接的方法
```
DataSource = new 第三方连接池；     接口接值，没毛病！
```

常用连接池：DBCP、C3P0、Proxool、hikari、Druid（国货之光）

### Druid
1、创建一个Druid连接池对象

2、设置连接池参数 【必须 ｜ 非必须】
	必须：数据库驱动全限定符、URL、user、password
	非必须：初始化连接数量，最大连接数量（不推荐设置，用默认即可）

3、获取连接 【通用方法，所有连接池都一样】

4、回收连接【通用方法，所有连接池都一样】

#### ![[SpringBoot#Druid连接参数配置]]

#### 硬编码方式
创建Druid连接池的示例
```
DruidDataSource dataSource = new DruidDataSource();    // new一个Druid对象

dataSource.setUrl("jdbc:mysql:///studb");              // mysql数据库路径
dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");   //驱动，jar包的全限定符
dataSource.setUsername("root");                          // 账号
dataSource.setPassword("password");                      // 密码

dataSource.setInitialSize(5);                           //初始化连接数量  
dataSource.setMaxActive(10);                           // 最大连接数量
```

^4b07ab

Spring配置类中，使用相同步骤，配置Druid连接池
![[Spring FrameWork#^0b84ec]]

#### 软编码方式 properties
数据库的连接信息一般存放在 resources/jdbc.properties文件夹下 
![[Django-3#^fda326]]

jdbc.properties示例代码
```
jdbc.url = jdbc:mysql://118.25.46.207:3306/studb
jdbc.driver = com.mysql.cj.jdbc.Driver
jdbc.username = root
jdbc.password = p@ssw0rd
```

然后再在spring xml配置文件中使用`<context>`将jdbc.properties读进去。

可以加载多个配置文件，但是只支持读取.properties格式的文件
<context:property-placeholder location="classpath:jdbc.properties， classpath:……"/>
```
	<!--注意，这里会自动导入一个xml标签的约束  -->
    <context:property-placeholder location="jdbc.properties"/>
        
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">  
        <property name="url" value="${atguigu.url}" />  
        <property name="driverClassName" value="${atguigu.driver}" />  
        <property name="username" value="${atguigu.username}" />  
        <property name="password" value="${atguigu.password}" />  
    </bean>

```


Druid连接池 + JdbcTemplate类
由Spring Framework提供的模板类，简化JDBC操作

JdbcTemplate类提供的queryForObject方法中的参数==rowMapper==就是一个接口，需要手动实现具体逻辑。和我们手写Dao层代码的思路是一样的：
`public <T> T queryForObject(String sql, RowMapper<T> rowMapper, @Nullable Object... args)`

但是，它只简化了数据库的增删改查语句，它不知道和那个数据库连接，需要搭配Druid连接池使用。
![[#^4b07ab]]
```
JdbcTemplate jdbcTemplate = new JdbcTemplate();

//Druid连接池建立连接
jdbdTemplate.setDataSource(一个Druid连接池对象)

jdbcTemplate.update();            // 更新数据

jdbcTemplate.queryforObject();    // 查询单个对象
jdbcTemplate.query;               // 查询多个对象，集合
```

#### [[标准包.类#ThreadLocal类]]的运用
利用线程本地变量，存储连接信息Connection，确保一个线程中多个方法可以获取同一个connection
优势：事务操作的时候，service 和 dao属于同一个connection，不需要传参的方式把connection对象传来传去（太low），大家都可以调用getConnection()自动获取的是相同connection对象

```
public class JdbcUtils {  
    private static DataSource dataSource = null;  
      
    private static ThreadLocal<Connection> tl = new ThreadLocal<>();

	public static Connection getConnection() throws SQLException {  
	    //先查一下ThreadLocal  
	    Connection connection = tl.get();  
	    if(connection == null){  
	        connection = dataSource.getConnection();  
	        tl.set(connection);  
	    }return connection;  
	}
```

^ba30d5

#### BaseDao封装
封装DQL查询方法，提取成通用方法，用到了泛型+反射机制，很好的例子

下面的例子，就是封装BaseDao时，利用反射机制，利用（`Class<T> clazz`）类模版，给每个对象的属性赋值的过程，很好的反射实践！ ^e728fd
```
//遍历结果集ResultSet，把查询结果中的一条一条记录，变成一个一个T 对象，放到list中。  

while(res.next()){  
    //循环一次代表有一行，代表有一个T对象  
    T t = clazz.newInstance();//要求这个类型必须有公共的无参构造，反射机制实例化对象  
  
    //把这条记录的每一个单元格的值取出来，设置到t对象对应的属性中。  
    for(int i=1; i<=columnCount; i++){  
        //for循环一次，代表取某一行的1个单元格的值  
        Object value = res.getObject(i);  
  
        //这个值应该是t对象的某个属性值  
        //获取该属性对应的Field对象  
        //String columnName = metaData.getColumnName(i);//获取第i列的字段名  
        //这里再取别名可能没办法对应上  
        String columnName = metaData.getColumnLabel(i);//获取第i列的字段名或字段的别名  
        Field field = clazz.getDeclaredField(columnName);  
        field.setAccessible(true);//这么做可以操作private的属性  
  
        field.set(t, value);  
    }  
  
    list.add(t);  
}
```

^ff4d82
