跟着~~@Mosh~~ [@尚硅谷:康师傅的课程学习Java基础](https://www.bilibili.com/video/BV1PY411e7J6/?spm_id_from=333.999.0.0&vd_source=72e3f029ff16aef4d56c5c73e18a983f)
Java的语法，更加严格，考虑的方面更多（内存控制等）
Java语言和C语言很像，但是抛弃了指针，而是引用，并提供自动分配和回收内存空间。
Java 的运行速度随着 JIT(Just-In-Time）编译器技术的发展越来越接近于 C++

## 基本特性
**Java分为三个体系**：
Java SE (Standard Edition)  核心，提供基础Java语言和运行环境。打成JAR包即可运行。
Java EE (Enterprise Edition)  在SE基础上拓展，解决企业级分布式开发需求。
Java ME (Micro Edition)    标准版的子集，针对移动设备

**Java运行环境**：
- JDK    Java Development Kit   java开发工具包
	javac   编译工具
	java     运行工具
	jdb      调试工具
	jhat     内存分析工具

- JRE   Java Runtime Environment   java运行环境，包含jvm以及后面开发用到的核心类库。
- [[JVM]]   Java Virtual Machine  运行java的虚拟机

| Java    | JDK             |                |
| ------- | --------------- | -------------- |
| Java 8  | JDK 8 或 JDK 1.8 | 你发任你发，我用Java 8 |
| Java 17 | JDK 17          |                |
### 安装JDK环境
https://www.oracle.com/java/technologies/downloads/
目前我的Mac上装的是==JDK1.8 和 JDK17==，一个经典，一个最新！
/Library/Java/JavaVirtualMachines/jdk-1.8.jdk
/Library/Java/JavaVirtualMachines/jdk-17.jdk

jdk包含了jre，jre包含了jvm
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/jdk.png)
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/%E6%88%AA%E5%B1%8F2024-01-20%2000.42.33.png)

JAVA_HOME
一定要写入环境变量，对后续插件安装、整合第三方服务需要java环境都很必要！

windows设置JAVA_HOME环境变量！
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/IMG_2047.PNG)
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/IMG_2049.PNG)

Linux下安装JDK
1、将linux版的jdk压缩包，上传到：/usr/local/jdk-8u411-linux-x64目录下
2、在当前目录下，使用解压命令，解压
```
tar -zxf jdk-8u411-linux-x64.tar.gz
```
3、配置环境变量
```
vim /etc/profile

# set java enviroment
JAVA_HOME=解压路径
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME

source /etc/profile
```

4、至此就完成啦，java -verson检查

