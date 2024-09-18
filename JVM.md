Java虚拟机，内存管理。
是Java语言的内功。Java API的开发，就好比是武功一招一式！

## JVM
它负责在实际计算机硬件层面执行Java字节码，主要提供一个独立的、与操作系统无关的执行环境，使得Java程序可以在不同平台上运行！“一次编写，到处运行"

| JVM                | XX                 |                             |     |
| ------------------ | ------------------ | --------------------------- | --- |
| ==**HotSpot VM**== | Sun公司发布的JVM        | 合二为一，Oracle默认使用<br>使用JIT编译器 |     |
| ==**JRockit**==    | BEA公司              | 合二为一                        |     |
| J9                 | IBM公司              |                             |     |
| TabobaoJVM         | 基于OpenJDK，淘宝自研的JVM | 阿里巴巴是Java重度用户               |     |
| Graal VM           |                    | 未来主流                        |     |
| Dalvik VM          | Google             | Andriod平台                   |     |

JVM其实已经不和Java深度耦合了，只要其他编程语言也符合JVM提供的规范，提供符合规范的字节码文件，都能执行（例如：Kotlin、Groovy、Scala、Jython、JRuby、JavaScript、Clojure等
每一个Java新版本的发布，除了发布Java语言的特性，还会发布虚拟机的规范。
[Java Language and Virtual Machine Specifiactions](https://docs.oracle.com/javase/specs/index.html)

Java既有编译器（JIT），也有解释器。两条腿走路！半编译，半解释型语言！
先执行javac，进行compile编译
再进行java，运行字节码文件

![[Java_01_基础#^a2fdb3]]
### 前端编译器  ->  javac
将 `.java` 文件转化为 `.class`
前端编译器：javac（全量编译，IDEA用这个）、ECJ（Eclipse，增量编译）

javac主要将Java源代码转换为字节码，生成的.class字节码交给JVM类加载器。
javac的编译，不在JVM中发生，而在Java中发生！

### 解释器
解释器：将字节码逐行翻译成机器码（解释器，慢）

### 后端编译器  ->  JIT
JIT编译器，将字节码转化为机器码（提升解释速度，热点）

注意：Java是编译型语言，不是指javac的过程，而是编译完成生成class字节码之后，利用JIT编译器预热！（后端编译器）

JIT编译器的预热机制，使得Java的运行速度逼近于C语言！牛逼！

![[Java_01_基础#^059118]]

简图
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/%E5%9B%BE%E5%83%8F2024-4-9%2022.28.JPG)

## 参数速查表
### 指令

| 指令名                 | 说明                   | ==命令行==                                        |
| ------------------- | -------------------- | ---------------------------------------------- |
| jps -l              | 查看正在运行的java程序的进程id   | 联想到：[[linux#ps -ef grep docker]]               |
| jinfo -flag 配置项 pid | 查询输入的参数是否生效          | jinfo -flag PringGCDetails 56208<br>输出： + 表示生效 |
|                     | MetaSpaceSize        | 查看元空间大小，21m左右                                  |
|                     | MaxTenuringThreshold | 新生代存活几次，进入老年代                                  |
| jinfo -flags pid    | 把当前进程所有配置信息都显示       |                                                |
|                     |                      |                                                |



### -XX 布尔类型

| -XX:                       | 备注                                                                                                                |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| +PringGCDetails            | 开启功能：打印GC过程的详细信息                                                                                                  |
| -XX:+PrintHeapAtGC         | 在垃圾回收之前和之后堆的占比情况                                                                                                  |
| -Xloggc:log/gc-oomHeap.log | 生成日志文件到指定目录                                                                                                       |
| +PrintFlagsInitial         | java -XX:+PrintFlagsInitial<br>不用运行任何java进程<br><br># 打印偏向锁相关的参数<br>java -XX:+PrintFlagInitial \| grep BiasedLock* |
| +PrintFlagsFinal           | java -XX:+PrintFlagsFinal -version<br>`:=`表示被修改过的行                                                                |
| +PrintCommandLineFlags     | 运行java程序时，<br>将简单版的配置信息打印到控制台<br>简单信息：初始堆内存、最大堆内存、默认用哪种GC垃圾回收器                                                    |
| +UseSerialGC               | 使用串行GC                                                                                                            |
| +ParallelGC                | 使用并行GC<br>java8 默认                                                                                                |
| +UseConcMarkSweepGC        | 使用CMS<br>老年代                                                                                                      |
| +UseParNewGC               | 使用ParNew<br>新生代，配了CMS，自动带出来                                                                                       |
| +UseG1GC                   | 使用G1                                                                                                              |
### -XX 键值对

| -XX:                    | 说明                        | 默认值                               |
| ----------------------- | ------------------------- | --------------------------------- |
| MetaspaceSize=128m      | 设置元空间大小                   |                                   |
| MaxTenuringThreshold=15 | 设置对象在新生代存活多少次，可以进入老年代     |                                   |
| InitialHeapSize         | 等价于-Xms2048m              | 设置堆空间初始值<br>默认物理内存1/16            |
| MaxHeapSize             | 等价于-Xmx2048m              | 设置最大堆空间<br>默认物理内存1/4，一般开发2G够用<br> |
| ThreadStackSize         | 等价于-Xss512k<br>设置虚拟机栈的初始值 | stack start<br>虚拟机栈空间大小，默认1m      |
| SurvivoRatio            | 设置新生代与老年代的占比              |                                   |
| -Xmn                    | 设置年轻代大小，一般不调              |                                   |
| NewRatio                |                           |                                   |

## 字节码
字节码文件是跨平台的！

Q：为什么要懂字节码指令？ #Java八股/JVM 
A：分析代码运行的细节（i++，++i）

### 常见字节码指令
一个字节 = 256个字节码指令。（汇编领域了
操作码（opcode)：代表某种特定操作含义；
操作数(oprand)：操作所需要的参数。

一个虚拟机栈，调用一个方法，就是一个栈帧，一个栈帧包含：操作数栈+局部变量表。
查看：OneDrive\文档\02-学习\Learn_Java_Mosh\JVMDemo\尚硅谷_宋红康_JVM指令手册.md

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B44%E6%9C%8810%E6%97%A5%2009_48_12.PNG)
#### 常量入栈
|        |                                                             |
| ------ | ----------------------------------------------------------- |
| iconst | 小数据（-1～5），一般从int的常量池中取<br>将常量加载到操作数栈上<br>iconst_5，只能推“5”到栈顶 |
| bipush | 大数据（-128～127），将单字节的整数常量值推入操作数栈的栈顶（栈底？<br>（-128～127）         |
| sipush | 超大数据（-32768～32767），采用sipush指令                               |
| ldc    | 将字符串加载到操作数栈上                                                |
|        |                                                             |

#### 出栈装入局部变量表
|        |                               |
| ------ | ----------------------------- |
| istore | 从操作数栈中取出，放入局部变量表中<br>istore_1 |
| astore | 引用类型，从操作数栈中取出，放入局部变量表中        |

#### 局部变量表取出，入栈
|       |                                                                     |
| ----- | ------------------------------------------------------------------- |
| iload | 将局部变量表中的==数==取出来，放到操作数栈上<br>iload_1                                 |
| aload | 将局部变量表中的==引用类型==取出来，放到“操作数栈”上；<br>（0位默认是this)，操作数合并进操作码，简化为：alaod_0 |

#### 调用方法
|                 |                                                                         |
| --------------- | ----------------------------------------------------------------------- |
| invokevirtual   | 【最常用】调用对象的实例方法，<br>可以被重写的方法，都是virtual;<br>支持多态<br>Father s = new Son(); |
| invokeinterface | 调用接口中的方法                                                                |
| invokspecial    | 【常用】调用构造方法 [[Java_02_面向对象#构造方法 Constructor]]                            |
| invokestatic    | 调用静态方法，静态方法不能被重写，区别于invokevirtual                                       |
| invokedynamic   | [[Java_03_高级特性#方法引用]]                                                   |
|                 |                                                                         |

#### 其他

|               |                          |
| ------------- | ------------------------ |
| iinc 1 by 1   | 在局部变量表中操作，索引1位置的数字，自增长1； |
| getfield      | 从对象中获取字段的值，并将值推送到操作数栈上   |
| putfield      | 将栈顶的值存入对象的字段             |
| new           | 创建一个对象，并将引用值对送到操作数栈上     |
| return        | 返回值                      |



### Class文件
二进制文件，转成16进制，进一步翻译，就是Jclasslib插件提供的信息（字节码指令信息）
jclasslib插件的使用：view-show bytecode with jclasslib，就能在左侧边栏显示字节码指令！

```
├── 一般信息
├── 常量池
├── 接口
├── 字段
├── 方法
	├── 【0】第一个方法
		├── code    # 这里就是该方法的字节码指令
	├── 【1】第二个方法
├── 属性
```

IDEA打开的class文件，显示的Java代码，其实是反编译的结果。本质是二进制文件，不信你拷贝出去点开看，就是0101001！

01-魔数：前四位，Java文件对应： ca fe ba be
02-副版本+主版本： 四位，对应Java版本（52 = jdk 1.8）

#### 03-静态常量池 / class常量池
是字节码文件中最多的内容，后面的方法、属性都是指向静态常量池中的引用！

常量池相当于是一个资源库，类、方法、属性都从这里去找！

最重要！~~0表示无，1-21，把代码中的属性、方法等信息都存进去！ ~~

是指在字节码文件中的常量池！

当类加载时，class中的静态常量池被加载到运行时内存中！

包含：字面量 和 符号引用

##### 字面量
字面量 - 数值型：你看到是什么，存在常量池的就是什么；
final修饰的常量，也称为字面量
![CleanShot 2024-06-10 at 22.38.55@2x.png](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-06-10%20at%2022.38.55%402x.png)

字面量 - 字符型，存了另一个地址#26，这个地址实际存字符串
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-06-10%20at%2022.35.34%402x.png)

##### 符号引用

- 符号引用：以一组符号来描述所引用的目标，符号可以是任何形式的字面量，符号引用于虚拟机实现的内存布局无关。
- 直接引用：直接指向目标的指针，直接于虚拟机内存布局相关。有了直接引用，那说明引用的目标已经存在于内存中了。


符号引用：如下图，存的是Object类信息
![CleanShot 2024-06-10 at 22.40.26@2x.png](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-06-10%20at%2022.40.26%402x.png)



04-访问标识：对应权限，public、private
05-类索引、父类索引、接口索引
06-字段表集合：Filed，计数+刻画信息
07-方法表集合：方法1、2、3
08-属性表集合：这里的属性，和Filed不是一个概念，是Class文件的属性！不要混淆！ ^218709

详图
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/%E5%9B%BE%E5%83%8F2024-4-9%2022.29.JPG)

## 类的加载
ClassLoader，负责将Java类的字节码文件加载到内存中，并转换成一个Class对象
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-04-10%20at%2014.07.47.png)

当前类.getClassLoader()方法，可以获取到是谁加载了当前类！

### 类的加载过程
#### 装载  Loading
过程一：Loading 装载阶段
把字节码文件，加载到内存中，成为类模板（方法区=元空间）

#### 连接  Linking
过程二：Linking 链接阶段
##### 验证  verify
1、Verify：==验证==：看一下class文件是不是合法的class文件，格式、语义等ca fe ba be。和装载过程同时进行，如果验证失败，不会进来！

##### 准备  prepare
2、Prepare：==准备==：类的静态变量分配内存，并初始化为默认值。注意:final常量在本阶段被赋值！
![[Java_02_面向对象#^c02f1c]]

##### 解析  resolve
3、Resolve：==解析==：
	1、将字节码内“符号引用（#14）”转变为“直接引用”（内存地址），在初始化完成之后，才执行的！
	2、类的静态代码块会执行。

#### 初始化  Initialization
过程三：Initialization 初始化阶段
将静态变量的值从默认值修改为正确的值。
![[Java_02_面向对象#^1cda5e]]
[[Java_02_面向对象#实例化与初始化的区别]]

开始执行Java代码了
执行
##### clinit()
类初始化方法，在类加载时用来完成类的初始化工作。
1、Java 虚拟机在类加载阶段自动调用的执行方法，用于对类进行静态初始化工作。
2、clinit 方法是由编译器自动生成的，用来初始化类变量和执行类的静态代码块。
3、clinit 方法在类加载阶段执行，且只执行一次。如果一个类没有静态变量和静态代码块，编译器不会生成 clinit 方法。
4、JVM 提供的特殊方法，不需要显示定义，由编译器根据类的静态成员变量和静态代码块自动生成。

clinit 方法是类级的初始化方法，用于完成类的静态初始化工作。

#### 使用

#### 卸载

总结：
显式赋值，在链接阶段的Prepare环节
不是显式赋值（涉及方法调用），在初始化阶段完成（因为需要调用其他类的方法）

子类加载前，必须先加载父类。
口诀：由父及子，静态先行。

下面是生命周期，使用阶段
过程四：Using 使用

过程五：unloading 卸载

##### init()
实例初始化方法，对应类的构造器，进行构造一个类的实例的方法
对象级的初始化方法，用于完成实例的初始化工作
### 类的加载时机
就是从  方法区 ->  堆空间
主动使用的情况
1、当创建一个类的实例，new关键字、反射等
2、调用类的静态方法
3、调用类的静态字段（static field)
4、使用反射类的方法forName时
5、初始化子类时，发现父类还没被初始化时也会初始化
6、接口中如果定义了default方法，那么接口实现类初始化前，接口也会default方法初始化
7、JVM启动时，用户必须指定一个主类，main()方法，会初始化这个主类

Q：有哪些方法可以触发类的加载？ #Java八股/JVM 
A：new关键字、调用静态方法、调用静态属性、接口default默认实现、反射、子类实例化触发父类、程序入口main方法

被动使用的情况：
不会引起类的初始化。也就是说，并不是代码中出现的类，就一定会被加载或初始化，不符合主动使用的条件，类就不会初始化
1、通过子类引用父类的静态变量时，不会导致子类初始化。
2、通过数组定义类引用，不会触发类的初始化（只是开辟了空间，用null占用）
3、引用常量不会触发此类或接口的初始化（常量final在链接阶段被显示赋值）

### 类的加载器
是指从硬盘中将class文件，装载到方法区时用到的：ClassLoad

重要机制：**同一份Class文件，被不同==类加载器==加载，最后生成的java.lang.Class对象是不同的！**
这个机制可以实现类的隔离和不同版本的并存。 ^60cca3

子类加载器可以访问父类加载器的类型，但是反过来不行！
父类加载器加载过的类，子类加载器就不会重复加载了（子类加载器可以访问父类，子类是知道的）

分为：引导类加载器（Bootstrap ClassLoader）和自定义类加载器（User-Define ClassLoader)

```
ClassLoader.getClassLoader;
```

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-04-10%20at%2016.25.37.png)

#### 启动类(引导类)加载器
Bootstrap ClassLoader
由==C语言编写==
或叫做：启动类加载器

主要负责：加载Java核心库（`JAVA_HOME/jre/lib/rt.jar`）（java.lang.Class、java.lang.Object、java.lang.String、==java.lang.ClassLoader==等等）
```
/Library/Java/JavaVirtualMachines/jdk-1.8.jdk/Contents/Home/jre/lib
```

处于安全考虑，Bootstrap启动类加载起只加载包名为：java、javax、sun开头的类

#### 拓展类加载器
Extension Class Loader
由java.lang.ExtClassLoader提供

2、继承自 extends ClassLoader类
3、父类加载器为：启动类加载起
4、主要负责：java拓展文件夹里的类库中的类！例如：XML、加密、压缩等功能！
```
/Library/Java/Extensions
```

#### 系统类加载器
AppClassLoader
由java.lang.AppClassLoader提供

2、继承自 extends ClassLoader
3、父类加载器为拓展类加载器

4、负责加载环境变量classpath 或 系统属性 java.class.path指定路径下的类库
5、默认就是类加载器！
6、是用户自定义加载器的父类加载器

主要负责：我们写的代码中的各种类！

##### 自定义类加载器
不用系统类加载器，也可以自己手写自定义类加载器！
tomcat和spring都有自己的自定义类加载器。

不重写loadClass()方法，因为这是遵循双亲委派机制的源代码，里面有preDefine方法对核心API加载过程的保护！
需要重写findClass()

为什么要自定义类加载器？
1、隔离加载类；(一份字节码文件，使用同一个加载器只能加载一次，如果两个加载器，就能对同一份字节码文件，加载两次，实现隔离！)
2、修改类加载的方式；
3、拓展加载源；例如从数据库、网络上加载；
4、防止源码泄漏；对生成的字节码文件进行加密，自定义类加载器加载字节码文件时解密

###### 常用方法
- getClassLoader()     获取当前类的类加载器。
- getClassLoader().getParent()           获取当前类的类加载器的父加载器
- 
###### 步骤
0、继承ClassLoader类，重写findClass()方法
1、转换为以文件路径表示的文件
2、获取指定路径的class文件对应二进制数据
3、自定义classLoader，内部调用defineClass()（之前会调用preDefineClass()接口，保护机制）

defineClass(name, data, offset, length)
	参数1：要定义的类的全限定名，例如”com.atguigu.java"
	参数2：表示类的字节码数据
	参数3：字节数组开始的偏移量，即：从第offset个字节开始
	参数4：读取字节的字节数，也就是从偏移量开始，读取长度为length的数据

`java.lang.NoClassDefFoundError`
注意：修改了代码之后，要重新compile编译之后，产生的class文件覆盖原来的才行。（class内包含的字节有变化了）

NoClassDefFoundError       类加载器运行时，找不到类
ClassNotFoundException    编译时，找不到类

在命令行中启动Java程序时，添加-verbose:class，可以输出类加载过程，便于优化
```
java -jar -verbose:class
```

### 相关机制
#### 双亲委派机制
启动类（引导类）加载器 -> 拓展类加载器 -> 系统类加载器 -> 自定义类加载器
父类加载失败由子类加载器处理

![[#^60cca3]]

Q：为什么双亲委派机制，好处是什么？ #Java八股/JVM 
A：使用双亲委派机制，可以保证，每一个类只会有一个类加载器。例如Java最基础的Object类，它存放在rt.jar包中，这是[[#引导类加载器 Bootstrap ClassLoader]]职责范围，只有当向上委派到Bootstrap时才会被加载。但是如果没有双亲委派机制，可以任由自定义加载器进行加载的话，这些Java核心类的API就会被随意篡改。

弊端：
顶层ClassLoader无法访问底层ClassLoader所加载的类。举个例子：
![[Spring FrameWork#^6a3f1c]]
JDBC的类，由引导类加载器加载的，各数据库厂商的实现类，是由系统类加载器加载的。上层想要调用下层就不行了！！
##### ClassLoader源码
```
if(c == null){
	long t0 = System.nanoTime();
	try{
		if (parent != null){
			c = parent.loadClass(name,false);  //往上委派
		}else{
			c = findBootstrapClassOrNull(name);
		}
	}
}
	if (c == null){
	//如果parent都不能load，那就自己loading
	}
```
上述代码框架，就是实现双亲委派机制！

##### 破坏双亲委派机制
1、利用线程上下文加载器（由系统类加载器加载的），帮助我们搭建一个桥梁，去辅助调用
```
Thread.currentThread().getContextClassLoader();  
```

2、热部署的实现，就让更新的class文件走自定义的类加载器，不走双亲委派机制漫长！

3、Tomcat8 较 Tomcat6 最大的区别是什么？
A：可以通过`<Loader delegate="true" />` 表示是否遵循双亲委派机制。
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B44%E6%9C%8812%E6%97%A5%2023_01_34.PNG)

哪些类型对应有Class对象？
- 外部类、成员（成员内部类、静态内部类）、局部内部类、匿名内部类[[Java_02_面向对象#类 Class]]
- 接口  [[Java_02_面向对象#接口 - interface 关键字]]
- 数组  
- 枚举  [[Java_02_面向对象#enum 枚举类]]
- 注解  [[SpringCloud#注解汇总]]
- 基元数据类型  [[Java_01_基础#基元类型]]
- void

### (方法区)元数据 和(堆)Class对象的理解
方法区中的类的元数据是JVM的规范，不便于直接访问，而堆中的Class对象是提供给程序员使用的一种API层面的机制，可以利用Class对象实现反射！

（堆）Class对象就是对（方法区）元数据的封装。

堆中一个个实际使用的实例对象，都是基于该类的Class对象创建出来的。联想到Spring框架中Bean都是基于BeanDefinition创建出来的，是一个思路！

**虚拟机栈 <--引用--> 堆的实例对象  <--类型指针--> (堆)Class对象  <-引用-> (元空间)元数据** ^147e0c


![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B47%E6%9C%8826%E6%97%A5%2014_47_33.jpg)

![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202407261453549.JPG)

![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202407261502872.JPG)

## 运行时内存
Runtime Data Area，包括方法区、堆、栈、程序计数器、本地方法栈。
用于存储类的结构信息、对象实例、方法调用状态、线程的执行状态；

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/runtime_area.jpg)

灰色部分是线程独享的；线程隔离区：每个线程生成一套新的
红色部分是多线程共享的；线程共享区：所有线程都共享信息，发生GC的地方！

程序计数器：就是CPU在各种线程之间跳转时，记录跳出时在哪个地方，一会回来时接续！

### 虚拟机栈
包含：操作数栈 + 局部变量表
设置栈的大小为1M，堆比较大，几百M！栈的速度非常快，都是引用类型的
栈是每个线程独立的，堆是同一个进程的所有线程共享的（一个进程通常包含3-5000个线程）

过小的栈可能导致栈溢出；过大的栈可能导致线程数目上的限制。

栈中不存在GC（垃圾回收）。
栈中存在OOM（内存溢出 OutofMemoryError）；
栈也容易报出StackOverFlow（递归导致，或者局部变量表很大）

#### 栈参数
-Xss size
Xss = Max Stack Size

设置（**一个线程里的**）栈大小的命令： `-Xss size`   一般就是默认的：-Xss 1024k
不要设置栈过大，否则导致系统用于创建线程的数量减少。
一般一个进程中通常有3000-5000个线程。
IDEA中，每个允许的SpringBoot Application可以设置Enviroment/VM Options : `-Xmx100m` 指定这个微服务实例的JVM最大内存为100m； ^ee16cb

jdk5.0 之前，默认栈大小：256k
jdk5.0之后，默认栈大小：1024k
```
-XX:ThreadStackSize=0    输出0，表示是默认选项，也即1024k
```

栈里存了栈帧（一个方法对应一个栈，StackTrace），出栈的方式：正常返回、抛异常返回
#### 方法调用压栈

#### 局部变量表
数据结构：Table -> 数组
存放方法中的局部变量

普通方法如果有两个变量，默认就是3个局部变量表，包含this


#### 操作数栈
用来保存方法执行过程的中间结果！
数据结构：栈
long类型的数据，在操作数栈中占位2个！

栈顶缓存技术：将操作数栈中常用的数据，缓存到CPU中，提升效率

#### 帧数据区
动态链接：当方法中需要引用另外类的方法时，在动态链接中记录关联关系。
方法返回地址：Return Address（ireturn -> int、lreturn -> long、dreturn、areturn 引用类型
附加信息：

Q：栈溢出的情况？ #Java八股/JVM 
A：StackOverflowError，无限制条件的递归，当把栈塞满时就会栈溢出，抛出错误；
调整栈的大小也不能保证不溢出，只会延缓溢出的时间（无限递归的话，迟早会满）

Q：栈内存越大越好吗？ #Java八股/JVM 
A：由于-Xxs是单个线程的内存大小分配，单个分配大了，会导致线程数减少。

Q：虚拟机栈存在垃圾回收GC吗？ #Java八股/JVM 
A：不存在GC

| XX         | 垃圾回收器 | 是否会溢出 |
| ---------- | ----- | ----- |
| 程序计数器      | NA    | NA    |
| 方法栈（及操作数栈） | NA    | 会     |
| 方法区、堆      | 会     | 会     |
Q：方法中定义的局部变量，是否线程安全？ #Java八股/JVM 
A：方法存放在一个栈帧中，方法栈是每个线程独一份的，不存在线程共享问题。但是，局部变量表中存放的变量有可能是引用类型，是指向堆空间，所以就可能线程不安全了。
关联：变量逃逸问题

本地方法接口、本地方法栈：调用C语言的第三方库。（目前Java第三方库很丰富了，这里就不常见了）。调用了C的库，就出JVM范畴了！

### 堆
栈管运行，堆管存储
一个进程只存在一个堆空间，被进程中所有线程所共享。

==**堆，是分配对象的唯一选择！**==

堆中存在GC，栈中不存在GC（栈中频繁发生进出栈，不需要GC），堆也不是栈中局部变量引用一断开，就马上GC，按算法触发的！
#### 堆参数
堆空间最大值 =  物理内存的1/4
堆空间初始化值 = 物理内存的1/64
```
Runtime.getRuntime().totalMemory();    //获取JVM的初始化堆内存总量

Runtime.getRuntime().maxMemory();      //获取JVM试图使用的最大堆内存
```

| XX                          | XX                                                                      |
| --------------------------- | ----------------------------------------------------------------------- |
| -Xms200M                    | 设置堆空间的初始化     -XX:InitialHeapSize=200M<br>一般开发过程中，堆空间设置为**==2G==**就够用了。 |
| -Xmx200M                    | 设置堆空间最大值         -XX:MaxHeapSize=200M                                   |
| -XX:NewRatio=2              | 默认是2，新生代占1，老年代占2；==一般不改！==<br>例如：改为4，表示新生代占1 ，老年代占4                     |
| -XX:SurvivorRatio=8         | 默认是8，Eden占8，SurvivorFrom占1，SurvivorTo占1；<br>其实是动态的，需显式声明                |
| -XX:MaxTenuringThreshold=15 | 默认15（4个字节1111就是15）<br>新生代年龄递增到多少进入老年代                                   |
| -XX:PrintGCDetail           | 打印详细的垃圾回收日志信息                                                           |
| -XX:HandlePromotionFailure  |                                                                         |
| -XX:+PrintFlagsFinal        |                                                                         |
| -XX:TLABWasteTargetPercent  | TLAB，是指Eden区划分不同部分，分给不同线程造对象；<br>设置TLAB占用Eden区的百分比。                     |

-XX 表示“非标准参数”，实验性功能，设置虚拟机的选项

OutofMemoryError:Heap Space

做个实验：
new一个50m的字节数组，而我们手动修改堆空间初始和最大值是10m，一创建这个对象，就会触发OOM：Heap Space
```
public class HelloGC {  
    public static void main(String[] args) {  
        System.out.println("*********hello GC");  
        Byte[] byteArray = new Byte[50 * 1024 *1024];    
        //构建一个50m的对象，堆空间设置的是10m,触发OOM:heap space  
    }  
}

>> Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
```

Q：初始化堆大小和最大堆大小设置成一样的，有什么好处？ #Java八股/JVM 
A：初始大小  --动态调整--> 最大值。 避免变大缩小时浪费性能，一开始就画死了。
（预测，新版中会删减这块）

#### 年轻代 -> 老年代
康师傅这段讲的不清楚，没有后端训练营讲的清楚：~~当创建对象时，发现Eden区满了，于是触发Minor GC，将存活的对象放到Survivor区，并记录年龄+1。同时触发Survivor区的GC，把幸存的对象放到SurvivorFrom区，SurvivorTo区就空了（谁空谁是TO），下一次再从Eden区加入的时候，就加到From区。当年龄到15时，发配到老年区。From和To倒腾一把，就是为了让他们内存空间连续，避免内存碎片！~~

HotSpot虚拟机中将新生代划分为一块较大的Eden区和两块较小的Survivor区，每次分配内存只使用Eden和其中一块Survivor，发生GC时，Eden和Survivor中存活的对象一次性复制到另一块Survivor上（年龄要+1）。然后直接清理掉Eden和已用过的那块Survivor。默认比例为8:1:1。因此，每次新生代可用空间为90%，剩余10%为冗余。

| XX  | XX       |
| --- | -------- |
| 新生代 | Minor GC |
| 老年代 | Major GC |

新生代和老年代在堆空间结构中的占比；
老年代OldRatio：存活比较久的对象，GC低频执行。
新生代NewRatio：新生的对象。Eden区满时，GC会频繁执行。

新生代占1，老年代占2；
Eden区：SurvivorFrom：SurvivorTo  = 8:1:1
主要利用垃圾回收算法中的复制算法。

Q：为什么要把Java堆空间进行分代？ #Java八股/JVM 
A：优化GC性能，把朝生夕死的对象集中在新生代中，过滤出那些生命周期长的放到老年代中。

### 方法区 / 永久代
方法区物理上可能在堆上，但是理论上，方法区不是堆的一部分，它也叫：Non-Heap。
基本上不GC，因为是类模版放在这里，很难被回收  ->  联想到：[[#类的加载]]

注意：方法区是Java虚拟机规范中的概念！
	永久代（JDK7及以前）和元空间（JDK8及以后）是方法区规范的具体实现（接口与实现类）

方法区存放：==元数据==（类型信息）、Field信息、方法信息、non-final信息、运行时常量池
	jdk1.7以后，静态变量都放在堆空间里。

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B44%E6%9C%8817%E6%97%A5%2022_36_33.PNG)


Q：HotSpot中方法区的演进？
A：jdk7之前，叫永久代（JVM内存）；jdk8开始，叫元空间（本地内存）。


Q：为什么java8使用元空间取代了永久代？ #Java八股/JVM 
A：[JEP 122: Remove the Permanent Generation](https://openjdk.org/jeps/122)
永久代使用的是JVM内存，元空间使用本地内存。元空间存放的是类模版，很难被GC掉，如果永久代放在JVM上更大概率发生OOM，但是放本地内存不太容易出现OOM。

#### 方法区参数

| XX                      | XX                        |
| ----------------------- | ------------------------- |
| ~~-XX:MaxPermSize=64m~~ | ~~设置初始永久代的大小，已在JDK8之后废弃~~ |
| -XX:MetaspaceSize=64m   | 设置元空间初始大小                 |
| -XX:MaxMetaspaceSize=?? | 设置元空间最大大小，默认无限制，也即物理内存    |

什么情况下会报OutofMemoryError:MetaSpace？ #Java八股/JVM 
大量new太多的对象，导致元空间里对象将内存占满就会报这个错！

#### 本地方法栈
Native方法（本地方法）是指使用其他编程语言（如C、C++等）编写的方法，它们不是由Java虚拟机直接执行，而是通过调用本地系统的原生方法库来执行。 ^9d4248

使用native关键字进行声明

使用本地方法的步骤如下：
1. 在Java类中声明一个本地方法，使用native关键字，但不提供方法实现。
2. 在Java类外部的本地方法库中实现该本地方法，通常使用C或C++编写。
3. 将本地方法库编译为与目标平台兼容的动态链接库，如使用JNI（Java Native Interface）技术。
4. 在Java程序中，通过加载本地方法库并调用本地方法来使用本地功能。
例如线程中的start()方法，就是native方法

#### 运行时常量池
就是将[[#03-静态常量池 / class常量池]]加载到虚拟机内存之后，就叫：运行时常量池。
##### 字符串常量池 StringTable --> 堆空间
JDK 8开始，从元空间挪到了堆空间中（便于GC）

以下几种情况，会将字符串加入字符串常量池：
1、使用字面量的字符串，会放入字符串常量池
```
String s1 = "helloworld";
```

2、通过 .intern()方法，就会写入字符串常量池

注意：new String()构造出来的字符串，不会存放在常量池中，只会存放在堆空间中，作为对象被对待！

##### 其他(Integer)数据类型常量池
除了字符串常量池，其他几种基本数据的包装类型对应常量池，（除了浮点数没有）

注意：int这种基本数据类型没有常量池的概念，它们是直接存储的值；都是针对包装类型的常量池，例如Integer类型的常量池！

- Integer类型常量池：主要存放-128～127范围内的int类型整数常量值。如果int的值超过了范围，只会去堆空间new，不会写入int类型常量池。所以
- Short类型常量池
- 等等

| XX           | XX        |
| ------------ | --------- |
| Integer类型常量池 | -128～127  |
| String类型常量池  | ASCII码的字符 |
| 3            | cc        |

## 对象内存布局

### 对象的实例化
Q：有多少种方式创建对象？
A：
直接new；反序列化（JSON - > pojo）；clone()一个对象；Class的newinstance()；静态方法Builder和Factory

从字节码操作角度看待对象创建过程：
new + dump，是复制品，用于init初始化方法赋值，过度使用

从执行步骤看待对象创建：
1、判断对象对应的类模版，是否加载了。
2、为对象分配内存空间。（int类型，4个字节；Double类型，8个字节）
	指针碰撞：基于已使用的内存地址，往右边移动一部分。确保内存没有碎片  联想到 [[数据结构-链表#单链表]]
	空闲列表：维护没有存放的内存地址记录。  [[Java_01_基础#数组]]
3、处理并发安全问题（多个线程共用堆）
	1、加锁
	2、在新生代的Eden区，分离TLAB
4、初始化分配默认值。
5、设置对象头信息
6、执行init方法进行初始化。  ->  [[Java_02_面向对象#构造器 Constructor]]

### 强、软、弱、虚引用

| XX                               | XX                                                                                                                                    |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| 强引用<br>`Person p = new Person()` | OOM都不回收，死都不收。默认，95%以上                                                                                                                 |
| 软引用<br>`SoftReference<T>`        | 地主家有余粮，就不和你计较了。<br>内存不足时，才会回收<br>一般用在内存敏感的程序，例如高速缓存中                                                                                  |
| 弱引用<br>`WeakReference<T>`        | 只要GC了，就会回收<br>（不管内存够不够）                                                                                                               |
| 虚引用<br>`PhantomReference<T>`     | 不单独使用，搭配ReferenceQueue<br>任何时候都会GC回收，一GC就触发虚引用，加入引用队列<br>[[标准包.类#finalize( )]]底层实现<br>联想到：[[Spring FrameWork#AOP 面向切面编程]]中的后置通知@After |

`XxxReference.get()`  获得这个对象

- 强引用：不回收，程序90%的场景是强引用

- 软引用：内存不足时回收，地主家有余粮，就不跟你计较。
```
User u1 = new User(1,"tom");
SoftReference<User> userSoftRef = new SoftReference<User>(u1);
u1 = null;    // 取消强引用

userSoftRef.get()  //就能获取到u1对象
```

造一个大对象，模拟内存不够的情况
```
JVM设置 -Xms10m -Xmx10m

byte[] bytes = new byte[20 * 1024 * 1024];  // 20MB的对象，放到堆空间
```

**软引用-适用场景**：
- 实现内存敏感的缓存数据（mybatis缓存也用到了）
- 如果图片不加载到内存中，用的时候去读，影响效率；如果强引用，加载到内存中，又太占用内存空间。此时图片对象，利用软、弱引用加载到内存中。内存足够时，我不动你，内存不够了，我就把图片大对象给GC掉。完美方案！

- 弱引用：发现即回收 ^92b50e
```
User u1 = new User(1,"tom");
WeakReference<User> userWeakRef = new WeakReference<User>(u1);
u1 = null;    // 取消强引用

userWeakRef.get()  //就能获取到u1对象
```

**适用场景**：保存那些可有可无的缓存数据；实现高速缓存！
WeakHashMap

- 虚引用：对象回收跟踪
唯一目的：跟踪垃圾回收过程。
适用场景：释放资源。（finalize也可以）

### 对象内存布局(对象头）
#### 对象头
64位系统中：class Pointer占8个字节；Mark word占8个字节；总共16个字节
##### 类型指针 Class Pointer
- 类型指针：指向方法区中==元数据==（类型信息）。

```
Customer c1 = new Customer;
   ^
   |
这个就是类型指针！Class对象，类模版

```

##### 对象标记 Mark Word
对象标记 ^6d2e5f
8个字节 = 64bit。
![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202408011435664.jpg)

- 1、对象在堆中的地址哈希值  [[标准包.类#hashCode()]] ^e112fb
```
Object o = new Object();

o.hashCode();

>> 87669269
```

- 2、GC分代年龄；

| -XX:MaxTenuringThreshold=15 | 默认15（4个字节1111就是15）<br>新生代年龄递增到多少进入老年代                                   |
| --------------------------- | ----------------------------------------------------------------------- |
- 3、锁状态标志； ^7e3322

| 存储内容                | 标识位 | 状态        |
| ------------------- | --- | --------- |
| 对象哈希码，分代年龄          | 01  | 未锁定       |
| 指向锁记录的指针            | 00  | 轻量级锁定     |
| 指向重量级锁的指针           | 10  | 膨胀（重量级锁定） |
| 空，不需要记录信息           | 11  | GC标记      |
| 偏向线程ID，偏向时间戳，对象分代年龄 | 01  | 可偏向       |

- （数组特有）长度；如果是数组的话，记录长度
#### 实例数据
当前类的实例变量；包括父类的实例变量（继承）
先放父类，再放自己类的
相同宽度的先放；
#### 对齐填充
补齐空间。保证8字节的倍数！

#### JOL
Java Object Layout
打印对象内存布局信息
```
public static void main(String[] args) {  
    Object o = new Object();  
    System.out.println(ClassLayout.parseInstance(o).toPrintable());  
}
```

![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202408011609674.png)
### 对象的访问定位
~~句柄访问：堆空间中维护一个句柄池（Java没有使用这个方式~~
指针访问：常见访问方式。

## 执行引擎
Execution Engine，负责执行字节码指令，包含解释器和即使编译器。解释器逐条解释执行字节码，而即时编译器会将热点代码编译成本地机器码，已提高执行效率

类加载 和 运行时内存  主要是把外部磁盘上的字节码文件载入到内存中，执行引擎就是开始正式运算了。将字节码指令解释/编译为对应平台的本地机器指令才可以。

### 半解释、半编译
#### [[#前端编译器 -> javac]]
前端编译器：将 `.java` 文件转化为 `.class`
#### [[#解释器]]
解释器：将字节码逐行翻译成机器码（解释器，慢）
#### [[#后端编译器 -> JIT]]
后端编译器：JIT编译器，将字节码转化为机器码（提升解释速度，热点代码，动态编译）
	AOT编译器：静态编译，先全部将字节码翻译成机器码（.java -> .class -> .so），再执行！ 缺点：没有跨平台，不能优化

半编译、半解释型语言：热机快，启动时能马上利用解释器去响应用户，之后执行多轮后，JIT缓存了大量热点代码，就不走解释器了，走JIT编译器了，聪明啊！
联想到  ![[redis#^fe08f1]]
方法调用计数器 + 回边计数器（循环次数）是否超过阈值，超过的话加入JIT编译。否则就走解释器

热度衰减：如果不加这个机制，只要运行时间够久，所有方法都会达到阈值进入JIT，这不是我们想看到的结果，所以引入热度衰减，在指定范围内如果没达到这个值，就衰减掉一部分计数值。

java -Xint -version        指定使用解释器来执行，interpretor
java -Xcomp -version   指定使用编译器来执行，compile
java -Xmixed -version   默认是混合模式

### 两种不同编译器：C1和C2
JIT有两种编译器：

| 编译器    | 简称  |         |              |
| ------ | --- | ------- | ------------ |
| client | c1  | 32位     | 简短可靠的优化      |
| server | c2  | 64位、服务器 | 激进的优化，执行效率更高 |

## 垃圾回收
Garbage Collection
垃圾：当对象不再被任何指针引用，就被认为是垃圾。
内存动态分配和回收技术是现代语言的标配！（C语言就差点意思，需要手动回收）
==**垃圾回收机制，是Java的招牌！**==
GC：除了释放没用的对象，还可以清除内存中的碎片，占用堆的内存集中到一起，后面集中空出大片的内存可以供我们存放后续对象。加入都是零散的占用，那利用率就很低了！

Q：关于垃圾回收器的工作重点？  #Java八股/JVM 
A：垃圾回收器主要在堆+方法区里运行，不在栈里运行。
堆中的话：
1、频繁在新生代收集
2、很少在老年代收集
3、几乎不在永久区 / 方法区（元空间）收集
栈中不会发生GC，因为FILO机制，也GC不了。
### 垃圾回收算法
#### 判别垃圾算法

| XX      | XX                                                                                                                     |
| ------- | ---------------------------------------------------------------------------------------------------------------------- |
| 引用计数算法  | 拿个变量记录引用计数，IDEA中的Usage；<br>优点：简单<br>缺点：无法解决循环引用（内存泄漏）<br>Java没用这个算法！                                                   |
| 可达性分析算法 | 确定好根GC Roots（局部变量表）,找到根，没有根的判定为垃圾<br>使用的是深度优先算法[[算法-深度优先]]<br>优点：解决循环引用<br>缺点：<br>保证一致性的快照中进行，需要Stop the world，结束线程的执行 |
Q：可以作为GC Roots的对象有哪些？ #Java八股/JVM 
A：==局部变量表、常见异常、类的静态变量、常量==、sychronized持有的锁、
阳哥：
- 虚拟机栈（局部变量表）中引用的对象
- 方法区（元空间）中类的静态属性引用的对象
- 方法区（元空间）中常量引用的对象。例如字符串常量池中的引用
- 本地方法栈中Native方法引用的对象（线程中start()方法）

内存泄漏：明明是垃圾，但是GC缺没有帮我们回收，这种情况就叫内存泄漏。

#### 清除垃圾算法
| XX                          | XX                                                                                                                                |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| **复制算法<br>Copying**         | 标记过程一样，复制一份内存空间，把存活的对象重排在一起，旧的内存空间清空。<br>优点：解决内存碎片化<br>缺点：两倍内存空间、内存地址发生变化。<br>**新生代**中的Eden + SurvivorFrom 和 SurvivorTo 就是使用复制算法！ |
| **标记-清除算法<br>Mark-Sweep**   | 从根出发，标记存在引用的。未被标记的被清除<br>缺点：内存碎片化<br>==忽略，不使用！==      Concurrent Mark Sweep  CMS                                                  |
| **标记-压缩算法<br>Mark-Compact** | 标记过程一样，边标记边拼接<br>优点：完美解决上面两种的不足。<br>缺点：效率低于复制算法(先标记后整理)；移动过程中需要STW<br>**老年代**使用标记-压缩算法。                                           |
|                             |                                                                                                                                   |

所谓清除，并不是真的置空，而是将需要清除的对象地址记录一下，下次有新对象时，覆盖！

三种算法的对比

|      | Mark-Sweep | Mark-Compact | Copying     |
| ---- | ---------- | ------------ | ----------- |
| 速度   | 中等         | 最慢           | 最快          |
| 空间开销 | 少（但有碎片）    | 少（无碎片）       | 需要2倍空间（无碎片） |
| 移动对象 | 否          | 是            | 是           |

#### 分代收集算法
针对不同生命周期的对象，采用不同的收集算法  #Java八股/JVM 
- 新生代：复制算法
- 老年代：标记-压缩算法

增量收集算法：由于垃圾回收需要STW。解决：垃圾收集线程与应用程序线程交替执行。每次垃圾回收只收集一小部分的内容空间。接着马上切换回应用程序线程，如此反复，避免STW。
缺点：吞吐量下降。

System.gc()    手动触发垃圾回收行为full gc
Runtime.getRuntime().gc();     同上

一般不主动调用，因为会STW。除非进行性能基准测试前，清一下！

当一个对象**首次**考虑要被回收时，会调用其[[标准包.类#finalize( )]]]方法，让这个对象讲一下遗言！

#### 内存溢出
内存溢出：垃圾回收的速度赶不上内存被占用的速度，就报OOM

| Error                                       | 说明        |          |
| ------------------------------------------- | --------- | -------- |
| OutofMemoryError:Heap                       | 堆空间的内存不够了 | [[#堆溢出]] |
| OutofMemoryError:MetaSpace                  | 元空间的内存不够了 |          |
| OutofMemoryError:GC overhead limit exceeded |           |          |

#### 内存泄漏
站着坑位，不作为；原本应该被GC，但是GC不掉，一直在内存里存着，就叫内存泄漏。

Q：内存泄漏的情况？  #Java八股/JVM 
1、静态集合类
长生命周期的对象持有短生命周期对象的引用，造成短生命周期对象不能被回收。
```
public class MemoryLeak{
	static List l1 = new ArrayList();  //静态变量
public void oomTest() {
	Object obj =  new Object();    //局部变量
	l1.append(obj);              //obj回收不掉了
	}
}
```

2、单例模式
和静态集合类相同，当一个单例对象持有局部对象引用，就会导致这个局部对象回收不掉。

3、内部类持有外部类

4、各种连接，数据库库连接、IO连接等
当与数据库建立连接，完成操作后，如果没有手动close()，就会导致不再使用的连接对象无法被GC掉。

5、变量不合适的作用域
```
public class UsingRandom {
	private String msg;

	public void receiveMsg(){
		readFromNet();
		saveDb();
		// msg = null;   // msg声明为成员变量，而不是局部变量，导致GC不掉
	}

}
```

6、改变哈希值 HashSet

7、缓存泄漏
缓存中的数据，不使用，但是GC不掉，就是泄漏
可以使用WeakHashMap（弱引用），一般用于缓存。

HashMap    是强引用
WeakHashMap   [[Java_03_高级特性#【实现类】WeakHashMap]]

8、监听器和回调

安全点：当你执行System.gc()，不会马上执行，会在安全点才会进行GC
安全区域：一段区域

### 垃圾回收器
垃圾回收线程的并行与并发   [[并发与并行#并行与并发]]

垃圾回收器的并行是指：多个垃圾回收器线程同时在运作
垃圾回收器的并发是指：垃圾回收器线程与应用程序线程同时在执行。

串行 VS 并行  影响吞吐量
并发 VS 独占式   影响响应时间

Minor GC   ->   针对新生代的垃圾回收器
Major GC    ->    针对老年代的垃圾回收器
Full GC       ->    整个堆空间进行GC           Full GC = Minor GC + Major GC
	当频繁出现Full GC，应用程序会变卡顿，就快好挂了！OOM

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/IMG_1904.jpg)

| 参数                                          | 新生代（算法）                          | 老年代（算法）                   | XX                                            |
| ------------------------------------------- | -------------------------------- | ------------------------- | --------------------------------------------- |
| -XX:+UseSerialGC                            | SerialGC<br>（复制算法）<br>           | ~~SerialOldGC~~<br>（标记整理） | 其他线程停STW，让我单线程执行GC<br>类比，饭店里，让所有顾客停止，一个清洁工打扫  |
| -XX:+UseParNewGC                            | ParNew<br>（复制）<br>               | SerialOldGC<br>(标记整理)     | 其他顾客继续吃饭，我们在边上打扫，不影响你<br>停顿比较低，与业务线程一起执行      |
| -XX:+UseParallelGC<br>-XX:+UseParallelOldGC | Parallel(Scavenge)<br>复制<br><br> | ParalelOldGC<br>（标记整理）    | 其他线程停STW，让多个线程程执行GC<br>类比，饭店里，让所有顾客停止，多个清洁工打扫 |
| -XX:+UseConcMarkSweepGC                     | ParNew                           | CMS<br>（标记清除）             |                                               |
| -XX:+UseG1GC                                | G1GC                             |                           | 将堆空间分割不同区域                                    |

查看默认GC
-XX:+PrintCommandLineFlages     查看命令行相关参数（包括使用的垃圾回收器）

jps   查看正在运行的java进程
jinfo -flag UseParallelGC pid    查看pid进程是否使用ParallelGC，+表示是，-表示否


补图

| XX                                         | XX                    |
| ------------------------------------------ | --------------------- |
| -XX:UseSerialGC                            | 单CPU或小内存，单机程序         |
| -XX:+UseParallelGC<br>-XX:UseParallelOldGC | 多CPU，大型运算，少客户端交互      |
| -XX:+UseConcMarkSweepGC<br>-XX:+ParnweGC   | 多CPU，低延迟，多客户端交互，互联网应用 |
#### Serial
Serial （复制算法）+ Serial Old（标记-整理算法）
串行垃圾回收器，STW


#### Parallel
Parallel Scavenge（复制算法）、Parallel Old

JDK 1.8默认使用此！

主打：吞吐量大！适用于服务端（自适应调节Eden、Servivor、老年代的比例、可控制吞吐量）
适用场景：订单处理、工资支付、科学计算

-XX:ParallelGCThreads     设置年轻代并行收集器的线程数，默认小于8
-XX:GCTimeRatio 99  设置吞吐量，默认99，应用程序线程占比99%，垃圾回收线程占比1%
-XX:MaxGCPauseMills     设置垃圾收集器最大停顿时间（STW时间）谨慎设置，别影响吞吐量
-XX:+UseAdaptiveSizePolicy     开启自适应比例(Eden : SurvivorFrom：SurvivorTo)，默认启用（CMS里默认不启用）
	不要和SurvivorRatio参数同时使用。
	要么用SurvivorRation指定，要么就是自适应。


#### ParNew
Serial的多线程版本，其余和Serial都一样。

一般和CMS搭配使用。新生代用ParNew，老年代用CMS

#### CMS
Concurrent-Mark-Sweep
标记-清除算法

主打低延迟，老年代GC，搭配：新生代GC：ParNew
第一款并发垃圾回收器，让应用程序线程与垃圾回收线程同时工作，适用于客户端

CMS操作步骤
1、初始标记 （==STW==，只标记与GC Roots直接关联的对象）
2、并发标记（并发，与应用程序线程同步执行，遍历整个对象图，最耗费时间）
3、重新标记 （==STW==，由于上一步是并发的，线程不安全，修正上一步的错误）
4、并发清理 （并发，清理垃圾）

Q：CMS为什么不用标记-整理算法？ #Java八股/JVM 
A：因为清理过程和应用程序线程并发的，内存地址是不能动的，整理算法会变更内存地址。

| CMS优点  | CMS缺点                     |
| ------ | ------------------------- |
| 1、并发收集 | 1、标记-清除算法，会产生内存碎片         |
| 2、低延迟  | 2、对CPU资源要求比较高             |
|        | 3、无法处理浮动垃圾：原来可达的对象，突然不可达了 |

-XX:+CMSInitingOccupanyFraction  设置堆内存使用率的阈值，一旦达到该阈值就开始GC
-XX:+UserCMSCompactAtFullCollection   执行完full gc后对内存空间进行整理，避免内存碎片积累过多。

#### G1
随着业务复杂，CMS性能不够用了。

用不同种类的region表示新生代、老年代。
同类的老年代，按占比优先级，执行优先级高的。Garbage First

Region之间是复制算法；整体上看是标记-整理算法。

G1操作步骤
1、初始标记（==STW==，只标记与GC Roots直接关联的对象）
2、并发标记（并发，与应用程序线程同步执行，遍历整个对象图，最耗费时间）
3、最终标记（==STW==，遗留少量SATB记录）
4、筛选回收：多条收集线程并行处理

用户指定期望停顿时间是G1的一个强大功能，一般设置为100-300ms。

补图：格子图

-XX:+UseG1GC                 开启使用G1垃圾回收器
-XX:G1HeapRegionSize    设置每个Region的大小，值是2的幂（一般32m），默认2048个区域
-XX:MaxGCPauseMills      最大停顿时间，一般100毫秒


优化步骤
1、开启G1垃圾收集器
2、设置堆的最大内存
3、设置最大停顿时间

一个web应用，java进程最大堆4G，每分钟1500个请求，45s年轻代的垃圾回收，31小时使用率达到45%，则开始并发标记，就那些混合回收。

G1垃圾回收过程：
1、年轻代GC
2、并发标记过程
3、混合回收：全部新生代，部分老年代（占比高的First）
4、FullGC

主流组合：
parallel GC ; 
ParNew + CMS ;
G1

ZGC（2019年。JDK13，未来默认！）

#### 性能指标
1、吞吐量 = 运行用户代码时间/系统运行总时间(运行用户代码时间+运行垃圾收集时间)
例如总共运行100分钟，GC用了1分钟，系统运行花了99分钟，吞吐量就是99%。
一般不能低于95%，越大越好。

2、暂停时间：执行垃圾收集时，应用程序的工作线程被暂停的时间
对于每个GC，都会有暂时时间，因为各组GC的组合
越低 = 低延迟，现代GC最关注的指标。

3、内存占用：堆空间所占内存的大小

### GC参数
[[#参数速查表]]

| XX                         | XX                 |
| -------------------------- | ------------------ |
| ~~-XX:+PrintGC~~           | ~~简单版，一般不用~~       |
| -XX:+PrintGCDetails        | 控制台打印详细信息          |
| ~~-XX:+PrintGCTimeStamps~~ | ~~增加时间戳，以及花费了多少秒~~ |
| -XX:+PrintGCDateStamps     | 增加年月日时分秒，一般用这个     |
| -XX:+PrintHeapAtGC         | 在垃圾回收之前和之后堆的占比情况   |
| -Xloggc:log/gc-oomHeap.log | 生成日志文件到指定目录        |
|                            |                    |

```
// 输出控制台，包含时间戳
-XX:+PrintGCDetails
-XX:+PrintGCDateStamps

//堆空间200M，元空间64M
-Xms200M -Xmx200M 
-XX:MetaspaceSize=64m

//输出dump.hprof文件
-XX:+HeapDumpOnOutOfMemoryError 
-XX:HeapDumpPath=heap/heapdump.hprof

//输出log日志文件
-Xloggc:log/gc-oomHeap.log
```

### GC日志分析
#### 日志文件：log
gc在线分析工具   https://gceasy.io/

```
[PSYoungGen: 21516K->2928K(26624K)] 38805K->22271K(97792K), 0.0097385 secs]
新生代       GC之前   GC之后 总大小

(Ergonomics)                自动调节比例：Eden、SurvivorTo、SurvivorFrom的比例

(Allocation Failure)        分配失败
```


补图

#### 堆转储文件：dump.hprof
dump.hprof 堆转储文件。包含堆内存快照信息，用于分析内存泄漏等情况。
当出发OutOfMemoryError时，就会生成对应dump.hprof文件

## JVM性能监控工具

### 命令行
jps     查看正在运行在JVM中java进程
```
jps

>>输出： 12696 StackAllocation
>>       4072  jar
>>       …… 

jps -l   详细信息，包含启动路径，便于不清楚的情况下，定位
```
![[SpringBoot#^3015e1]]


jstat -gc   查看JVM统计信息，主要查看GC的情况
```
jstat -gc pid 1000 10

查看pid这个进程号的gc情况，在1秒=1000毫秒，打印10次

```

jinfo -flag 参数名 pid  主要查看JVM的配置
```
jinfo -flag 参数名 pid

jinfo -flag DoEscapeAnalysis 12696

>> 输出：-XX:+DoEscapeAnalysis
```

jmap -dump     生成堆转储文件
    -heap        输出整个堆空间的详细信息，包括GC使用，堆配置信息、内存
     -histo       输出堆中对象的统计信息，包括类、实例数和合计容量
```
jmap -heap pid      查看堆空间的使用情况，新生代（Eden、From、To），老年代
```

jstack   打印JVM中线程快照
[[#CPU占用很高排查方案]]
```
jstack pid > jstack.log          将线程的详细信息输出到log文件中
```

### GUI

| XX                    | XX                        |
| --------------------- | ------------------------- |
| ~~jConsole~~          | jdk软件包中自带的，这个太low了        |
| JProfiler             | 查看有没有死锁、内存占用等，付费 @鱼皮      |
| Visual VM             | jdk软件包中自带的（推荐使用)          |
| mat                   | 分析JVM内存使用情况，eclipse家的 @鱼皮 |
| Arthas                | Alibaba开源的Java诊断工具，在线实时。  |
| [[SpringBoot#Jmeter]] | 模拟高并发的Http请求，压力测试<br>     |

`java -jar arthas-boot.jar pid`    
进入`arthas@pid`的linux命令行，输入各种命令进行查询
[Arthas文档](https://arthas.aliyun.com/doc/)

## JVM调优案例
1、避免OOM；2、解决已发生的OOM；3、优化提升速度

调优的大方向：
1、合理编写代码
2、充分并合理利用硬件资源
3、合理进行JVM调优

调优的依据：
- 运行日志                      日志文件log  会覆盖之前的（实际开发中，会拆分到不同）
- 堆转储文件dump.hprof         不会覆盖
- 线程快照
- 堆栈快照

| 文件      | 工具                        |
| ------- | ------------------------- |
| log日志   | gceasy.io                 |
| dump堆转储 | jvisualvm  jprofiler  MAT |
|         |                           |
|         |                           |

### OOM汇总
java.lang.Object.Throwable.Error.VirtualMachineError
OOM是错误，不是异常！

| XX                 | XX                                 |                                                         |
| ------------------ | ---------------------------------- | ------------------------------------------------------- |
| StackOverflowError |                                    | 虚拟机栈，内存不够了                                              |
| OutofMemoryError:  | Java heap space                    | 堆内存不够了                                                  |
|                    | GC overhead limit exceeded         |                                                         |
|                    | Direct buffer memory               |                                                         |
|                    | ubable to create new native thread | 普通Linux用户，单个进程（程序实例）最多创建1024个线程，当你的java程序创建了超过这个数字，就会报！ |
|                    | Metaspace                          | 元空间不够了                                                  |


### 堆溢出
java.lang.OutOfMemoryError: Java heap space

### 元空间溢出
java.lang.OutOfMemoryError: Metaspcae
主要GC的方向是：常量池和对类型的卸载

### GC overhead limit exceed
如果发现，在GC过程中，发现超过98%的时间在做GC，并且回收不到2%的堆内存，就会出现GC overhead limit exceed。一种预判性异常。

-XX:-UseGCOverheadLimit    添加减号，禁用这个检查！最终大概率会报OOM

### 线程溢出
java.lang.OutOfMemory:ubable to create new native Thread
![[#^ee16cb]]

能创建的线程数 = （最大寻址空间 - JVM内存 - 保留操作系统内存）/ThreadStackSize
32位有效，因为最大寻址空间，2^32，4G
64位情况下，最大寻址空间，2^64，约等于==17179869184G==，可以看做无穷大，公式就不生效了，操作系统限制最大线程数：1.5w个

结论：设置-Xss1024k和512k没影响，因为都是上限就是1.5w


Q：性能评价和测试指标？
Q：生产环境应该给服务器分配多少内存合适？
Q：如何对垃圾回收器的性能进行调优？
Q：生产环境CPU负载飙高该如何处理？
Q：生产环境应该给应用分配多少线程合适？
Q：不加log，如何确定请求是否执行了某一行代码
Q：不加log，如何试试查看某个方法的入参和返回值？

### JIT优化
后端编译器，将字节码 编译为 机器码（JIT会缓存热点代码）。

不是所有代码都需要走JIT编译，有些只执行一次的代码直接走解释型即可，不用走JIT编译器。

逃逸分析    默认开启    -XX:-DoEscapeAnalysis    指定不开启。
如果开启了逃逸分析，就会判断方法中的变量是否发生了逃逸，没有逃逸就会使用栈上分配。

标量替换：？？？？   -XX:+EliminateAllocations   默认开启。
	只有在逃逸分析开启情况下，才可以设置这个选项。


### 合理配置堆内存
不能给堆内存分配太大空间，Full GC时会卡！

| XX  | XX                                                           |
| --- | ------------------------------------------------------------ |
| 堆空间 | 进行Full GC之后，老年代存活下来的容量的3-4倍！<br>Old Generation Used Capacity |
| 元空间 | 进行Full GC之后，老年代存活下来的容量的1.2-1.5倍                              |
| 新生代 | 进行Full GC之后，老年代存活下来的容量的1-1.5倍                                |
| 老年代 | 进行Full GC之后，老年代存活下来的容量的2-3倍                                  |
如何计算Full GC之后，老年代存活下来的容量？
1、查看log日志文件；
2、强制触发full gc（慎用，web应用会暂停）
	jmap -histo:live pid       执行生成dump文件，会顺带触发full gc！
	jmap -heap pid             显示full gc之后，堆空间占比情况

新生代与老年代的比例：
结论：面对外部大流量，低延迟系统，建议关闭此参数。（会自适应把老年代加得很大，导致进行一次FullGC耗时很长） 
手动关闭   `-XX:-useAdaptiveSizePolicy`
检查是否生效 `jinfo -flag useAdaptiveSizePolicy pid`   

### CPU占用很高排查方案
死锁会导致CPU飙高的情况（CPU频繁切换）
Q：系统CPU经常100%，如何调优？ #Java八股/JVM 
A：如下步骤排查：
ps aux | grep java         查看当前java进程使用cpu、内存、磁盘情况，获取使用量异常的进程
top -Hp 进程id             根据进程id，查看各个线程，定位异常线程的pid
把pid转为16进制
jstack 进程id | grep -A20  线程id(16进制)         得到相关进程的代码（-A20，after20，之后20行）

### G1并发GC线程数对性能的影响
当前优化条件的配置信息
![[JavaWeb#^dbaf3a]]
```
export CATALINA_OPTS="$CATALINA_OPTS -XX:+UserG1GC"        # 使用G1GC垃圾回收器
export CATALINA_OPTS="$CATALINA_OPTS -XX:ConcGCThreads=2"  # 并发GC线程数2
```

-XX:ConcGCThreads=1        设置并发标记的线程数，将n设置为并行左右

`jinfo -flag ConcGCThreads pid`   查看这个线程的G1并发GC回收线程数

结论：G1并发GC线程数对性能主要影响的是：吞吐量。

不同垃圾回收器对业务的影响不同
G1能提高业务系统的吞吐量（运行业务代码时间，运行GC时间的占比）

### 运行百万级订单交易系统JVM参数设置
例如4小时产生300万订单量。

300万/4/3600秒 = 208.3单/秒

按300单/秒进行计算。

按3台服务器部署（每台4核8G)，每台机器要求每秒100单

每个订单对象假设1kb（id、string、double），也就是1秒生成100kb数据

对象包含优惠券等，扩大20倍，约为2M对象。

总结：每台服务器，每秒，有2m的数据对象，进入Eden区

8G内存中，4-5G分配给JVM的堆空间，其中新生代占1G，老年代3G

新生代1133M（1G）,Eden区占1/8，

1133M/8/2M(每个订单对象)/60s = 8.8分钟。
意思是，每11.1分钟，就会把Eden区占满，执行Minor GC
注意：MinorGC的触发条件是：Eden区占满！！

新生代可以适当扩大，让对象留在新生代，用完就消亡了，不要进入老年代，因为一旦进入老年代就很难回收了。

业务量暴增的情况
1、服务器扩容，增加到5-10台
2、不扩容：每秒订单数上涨到几千单，每秒有20m的对象，大量对象会移入老年代，老年代一满，就开始Full GC（STW），用户就会感觉卡！

如果需要响应时间控制在100ms，就需要用压力测试Jmeter工具！

Q：12306遭遇春节大规模抢票，如何支撑？
A：分布式本地库存 + 单独服务器做库存均衡！分而治之！ [[SpringCloud#分布式微服务]]


### SpringBoot微服务调优
1、在IDEA中设置VM-Option，再打包
2、在cmd运行时，
```
java -server jvm各种参数 -jar  jar包或war包的路径

java -server -Xms1024m -Xmx1024m -XX:+UseG1GC - jar ./hello.war
```




2024.4.22 完成精简版JVM的学习，太抽象了，老师不好讲，我也学的比较断断续续，不如技术框架好学！

## JMM内存模型
Java内存模型
JVM规范中定义一种Java内存模型（Java Memory Model），屏蔽各种硬件和操作系统的内存访问差异。
以实现Java程序在各种平台下能达到一致的内存访问效果。

JMM是一种抽象概念，并不实际存在，一种接口规范。
主要围绕多线程的原子性、可见性、有序性展开。

能干嘛？
1、通过JMM实现线程和主内存之间的抽象关系
2、实现Java程序在各种平台下能达到一致的内存访问效果。

### 三大特性
#### 原子性
一个操作时不可打断的，多线程环境喜爱，操作不能被其他线程干扰

volatile关键字不能保证原子性，只能保证可见性和有序性（禁止指令重排）

思考：为什么volatile关键字不能保证原子性？
回答：因为可见性的即时更新，导致两个线程中一个线程完成了计算，立刻更新回主内存，另一个线程也立刻感知到，它的操作就作废了，导致次数被白费了，导致原子性失效！

#### 可见性
当一个线程修改了共享变量的值，其他线程是否能够立即知道该变更；
JMM规定了所有变量都存储在主内存中。

每一个线程不能直接操作主内存中的共享变量，要读一份共享变量的副本回线程内，修改完，再提交回来！
由于副本拷贝机制，不同线程之间无法直接对方线程中的变量，需要通过主内存完成传递！
![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202407312130904.png)


#### 有序性
指令重排序
你写代码的时候是1-2-3-4，指令执行的时候有可能是2-4-1-3

JMM规定：尽量维持顺序化语义，但是如果程序最终结果和顺序执行相同，（因为单线程里，重排也没多大影响）==允许与代码顺序不同==，此过程就叫指令重排！（最大限度的发挥CPU性能）
> 指令重排能保证串行时（单线程）语义一致；但是==没有义务保证多线程下的语义一致==（有可能产生脏读）

多线程环境下，需要==额外使用voletile关键字明确禁止指令重排==，保证数据的依赖性！

### 先行发生原则 happens-before
线程1的操作结果，需要对另一个线程可见；
或者在不影响结果的前提下，代码重排序。都需要遵循此原则。
包含可见性和有序性（指令重排）的约束。

如果一个操作先行发生在另一个操作，那么第一个操作的结果对第二个操作可见，而且第一个操作的顺序一定排在第二个操作的前面；
如果最终结果一致，可以不遵循happens-before的顺序，合法！如果结果不一致，重排会禁止！

例如：安排周一：张三  周二：李四 值日
周一打扫完的结果，对周二的李四可见；如果李四有事，可以允许调换。

### 内存屏障
happens-before原则的底层实现！

通过内存屏障实现volatile的可见性和有序性
1、Store Memory Barrier   内存屏障之前的所有写操作都要写回主内存
2、Load Memory Barrier   内存屏障之后的所有读操作都能获得内存屏障之前写操作的最新结果（实现可见性）



在下面的案例中，有两个线程读取同一个类中的isStop属性字段，如果不加volatile关键字，t2线程对isStop的修改，t1线程是不可见的，t1只能看到之前的值！
加上volatile关键字后，这个变量一有变动，马上更新到住内存中，对其他线程马上就能读取到！
```
public class InterruptDemo {  
    static volatile boolean isStop = false;  
  
    public static void main(String[] args) {  
        new Thread(()->{  
            //线程需要执行的逻辑
            if(isStop)
        },"t1").start();  
  
        new Thread(()->{  
            //线程需要执行的逻辑  
            isStop = true;  
        },"t2").start();  
    }  
}
```