**Java程序执行过程**：
Source Code (.java)   ——>    Java Compiler(javac)  ——>    Byte Code(.class)  —（JVM基于Byte Code根据不同系统类型）—> 转为不同的机器码（这一步体现高可移植性）
	![[#^56a9c7]]

Java项目包结构
包命名，不能叫java,否则报错。
```
java.lang.SecurityException: Prohibited package name: java.advanced.exceptions
```

### Java与Python对比
- Python语法更简单，使用缩进表示代码块；Java语法更严格，使用花括号表示代码块，并且需要以分号结束每一行代码；
- Python是动态类型语言，变量在运行时动态更改（例如将变量转化为字符串`str(variable)`)；但是Java是静态类型语言，变量的类型在编译时必须明确，并且不能改变；
- Python使用垃圾自动回收机制，自动管理内存；Java需要通过new和delete关键字手动分配和释放内存；==Java的垃圾回收机制非常牛逼（招牌），python的自动垃圾回收机制非常垃圾！==
- python的装饰器（@Decorator）和java的装饰器（@Annotation）有点类似。python的装饰器更关注扩展函数功能，java的注解更关注添加元数据。
- Python是解释型语言，[[编程范式#过程式编程]]，依赖解释器逐行将代码解释成机器码运行；==Java是编译型语言，在程序运行前，事先全局编译成机器码（javac —> main.class）,再由java虚拟机（JVM）来执行==；编译型语言比解释型语言运行更快，更高效 ^56a9c7

- Java在8版本时，可以允许 [[JavaScript]] jjs，但是在Java17中给删除了。

在 Java 中，所有方法必须声明在类中。 Java 是一种面向对象的编程语言，所有的代码都必须位于类中。即使是静态方法（在 Python 中可以独立于类存在），也必须声明在类中。如果在类之外声明方法，Java 编译器会报错。 ^60064e

举个例子，在 Java 中，一个简单的类和方法可以像这样定义：

```
public class MyClass {

    public void myMethod() {

        // 方法体

    }

}
```

在这个例子中，myMethod 方法是声明在 MyClass 类中的。在 Java 中，所有的方法都必须跟随类的声明。

### 编译语言与解释语言对比
1、源代码在编译时会进行静态优化和转换（JIT）；而解释型语言在运行时动态解释和执行代码，通常会有一定性能损耗
2、编译型语言在第一次运行时要进行编译（需要一定时间），之后就不用编译了，直接读编译完成的.class文件）；解释型语言虽然每次都可以直接运行，但是每次都需要给解释器解释
3、编译型语言语法更严格，减少运行时错误
4、编译型语言通常具有更好的平台依赖项；解释型语言需要依赖解释器，需要在每个平台都安装相应的解释器来执行代码 ^a2fdb3

**JIT**  Just-In-Time，广泛运用在解释型语言中，是解释型语言向编译型语言学习的优化技术。传统解释型语言不实现编译，在运行时逐行转换。JIT技术可以事先将热点代码编译成机器码，提升解释型语言的运行效率！
[python潮流周刊35：Python 3.13 gets a JIT](https://tonybaloney.github.io/posts/python-gets-a-jit.html?utm_source=substack&utm_medium=email) ^059118

Java虽然是编译型语言，但是也用到了JIT，在JVM中需要将字节码转换为机器码的过程中，用到了JIT技术！Java 的运行速度随着 JIT(Just-In-Time）编译器技术的发展越来越接近于 C++。

[[标准包.类]]
```
├── java
	├── .lang
	  	├── .Math        数据运行（向上取整、取余）
	├── .util
	  	├── .Date         日期
	  	├── .Arrays       操作数组的类
	  	├── .Scanner	  从用户输入中获取数据(例如从Terminal)
	├── .text
	  	├── .NumberFormat      
```


==TODO==
对象的比较
在Java中，对象的比较可以通过实现`Comparable`接口来完成。
在python中，则需要在类中定义魔法方法！

## IDEA开发工具
[[#IDEA小技巧]]
[[SSM#IDEA目录结构]]
### 创建IDEA传统工程
最纯洁的项目目录结构，区别于Maven的目录结构
新建项目的时候，如果不选择Maven构建工具，创建的目录就是以IDEA构建的方式 ^510575

联想到：[[SSM#Maven工程]]

IDEA编译到==out==文件夹，Maven编译到==target==文件夹
```
├── idea-project
	├── src         # 源代码
	├── out         # 编译输出class字节码文件
	├── lib         # 第三方Jar包，依赖库，CV进来后，右键Add to Library
```

Project Name：HelloWorld
src中右键/新建Java Class：me.shen1993.Main
	me.shen1993   表示Package名
	Main                表示Class类名


重要：新建项目的时候，==千万不要点“create git repository”==，这样会把IDEA生成的隐藏文件都纳入git管理，会非常麻烦。不勾选，进入项目，加入ignore规则后（手动排除那些不必要的文件），再手动生成git仓库，完美！ ^3b799a

### 导Jar包/库
用原生IDEA构建项目，引入依赖（jar包）非常麻烦，要手动去网上找jar包，存放到lib文件夹中。
这时候就体现出Maven构建项目的便捷了！，pom文件写一行dependecy就行了！方便

2024.8.1 在juc-part中需要导[[JVM#JOL]]把这个问题彻底解决，彻底搞清楚了！
1、普通IDEA项目里，建一个专门放类库的文件夹，lib
2、参考maven的`<dependency>`去maven中央仓库下载Jar包
3、拷贝到lib目录里
4、Project Structure - Modules -juc-part - Dependencies - （Add JARs）选中lib中的JAR包
搞定！


当前项目：

### 类路径 / 工作目录
类路径，就是target目录，JVM编译完成后，用来查找`.class`文件的路径
![[SSM#^596470]]

工作目录：任何相对路径都是从此处开始解析的！

### Debug模式
注意：
1、DEBUG的时候，一定不要动代码！不要改注释的行数，一改的话，蓝色执行条就错位了！后续打断点也都错行了，草！
2、尽量不要把断点打在方法名上，否则容易项目启动不起来！

F7   进入方法内部
F8   单步运行
F9   大步快走（运行至下一个断点处）
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-07-25%20at%2010.44.11.png)

1、打断点
2、Run / Debugger main.java（会在断点处停住）
3、Step Over 表示执行下一部
    Step into 表示进入该函数中
4、调用函数的栈视图
5、变量当前的值视图！

**1、Compile-time Errors**
（或者称为：syntax Errors 语法错误），会在编译的环节报错，根本编译不成功，这种错误比较简单，IDEA会自动发现并提示！

**2、Run-time Errors**  
就是语法都是正确的，编译通过，但是运行时出现的错误！需要借助Debugger工具解决（思路就是：逐行运行，看在哪一行出现错误）

### IDEA小技巧
- IDEA构建到out文件夹，Maven构建到target文件夹
用原生IDEA构建项目，引入依赖（jar包）非常麻烦，要手动去网上找jar包，存放到lib文件夹中。
这时候就体现出Maven构建项目的便捷了！，pom文件写一行dependecy就行了！方便

- shift + F6     一次性重命名变量名；右键/refactor重构
- 选中一个被调用的方法，ctrl+b，跳转到函数声明处。
- 选中一个方法，或一个Expression，Refactor this，进行重构！
- cmd + N : 快速插入constructor、overrider、getter&setter等
- lombok插件
	@Data注解方式(加在类上！)插入getter、setter、toString
	@AllArgsConstructor  自动生成一个全参构造函数，免去手动编写构造函数
	@NoArgsConstructor
- 利用IDEA快捷键，flip equlas()，实现将字符串判断相等，进行反转避免空指针异常。 ^3ee420
- 利用IDEA快捷键，introduce local variable，自动补全变量声明（Alt + Enter)

[lombok全部注解-官方文档](https://projectlombok.org/features/)
- 【阅读源码，了解调用/继承链】选中一个方法，Navigate-Type Hierachy，表示类和接口之间的继承链。例如：补图
Method Hierachy  类中方法之间的依赖关系，包括方法重写、方法重载、方法调用
Call Hierachy    方法的调用与被调用关系。
[@鲁班大叔：源码阅读网](https://coderead.cn/v2/) 提供可视化的UML时序图调用链（不过方法不太全）
- settings-editor-code and templates 自定义模板，例如xml的约束头！
- package可以用点创建多层结构，directory用斜杠创建多层结构！ ^0b9096
- 安装：阿里巴巴代码规范检查插件："Alibaba Java Coding Guidelines"，提供基于《阿里巴巴-Java开发手册》的代码审查，初级阶段多用用，养成良好的代码规范
- 后续开发过程中，需要临时写一个测试用例，就在JavaSE-part/test/Test_Java这个类中去写，这个项目天生具备JUnit环境，@Test就能马上执行！不用在一个main方法了来回折腾！这是一个好习惯！
- mac模式下，control + option + j   多行合并成一行，mysql语句有用！
- mac下，ctrl + option + o    优化导包，删除没用的导入

### Live Template

| 快捷键           | 模板代码                                                       |
| ------------- | ---------------------------------------------------------- |
| mycurr        | Thread.currentThread().getName()                           |
| tus           | TimeUint.SECONDS.sleep(1);                                 |
| newthread     | new Thread(() -> {  <br>    //具体业务逻辑  <br>},"t1").start(); |
| newthreadPool |                                                            |
| pvt           | @Test  <br>public void test01() {  <br>      <br>}         |

### 插件收集
- Nyan Progress Bar  进度条变成彩虹小猫
- Sequence Diagram   源码调用关系的时序图
- GenerateAllSetter插件，调用该对象的所有setter方法，减少重复操作
![[编程思考#^a0d0f1]] ^c62eaf


********************************************************************
[[Java_02_面向对象]]

[[Java_03_高级特性]]



## 基本原则
Pascal 帕斯卡命名规则      单词的第一个字符大写
comel  驼峰命名规则         除了第一个字符，其余单词第一个字符大写

| 名字 | 原则 | 示例 |  |
| ---- | ---- | ---- | ---- |
| 类名 | Pascal 帕斯卡命名 | HelloWorld | 每个单词首字母都大写 |
| 方法名 | comel 小驼峰命名 | helloWorld | 除第一个单词之外，其余单词首字母都大写 |
| 变量名 | comel 小驼峰命名 | now |  |

### 程序入口 main()
IDEA只能从~~Main类~~，main方法启动编译过程，~~其他的类里都是不能运行的~~。不包含main方法的代码是不能运行的。这一点和python有所区别，python可以在任何地方单独运行。

理解1：看做是一个普通的静态方法
理解2：看做是一个程序入口，格式是固定的。

注意：main()方法不一定要在Main类中，可以在自己写的类中，也是可以启动程序的。这一点我之前理解错误了。我以为只能在Main类中启动，其实应该是包含main()方法。

作为Java程序的主入口main方法体现了单例模式。声明为static，是为了方便主程序启动，不用实例化，直接`Main.mian()`就能启动了。 ^f9c866

[[设计模式#单例模式]]

```Main
class Main {
	public static void main(String[] args) {  // String[] args是用于从命令行获取形参
		// 具体代码
	}
}
```

^4d4cb0
### 参数
#### 形参与实参
形参： 就是形式参数，用于定义方法的时候使用的参数，是用来接收调用者传递的参数的。
实参： 就是实际参数，用于调用时传递给方法的参数。实参在传递给别的方法之前是要被预先赋值的。
![[标准包.类#^3f6a72]]

值传递： 是指在调用方法时，将实际参数拷贝一份传递给方法；值传递，传递的是副本
引用传递：是指在调用方法时，将实际参数的地址传递给方法；引用传递，传递的是实际内存地址

Q：Java中参数传递是传值，还是传引用？ #Java八股/JVM 
A：值传递
[Java到底是值传递还是引用传递？](https://heapdump.cn/article/4859141)
深入剖析：
情况1：基本数据类型（那八种），直接拷贝一份，值传递，传进去的改了，原本的是不变的
情况2：引用数据类型（Reference），值传递，这个值是指对象的引用（那个堆内存的地址0xab），不是对象本身。
如果这个引用所指的对象进行任何改动，都会影响原始对象的状态；
但是如果在方法内部改变了引用（new了一个新对象），指向了新的对象，那改变则不会影响原始对象（因为原先的引用断了）

#### `...`表示可变形参 
用三个点表示可变参数（varargs），允许方法接受零个或任意多个同类型的参数。

```
public void calculateSum(int... numbers) {
    int sum = 0;
    for (int num : numbers) {
        sum += num;
    }
    System.out.println("Sum: " + sum);
}

calculateSum(); // 不传递参数
calculateSum(1, 2, 3); // 传递多个参数
calculateSum(10, 20, 30, 40, 50); // 传递多个参数

Sum: 0
Sum: 6
Sum: 150
```

### 修饰符
修饰符用来定义类、方法或变量，通常放在语句最前面。
#### 访问控制修饰符
保护对类、方法、变量的访问
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/e89f2d864d2f189e82e62bd876d7e2a9.PNG)

Mosh建议：protect和package- private都让人comfuse，远离它们！**==只用public和private==**！
public that should be exposed outside of the class！
private that hiding the implementation detail（未来如果要修改的话，不影响其他类）

| 访问控制修饰符             | 包下的权限                 | 子类下的权限                                      | 方法                                       | 备注                   |
| ------------------- | --------------------- | ------------------------------------------- | ---------------------------------------- | -------------------- |
| public              | 同一个包内、其他包：都能使用        | 子类可以使用                                      | public修饰的方法，通过类的实例化对象，可以调用               | 默认是public类，作为外部类对外暴露 |
| private             | 自己类内部可见；同一包内其他类也不能使用。 | 不可继承给子类                                     | private修饰的方法，只能在类内部this.方法来访问<br>类外部访问不了 | private类只能作为内部类      |
| protected           | 同一包内都能使用，<br>其他包不能使用。 | 子类继承自其他包，<br>可以使用（（在父类的package的class Main中） |                                          | 可见性是：==同一包 、 所有子类==  |
| package-private（默认） | 同一包内可使用，<br>其他包不能使用。  | 子类不能通过继承使用                                  |                                          |                      |
|                     |                       |                                             |                                          |                      |
protected 是最难理解的一种 Java 类成员访问权限修饰词！
[菜鸟教程：Java protected 关键字详解](https://www.runoob.com/w3cnote/java-protected-keyword-detailed-explanation.html)
[CSDN：Java访问权限控制：你真的了解protected关键字吗？](https://blog.csdn.net/justloveyou_/article/details/61672133)

protected权限的一个应用场景：接口中利用抽象类实现同一接口上所有类共用的方法。[[Java_02_面向对象#接口的新（糟糕的）特性]]



经验：
1、一般utils工具类，都声明为public static，通常也不需要@Component加入ioc容器，因为可以直接通过类名调用。
2、一个java文件，只能有一个public修饰的类！
3、新建一个class，至少是public权限（因为对外要用），**private只能用来修饰内部类**！！

#### 非访问修饰符 - 关键字
| 类、方法和变量修饰符   | 说明                                                                                                                                                      |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| static       | 静态变量、静态方法   ==与之对应，默认是instance变量，实例变量==                                                                                                                 |
| final        | 修饰类：==表示不能被继承==，代际传承到最底部了，最后一代！<br>修饰方法：不能被子类重写<br>修饰变量：常量，全部大写PI       `final float PI = 3.14F;`<br>修饰参数：向匿名内部类传递数据（没懂？？）                              |
| abstract     | 创建抽象类和抽象方法，抽象类无法实例化，使用Factory Method来生产实例对象 [[Java_02_面向对象#抽象类/方法 abstract关键字]]                                                                         |
| synchronized | 线程、同步 [[Java_03_高级特性#线程同步机制2：Lock锁]]                                                                                                                    |
| transient    | 加到Field上，在实现Serializable接口的类中，加该关键字的Field将不被序列化；static也不会被序列化  [[标准包.类#Serializable]]<br>案例：<br>[[标准包.类#GenericServlet 抽象类]]中的Field config就是transient修饰 |
| volatile     | 保证：可见性、有序性（禁止指令重排）；不保证原子性<br>一般运用在多线程场景下；<br><br>案例：<br>[[标准包.类#线程中断机制]]中利用volatile关键字修饰的中断标识位，让不同线程更新状态                                                |
| native       | 调用本地C语言函数库                                                                                                                                              |

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/cb91e1d7e69552d0c1db2bb3663a03bf.PNG)

#### 返回类型
```
public static int rank(int key, int[] a) {}
               ^

//这里声明了返回类型必须是int
```

void
声明返回类型为void的static方法，可以不用写return，运行完就结束（悄无声息）

![[#^fd378a]]
一个Java方法只能返回一个值，它的类型是方法签名中声明的返回类型。

#### 声明类/方法的基本格式
```
public   static        <T>            T        myMethod() { }
访问权限   关键字  泛型声明(如果是的话)  返回类型    方法名(参数)
```

^f06487

      泛型声明(如果是的话)  返回类型  方法名 ^fa7e02

### 命名规范
#### 类的命名规范
Pascal方法：每个单词首字母都大写
`public class Main() { }`        ==Main就是类命名==
#### 类下方法的命名规范
小驼峰方法：除了第一个单词，其余单词首字母都大写
`public static void main(String[] args) { }`     ==main就是方法命名==
#### 变量命名的规范
小驼峰方法：除了第一个单词，其余单词首字母都大写
`Date now = new Date();`    ==now就是变量命名==
#### 注释
单行注释    //

多行注释  `/*    */`
```
/* 这是第一个Java程序 
   它将输出 Hello World 
   这是一个多行注释的示例 */
```

文档注释
这种注释可以被工具提取并生成API文档，一般写在fields、methods、class的声明前面。
特定的标签：
	@param 用于描述方法参数
	@return 用于描述返回值
	@throws 用于描述可能抛出的异常
```
/** 
 * 这是一个文档注释示例 
 * 它通常包含有关类、方法或字段的详细信息 
 */ 

public class MyClass { // 类的成员和方法 }
```


主方法入口：所有的 Java 程序由 **public static void main(String[] args)** 方法开始执行

## 数据类型
Java是强类型语言，每个变量前必须明确是什么类型（python是动态语言，并不会标注每个变量是什么数据类型，要查看的话得用 type(变量)来查看 ^fd378a

数据类型  变量名  = 值 ;
```
int i = 10;

User user1 = new User();   User是引用类型
```

### 基元类型
Primitive Type
中文翻译为：内置数据类型，或原始数据类型
存储简单的值，只有下表这8种！
注意：字符串不是基础类型，是引用类型。由char构成

向上转向的等级：==**byte > short > int > long > float > double**==

| Type           | 默认值      | Bytes      | Range                              | 描述                                                                    |
| -------------- | -------- | ---------- | ---------------------------------- | --------------------------------------------------------------------- |
| byte           | (byte)0  | 1byte = 8位 | [-128,127]                         | 表示8位有符号整数  2^7<br>==jdk 9 单个字符用byte数组！==                              |
| short          | (short)0 | 2          | [-32768,32767]                     | 表示16位有符号整数  2^15                                                      |
| **==int==**    | 0        | 4          | [-2_147_483_648  ,  2_147_483_647] | 表示32位有符号整数  2^31                                                      |
| long           | 0L       | 8          | [-2^63 , 2^63-1]                   | 表示64位有符号整数  2^63；声明的时候需要加L，long a = 10000L                            |
| float          | 0.0F     | 4          |                                    | 表示32位浮点数 float a = 234.5f                                             |
| **==double==** | 0.0      | 8          |                                    | 表示64位浮点数 double a = 0.0d                                              |
| boolean        | false    | 1          | true / false                       | 默认值是 false；                                                           |
| char           | \u0000   | 2byte      | A,B,C,…                            | 单个字符，构成String，用单引号<br>内存中，1个char = 2个字节<br>==jdk 8 单个字符用char数组！==<br> |

注意：所有引用数据类型的==**默认值都是null**==，在null的基础上调用方法，会报**NullPointerException**

- 基元数据类型只在Stack（[[数据结构_栈]]）上开辟内存空间，并在上面存放实际的值！ ^3916a6
- 基元数据类型，不用new关键字来实例化对象，直接赋值。
	- 解释：因为Heap上存的是new出来的对象，但是Primitive Type只存在于Stack，不涉及Heap，所以不用new关键字！也没有可以调用的属性或方法（因为它根本就不是类，只有类class下才有方法）

基本类型，只要声明一个，就会在栈上产生一个，那怕值相同
引用类型的复制，是有引用的内存位置来复制的；
基本类型的复制，是通过值进行复制的。变量在不同的内存地址，值与值之间完全独立的。
```
例子1
public static void main(String[] args) {
	int myAge = 30;
	int herAge = myAge;    // herAge变量在Stack内存空间开辟了新的内存地址！值都是30
}

例子2
public class Main {  
    public static void main(String[] args) {  
        byte x = 1;  
        byte y = x;  
        x = 3;  
        System.out.println(y);  
    }  
}

>> 输出：1     //表示x和y是内存中两个独立的位置。
```

```
public class Main {  
    public static void main(String[] args) {  
        byte age = 30;  
        long viewsCount = 3_123_456_789L;   
        // 默认识别为int,需要在大数字后面加上L（表示long）声明  
        float price = 10.99F;               
        // 默认识别为double，需要在小数后面加上F（表示float）声明  
        char letter = 'A';                  
        // 单引号用于单个字符；双引号用于多个字符；强制规定！  
        boolean isEligible = false;  
    }  
}
```

常量
```
final float PI = 3.14F;  //常量 
```

### 引用类型
Reference Type
三种引用数据类型：类、接口、数组、枚举、注解……

- 除了上表中8种数据类型之外，其他所有Java数据类型，都是Reference Type。
- 在Java中，引用类型的变量非常类似于C++语言中的指针。引用类型指向一个对象。变量一旦声明后，就无法改变。
- 对象、数组都是引用类型
- 所有引用类型的默认值都会null
- Heap（[[数据结构_堆]]）上存放基于类实例化的对象（new关键字声明方式）。Reference Type在Stack（[[数据结构_栈]]）上开辟内存空间，并存放Heap中对象的内存地址。Stack中存的不是实际值，而是应用Heap的内存地址。 ^fa8365

对比两种情况：
```
Point point1 = new Point(1,1);  
// 这里只new了一个实例，在Heap中只生成一个内存地址
Point point2 = point1;    // point2其实是把存放Heap内存地址的值传递给point2

>>输出的内存地址是一样的！
```

```
var point1 = new Point(1,2);     // 两次分别new，将创建两个独立的对象
var point2 = new Point(1,2);     // 在Heap内存地址完全不同。
System.out.println(point1);  
System.out.println(point2);  
System.out.println(point1 == point2);  // 引用类型数据存放的值是Heap内存地址，是不同的

>>输出：
>>me.shen1993.Point@7a81197d
>>me.shen1993.Point@5ca881b5
false
```


- 引用类型的变量需要通过new关键字来实例化对象（基本类型不需要），并且可以调用对象的属性和方法
```
public class Main {  
    public static void main(String[] args) {  
        Date now = new Date();
        // 第一个Date，变量类型的声明
        // now是变量名
        // new Date() 通过new关键字，基于Date()类实例化对象
        System.out.println(now);  
    }  
}
```

注意：引用数据类型之间使用[[#比较运算符]]，比较的是存放Heap内存地址的值是否相同，如果new了两次，将会在在Heap内存地址中生成两个内存地址（存放实例对象），那么用比较运算符`==`将返回False，原始的`Object类.equal()`也是一样的效果。所以我们需要@Override重写equal()方法。 ^236180

```
Scanner scanner = new Scanner(System.in);  
String input = "";  

// 情况1
while (input != "quit") {  
    System.out.print("Input:");  
    input = scanner.next().toLowerCase();  
    System.out.println(input);  
}

//情况2
while (!input.equals("quit")) {  
    System.out.print("Input:");  
    input = scanner.next().toLowerCase();  
    System.out.println(input);  
}

情况1：比较的是两个字符串对象的引用。这里比较的是input变量是否指向了内存中的字符串“quit”字符串对象。加入input指向了另一个字符串“quit”，及时字符串的内容都是quit，也会返回True。

情况2：调用equals()方法，直观地，比较字符串的文本值是否相同。这个才是正确的用法。

```
所有引用数据类型的==**默认值都是null**==，在null的基础上调用方法，会报**NullPointerException**


### 包装类
基元类型 -包装成-> 引用类型
```
// int  ->  Interger类
// float  -> Float类
// boolean   -> Boolean类
```

^5aca87

![[Java_02_面向对象#^0524d7]]

实体类的Field都尽量用包装类接值，因为可以容纳null，基本数据类型不能容忍null
```
@Data  
public class MergeVo {  
	//整单id，用Long声明属性，可以接收null，用基本数据类型，就无法容忍null  
   private Long purchaseId; 
   private List<Long> items;   //合并项集合  
}
```
#### [[标准包.类#包装类通用方法]]
#### String类
char -包装成->  String类
char是基元类型，多个char组成字符串。String表示的是引用类型。

注意：String类在Java中都是不可变的，一旦new创建了一个String实例对象，就无法修改它的内容（线程安全，提供内存利用率，安全性）。
下述所有调用的方法产生的改变，本质上都是new创建一个新的String实例对象。

```
char data[] = {'a', 'b', 'c'};     // 字符数字
String str = new String(data);     // 多个字符，拼装在一起

String类的构造方法，就是把char组合拼在一起！

```

单个字符Char（内置数据类型）用==单引号== ' ' 来包裹！
多个Char组成的字符串必须用 ==双引号==“ ” 来包裹

```
public class Main {  
    public static void main(String[] args) {  
        String message = new String("Hello World");    
        // 这是规范，但是 is redundant，经常用，所以简化为下面  
        String message = "Hello World";                       
        // 简化了，但不代表是基本类型，本质还是引用类型  
    }  
}
```

String可以用加好连接两个字符串
```
String message = "Hello World"+"!!";
System.out.println(message);

>>输出：Hello World!!
```

基于类的实例（message），调用各种方法
```
String message = "Hello World"+"!!";
System.out.println(message.endsWith("!!"));   //基于message这个实例对象，调用endwith方法
```

JDK9 中 String的实现，由`char[]` 数组 修改为 `byte[]`数组。
因为，一个char=2个字节，英文字母都只占1个字节，浪费空间！

##### 独有的方法
[[标准包.类#String类]]  这里罗列了包装类通用方法的String类版本

#### Integer类
int -包装成->  Interger类
包装类，封装基元类型中的int的值，提供常用方法来处理和装换整数。

```
public void test5() {
	Integer i1 = 10;
	Integer i2 = 10;
	sout(i1 == i2);        // true

	Integer i3 = 128;
	Integer i4 = 128;
	sout(i3 == i4);        // false

}
```
因为，Integer包装类中有一个IntegerCache缓存类，在-128-127之间的话，都引用数组中的对象，超过范围每次都new一个新的对象，`==`判断索引位置，所以不同！
##### 独有的方法
[[标准包.类#Integer类]]   这里罗列了包装类通用方法的Integer版本


包装类对象的缓存问题
凡是在下表范围内的，用`==`都是true，超过范围了，都是false；

| 包装类      | 缓存对象         |
| -------- | ------------ |
| Byte     | -128~127     |
| Short    | -128~127     |
| Integer  | -128~127     |
| Long     | -128~127     |
| Float    | 没有           |
| Double   | 没有           |
| Charater | 0-127        |
| Boolean  | true 和 false |

### 数组
是多个==同种数据类型==的值的集合。

Q：为什么数组要求必须同种数据类型，但是链表的每个元素可以不同？
A：数组元素必须是相同类型的，这样才能通过计算偏移量来获取对应元素位置。例如，数组同时包含 int 和 long 两种类型，单个元素分别占用 4 字节 和 8 字节 ，此时就不能用以下公式计算偏移量了，因为数组中包含了两种“元素长度”。 ^b2dae7
```
元素内存地址 = 数组内存地址（首元素内存地址） + 元素长度 * 元素索引
```

Q：ArrayList和Array（数组  `[]`）的区别吗？
A：
 Array       ->  `[]` 
ArrayList  ->  List接口的实现类，也就是列表（动态数组）
Arrays类，工具类，提供各种静态方法，不能实例化[[标准包.类#Arrays类]]

**数组[ ] 和ArrayList区别如下**：
ArrayList内部基于动态数组实现，比数组[ ]更加灵活
ArrayList可以动态扩容缩容，数组[ ]被创建之后不能改变长度
ArrayList允许使用泛型（`List<Strign> list = new ArrayList<>()`）,数组[ ]不行
ArrayList中的元素只能存放对象，不能使用基础数据类型，必须通过包装类处理，数组[ ]可以存放基础数据类型
ArrayList支持插入、删除、遍历等操作，丰富API；数组[ ]只能通过下标访问
ArrayList创建时不需要指定大小，数组[ ]创建时必须指定大小（`a = new double[3]`） ^404672
#### 基本数组`[]`
使用`[ ]`声明数组
`[ ]`是一种语法，是引用数据类型！用于表示数组类型，用于定义数组变量的一种方式

最常用的声明数组的示例
```
int[] nums = new int[3];   # 基本数组，必须指定长度
```

```
double[] a;        // 声明数组
a = new double[N]   // new一个N长度的数组
for (int i = 0; I < n; i++)
	a[i] = 0.0;                // 利用循环，初始化数组

简化写法
double[] a = new double[N]     // 声明+new,但不初始化

最常见的写法
double[] a = {0.0, 0.0, 0.0}   // 声明+new+初始化赋值

double[] a = new double[]{0.1, 0.2, 0.3}  //这里的{}本质上就是代码块
```

^8b58fc

声明，但是不new，未指定长度（heap内存空间中还没有）
```
int[] numbers1;
```

声明+new，指定长度，但是栏位是空（heap内存空间中开辟了，但是未存值）
```
int[] numbers2 = new int[5];    // 指定长度，但是栏位是空
```

声明+new+初始化值
==类型推断==，可省略{ }前面的数据类型 ^eb9d8f
```
int[] numbers3 = new int[]{1,2,3,4,5}  // new int[]可以省略，类型推断！
int[] numbers3 = {1,2,3,4,5}    // 初始化，且赋值
```

##### 常用属性
###### .length
返回数组的长度。不是size，也没有方法size()
注意区别于 String类中的`.length()`方法 ^424238

`a[i]`   表示索引第i+1个值。

注意点：
- 如果只声明，未指定长度，需要在后续代码中分配内存空间或使用已有数组来初始化。
- 在Java中，数组的长度是固定的（连续存储空间），一旦创建好之后，就不能增加或删除！不像其他编程语言，是动态数组！

##### 常用方法
1. 访问元素：通过索引（或下标）访问数组中的单个元素。
2. 修改元素：通过索引修改数组中的单个元素的值。
3. 迭代（遍历）：按顺序访问数组中的每个元素。（利用for循环遍历索引）
4. 查找：查找数组中指定元素的索引或是否存在某一元素。
5. 插入：向数组中指定位置插入一个元素。
6. 删除：从数组中删除指定位置的元素。

打印数组
```
System.out.println(Arrays.toString(nums));
```

#### [[Java_03_高级特性#【实现类】ArrayList类]]
针对基础数组`[N]`的包装类，可以动态加入元素，更加灵活！


![[标准包.类#^c3031a]]
![[Java_03_高级特性#^851e26]]


#### 操作数组 Arrays类
==Arrays类只是提供操作数组的方法。== ^74d716

[[标准包.类#Arrays类]] ^e9d20e

##### 常用方法
- toString()    字符串形式，打印
- equals()       判断是否相等
- sort()           排序
- binarySearch()   二分查找
- copyOf(数组，长度)     拷贝一个数组，并指定拷贝的长度
- asList()     将常规数组转换为==List对象==，建立常规数组和集合之间的桥梁，可以利用集合的方法来操作数组元素。需要注意：
	1、并不是ArrayList对象，而是Arrays内部类
	2、固定长度，不能==增加、删除==
```
List<Long> ids = new ArrayList<>();    
ids.add(1L);    
ids.add(2L);      # 可以动态添加
  
List<Long> ids = Arrays.asList(1L,2L,3L);   # 长度固定，不能添加

```
	3、可以利用List的特性进行==获取、修改==元素！
```
String[] string = {"A","B","C"};

List<String> list = Arrays.asList(string);   //注意，用List去接值
```

常规数组，利用Arrays类提供的静态方法的示例！
```
int[] nums = {5,2,9,1,3};

Arrays.sort(nums);
System.out.println(Arrays.toString(nums));
```

## 变量的作用域
### 局部变量
==**方法内的变量！**==
只在=={ }==代码块中生效！
在（构造）方法中定义的变量。方法结束后，局部变量自动销毁
- 局部变量在方法被执行的时候创建，当它们执行完成后，变量将会被销毁。
- 局部变量只在声明它的方法、构造方法或者语句块中可见，不能被其他方法或代码块访问
- 局部变量没有默认值，所以局部变量被声明后，必须经过初始化，才可以使用（如果在声明时未初始化，变量将被赋予默认值）。
```
public class ExampleClass {
	public void exampleMethod() {
	    int localVar = 10; // 局部变量，在方法中定义的变量
	    // ...
	}
}
```
### 成员变量 = Field
**==属性值！==**
定义在类中，方法体之外的变量，成员变量在创建对象的时候实例化，成员变量可以被类中的方法访问到。
- 当一个对象被实例化之后，每个成员变量的值就跟着确定
- 成员变量在对象创建的时候创建，在对象被销毁的时候销毁
```
public class ExampleClass {
    int instanceVar; // 实例变量,在方法体之外的变量
	public void exampleMethod() {
	}
}
```

### 静态变量
[[Java_02_面向对象#静态变量 static field]]
也叫做：类变量，是属于类的变量，而不是某一个具体实例的。即无论创建多少个类实例，静态变量在内存中只有一份拷贝，被所有实例共享。

也是在类中，方法体之外，但是必须声明为static，属于类而不是实例，所有该类的实例共享一个类变量的值（有点类似于：群体意识，民族共识的概念）


## 运算符 / 二元操作符
### 算数运算
加、减、乘、除
```
public class Main {  
    public static void main(String[] args) {  
        int result = 10 + 3;  
        System.out.println(result);  
    }  
}
```

其中除有点特殊
```
public class Main {  
    public static void main(String[] args) {  
        int result = 10 / 3;  
        System.out.println(result);  
    }  
}

>>输出：3   因为两个int值，返回的值也是int。

public class Main {  
    public static void main(String[] args) {  
        double result = (double)10 / (double)3;   // Explicit casting
        System.out.println(result);  
    }  
}

>>输出：3.3333333333333335
```

递增 ++
==注意：i++拆解开是三步合并的，**i++是非原子操作**！多线程情况下，不可靠！必须要加锁！==
 i++在指令上是三步：
	 1、拿到i的值；
	 2、对i+1操作
	 3、在写回i

![[JUC#incrementAndGet()]]

思考：为什么i++在多线程环境下，是线程不安全的？
因为i++操作是由三个步骤组成的，在多线程并发执行的时候会出现竞态条件。因为JMM规定，当线程要操作主内存中数据是，需要将主内存数据拷贝到线程中生存一个副本进行操作，然后再写回主内存。多线程环境下，线程A去主内存中拿到了i，去执行+1操作，还没完成的间隙，线程B也拿了之前的i作为副本回去，也+1操作，导致都是在同一个旧的i上+1，线程B相当于浪费了一次+1的机会，因为值和线程A的+1后结果是一样的，最终导致结果是错误的！
这种情况下，应该使用AtomicInteger类的increment()方法或在i++所在方法上加synchronized关键字解决！ ^a2d163
> 注意：虽然使用volatile关键字修饰i，线程A一更新马上写回主内存，这样就能保证后面的线程B拿到的是最新的结果，但是由于volatile不保证原子性，所以还是无法保证结果安全！

i++ 与 ++i 的区别，示例代码
总结：当存在赋值时，当++在后缀，则先赋值，再递增；如果++为前缀，则先递增，再赋值；
```
public class Main {  
    public static void main(String[] args) {  
        int x = 1;  
        ++x;       
        // x++ 和 ++x在单个值时是一样的结果，但是在赋值情况下有区别  
        System.out.println(x);  
    }  
}

public class Main {  
    public static void main(String[] args) {  
        int x = 1;  
        int y = x++;   
        // post-fix 后缀，也就是后执行；
	    // 所以x的值先拷贝给y,所以y=1;然后x自己再执行递增；
        System.out.println(x);  
        System.out.println(y);  
    }  
}
输出：2  // x =2
输出：1  // y = 1

```


复合赋值
```
int x = 1;
x += 2;      // x = x + 2
```

%   取余数
```
10/3 = 3 余 1
```

用2作为基础，取余数，只会得到0和1，可以利用这个特点运用到开发中。
```
Constants.ipFrameSize = 2

// Long类型的ip地址，除以2，并取余数，转为int类型  
int i = new Long(ip % Constants.ipFrameSize).intValue();  
// 组装redis的key, ip_lottery_day_num_i  
String key = String.format(Constants.ipLotteryDayNumPrefix+"%d", i);
```
### 关系运算

| 运算符 | 描述   |
| --- | ---- |
| ==  | 等于   |
| !=  | 不等于  |
| >   | 大于   |
| <   | 小于   |
| >=  | 大于等于 |
| <=  | 小于等于 |

### 位运算
是基于二进制的
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-07-03%20at%2022.10.50%402x.png)

一般用于计算，放在数组（HashMap)的哪个索引位置上！

| 运算符 | 描述    |                                            |
| --- | ----- | ------------------------------------------ |
| <<  | 左移    | 3 << 2<br>表示3这个数的二进制表示，左移2位<br>相当于3乘以2的2次方 |
| >>  | 右移    |                                            |
| >>> | 无符号右移 |                                            |
| &   | 与     | 6 & 3 = 2                                  |
| \|  | 或     |                                            |
| ^   | 异或    |                                            |
| ~   | 取反    |                                            |

联想到：AsyncFlow项目中，实现max_retry_interval的渐进式重试时间，就是用到了位运算
![[AsyncFlow项目#^263e0a]]

### 其他二元操作符
instanceof 也是一种二元操作符
![[Java_02_面向对象#类型检查 instanceof]]

[[#三元操作符]]

## 基本数据类型的类型转换
本质上，就是类型转换，此处主要针对的是基本数据类型的类型转换
[[Java_02_面向对象#类型转换]]

引用类型的显式转换
需要用到`if 实例 instanceof 类` 的判断进行类型检查，才能显示转换！不然会报错！
![[Java_02_面向对象#^f0bd0b]]


总结就是：
1、低精度的会隐式（自动）转为高精度的数据类型，参与计算
2、高精度不能与低精度的数据进行计算，除非将高精度的显式（手动）转为低精度的。
	这样的话，高精度的数据会损失精度（注意：不是四舍五入，直接截断）
#### 自动类型转换
隐式转换 Implicit Casting
==**byte > short > int > long > float > double**==
- 精度低的基元数据类型 自动（隐式的）转化为精度高的数据类型
- 精度低的会自动（隐式的）转化为精度高的数据类型
```
int x = 1;
float y = 2.3F;

1 + 2.3  >  1.0 + 2.3

其中的1作为低等级的int，会自动转化为高等级的float, 1.0
```

####  强制类型转换
显式转换 Explicit Casting

内置数据类型的显式转换
如果将高精度的x，加到低精度的y上去，是不行的！
需要手动将 x “显式的” 转换为int，然后再运算

```
int x = 1;
float y = 2.3F;

x + (int)y 

手动将float类型，转为int类型，结果为int类型： 1+2
```

注意：浮点型转换为整数型，会截断小数点，而非四舍五入

注意：只有兼容性的才能显式转换，将String x = "1"显式转换为int是会报错的。**==ClassCastException==**（遇到这个报错，就加 `if 实例 instanceof 类` 的判断进行类型检查，才能显示转换）

用Interger这个Java类，可以实现将String解析转化为 int
```
public class Main {  
    public static void main(String[] args) {  
        String x = "1";  
        int y = Integer.parseInt(x) + 2;  
        System.out.println(y);  
    }  
}
```


[[标准包.类#Math类]]
[[标准包.类#NumberFormat类]]

## 流程控制语句
### 比较运算符
等于  ==

Q：`==` 和Object类提供的equals()方法有什么区别？[[标准包.类#equals()]]
A ：`==`  用于比较对象的引用是否相等
equals()   用于比较对象的值是否相等！（重写后的equals）

不等于  !=

### 逻辑运算符
&&  表示 and
```
int temperature = 22;
boolean isWarm = temperature > 20 && temperture < 30;
System.out.println(isWarm);
```

||   表示OR

Q：i++  和 ++i 有什么区别？
A：i++ 是后增量，先使用，后增加
```
int i = 5;
int result = i++;

// 这里result的值是5，而i的值是6；
```

++i 是前增量，先增加，后使用；
```
int i = 5；
int result = ++i;

// 这里result的值为6，i的值也为6
```

通过字节码，可以分析出`iinc 1 by 1`的顺序是不同的！
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-04-09%20at%2023.10.34%402x.png)
### 条件判断
#### if语句
1、if后跟的判断条件，必须要用括号包起来！
2、如果条件满足，需要执行多个命令，用花括号包起来，作为一个代码块！单条命令则不需要！

#Java编程规约 关键字if与括号之间必须有一个空格，括号内的f与左括号，0与右括号不需要有空格。
```
if (flag == 0) {
	// 业务逻辑代码
} 
```

```
```
```java
if(Boolean Expression 1) { 
	//如果布尔表达式为true将执行的语句
}
else if (Boolean Expression 2) {
	// 语句
}
else {
	// 语句
}
```

```
public class Main {  
    public static void main(String[] args) {
       
        Scanner scanner = new Scanner(System.in);  
        
        int temp = scanner.nextInt();  
        
        if (temp > 30) {  
            System.out.println("It's a hot day");  
            System.out.println("Drink wather");  
        }  
        else if (temp >20 && temp <= 30)  
            System.out.println("Nice day");  
        else  
            System.out.println("Cold day");  
    }  
}
```

#### 三元操作符
条件表达式 ? 表达式1 : 表达式2

当条件表达式为真（true）时，三元操作符返回表达式1的值；当条件表达式为假（false）时，三元操作符返回表达式2的值。

如果income > 100_000成立，则返回“First”  否则返回“Economy”
	联想Python中的三元操作符 [[函数#三元操作符]]
```
int income = 120_000;  
String className = income > 100_000 ? "First" : "Economy";  
System.out.println(className);
```

联想到python中也有：[[函数#三元操作符]]

#### switch case语句
与if判断类似。
switch case 语句判断一个变量与一系列值中某个值是否相等，每个值称为一个分支。
```java
switch(expression) { 
	case value: 
		//语句 
		break; //可选 
	case value: 
		//语句 
		break; //可选 
		//你可以有任意数量的case语句 
	default: //可选 
		//语句 
}
```

```
switch (role) {  
    case "admin":  
        System.out.println("You are an admin");  
        break;  
          
    case "moderator":  
        System.out.println("You are a moderator");  
        break;  
          
    default:  
        System.out.println("You are a guest");  
}
```

### 循环
#### for循环
for循环适用于，你清楚知道自己要循环几次的情况
```
for (计数器初始化； 布尔表达式； 更新值)

例如循环5次：
for (int i = 0; i < 5; i++)
```

#### for-each循环
增强型 for 循环，也称为 foreach 循环。

你要有意识：用增强型for-each时，==可以考虑改为流式编写==；[[Java_03_高级特性#流 Stream]]
```
// List<Subject>  --->  List<SubjectEeVo>  
List<SubjectEeVo> subjectEeVoList = new ArrayList<>();  
for(Subject subject:subjectList){  
    SubjectEeVo subjectEeVo = new SubjectEeVo();  
    BeanUtils.copyProperties(subject,subjectEeVo);  
    subjectEeVoList.add(subjectEeVo);  
}  
  
// 流式写法  
List<SubjectEeVo> subjectEeVoList = subjectList.stream()
	.map( subject -> {
		SubjectEeVo subjectEeVo = new SubjectEeVo();
		BeanUtils.copyProperties(subject, subjectEeVo);
		return subjectEeVo;})
		.collect(Collector.toList());
		
```

用于迭代遍历数组或集合中的元素，每次迭代都取出一个元素赋给循环变量
```
for (变量 : 集合)
	// 将集合中每个元素取出，赋给变量，变量必须声明为合适的数据类型

String[] fruits = { "Apple", "Banana", "Orange"};  
for (String fruit : fruits)  
    System.out.println(fruit);
```

限制条件：
1、只能从头开始，不能反过来，但是如果用index就可以反过来
2、只访问到具体的值，不能获取到index
 
联想到：[[Java_03_高级特性#forEach(consumer)]]

联想到：[[标准包.类#Iterable接口]]
![[标准包.类#^5ce23f]]
#### while…break循环
**==

while循环适用于，你不知道要循环几次，但是知道满足那个条件！
```
Scanner scanner = new Scanner(System.in);  
String input = "";  
while (!input.equals("quit")) {          // 注意，字符串之间不能用 !=
    System.out.print("Input:");  
    input = scanner.next().toLowerCase();  
    System.out.println(input);  
}
```
![[#^236180]]

==**注意：while (true) 一定要搭配 break使用！==
利用while(true)…if()…break 实现自旋 ^094fff
```
while(true){  
    if(isStop){  
        System.out.println(Thread.currentThread().getName()+"\t isStop的值被修改为true，程序停止");  
        break;  
    }  
    System.out.println("---hello volatile");  
}
```

#### do…while循环
至少执行一次！很少使用，一般用while循环
```
do {
	// 至少执行一次
} while (boolean Expression);
```

### break 和 continue
break  在循环中，立刻刹车！跳出循环
continue   在循环中，回到循环开始的地方

```
Scanner scanner = new Scanner(System.in);  
String input = "";  
while (!input.equals("quit")) {  
    System.out.print("Input:");  
    input = scanner.next().toLowerCase();  
    
    if (input.equals("pass"))    // 回到循环开头
        continue;  
    if (input.equals("quit"))    // 跳出循环
        break;  
        
    System.out.println(input);  
}
```



## Clean Coding
简洁、Plat平铺的Code，更容易理解，更容易在其他地方重用，更容易维护

基于第一版贷款计算器，进行大刀阔斧的修改，使其符合Clean Code！

### 创建方法

```
public class Main {  
    public static void main(String[] args) {    
        String message = greetUse("Rocky","Shen");  
        System.out.println("Message:"+message);  
    }  

	// 这是创建的一个新方法，在main方法之外，main方法去调用它！
    public static String greetUse(String firstName, String lastName) {  
           return "Hello " + firstName + " " + lastName;  
        }  
}
```

注意：1、如果函数方法需要return，必须指定返回值的属性！不能填void了！
2、在java中，一般情况下一个函数只返回一个结果！（这点和python不同，python可以随意返回值）


### 重构代码
重构就是在不改变函数方法功能的情况下，优化代码！
把所有代码全部写到main方法中，太多了！一般一个方法控制在10-20行代码

关注两件事情：
1、代码中重复模式的，可以[[抽象]]成一个独立的方法。
2、前后高度相关代码片段，可以独立封装成一个方法。

在贷款计算器这个案例中，对应上面的重构原则：
1、在输入验证时使用的while循环其实是重复的，可以[[抽象]]成一个独立的方法，而不是重复写三边相同的逻辑！
2、将计算mortgage的核心代码封装在calculateMortgage方法中，然后在main主入口中调用核心功能函数，传递不同的参数获得不同的结果。

在IDEA中，Refactor/Extract Method，自动重构，并在main方法中自动调用。太方便了吧！


## 运行-基础Jar包
这里指的是部署命令行驱动的脚本，Java Web Application需要更复杂的步骤。

Jar包（java archive的缩写），可以分发给任何安装了JRE的电脑使用

Step 1：File / Project Structures / Artifacts
Step 2: Artifacts中选择Jar，然后勾选Class Main文件，Apply
Step 3：Build / Build Artifacts（会根据前一步定义的进行构建）
Step 4：生成的Jar包会放在output/artifacts/HelloWorld_jar/HelloWorld.jar
Step 5：右键这个Jar包，Open In Terminal，执行命令：`java -jar HelloWorld.jar` 就可以基于命令行进行交互使用这个程序了！

[[SpringBoot#SpringBoot项目部署]]

![[SpringBoot#^3015e1]]
