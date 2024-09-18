面向对象编程，一种编程范式！核心就是：基于类生成实例化对象，操作对象的State（属性）、Method（方法或函数）。 

### 类和对象
联想：Python中的[[类]]，概念是一样的，通过Python还是Java去实现而已！
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/IMG_1032.PNG)

一个类包含：
	状态（Python中称为属性，Java中称为Fields）
	方法（Java中称为Methods）。当执行方法时，Fields的值会变化更新。

如何确定哪些应该定义为一个类？
一个类，应该包含明确的，区别于其他类的独立职责！（类似餐厅的：主厨、服务员、收银员）

#### 类 Class
Step 1：在Module（me.shen1993这个项目文件夹）右键，新建class
Step 2：在自动生成的Class模板代码中声明：fields 和 method

默认的类的结构：
	field  
	constructor 
	method override
	public method
	setter & getter
![[Java_01_基础#^c62eaf]]

```
public class TextBox {  
    public String text;  // field  

	// method
    public void setText(String text) {  
        this.text = text;    
        // field和方法的参数重名了，需要this关键字指定哪个是field！
    }  
  
    public void clear() {  
        text = "";    // 不用this，因为没重复  
    }  
}
```

##### 线程 -操纵-> 资源类
~~领域驱动设计Domain- Driven Design（简称DDD），明确业务和应用的边界。分为两种类：领域类和服务类。领域类+服务类 营造一个场景。之后再让具体的人来使用这个场景~~
1. ~~==领域类（Domain Class）==：关注业务逻辑，它描述了领域中的一个概念，通常具有字段（field）和方法（method），用于表示概念的属性和行为。~~
2. ~~==服务类（Service Class）==：关注具体应用，它是一种功能类，负责封装一系列领域类的实例化和操作方法，用于提供系统的功能和服务。服务类通常与领域类交互，基于领域对象执行业务逻辑。~~

如下图：
1、定义了账户类（资源类），描述账户的概念，字段：余额，行为：存钱、取钱；
2、在服务类（main方法，运行的线程）中：ExceptionDemo类中实例化了Account类的对象，并封装了取钱的逻辑
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/05d112dce055a38ca31567ca4f97057c.PNG)

示例代码
```
// Phone作为资源类
class Phone {  
    public synchronized void sendEmail(){  
        System.out.println("------send email------");  
    }  
    public synchronized void sendMSM(){  
        System.out.println("------send msm------");  
    }  
}  

// main作为所有线程的起始点！在线程里操作资源类！
public class Lock8Demo {  
    // 一切程序的入口，各种线程启动的地方  
    public static void main(String[] args) {  
        Phone phone = new Phone();  
  
        new Thread(() -> {  
            //具体业务逻辑  
            phone.sendEmail();  
        },"t1").start();  
          
        new Thread(() -> {  
            //具体业务逻辑  
            phone.sendMSM();  
        },"t2").start();  
    }  
}
```

##### this关键字
注意：this关键字，是指“类当前实例自己”。  联想到：**python中的**[[类#self 变量]]

（构造函数中）当Field字段和传递的参数一样的时候，必加this,用以区分Field字段和参数字段
```
public class VideoProcessor {  
    private VideoEncoder encoder;  
    private VideoDatabase database;  
  
    //自定义构造器  
    public VideoProcessor(VideoEncoder encoder, VideoDatabase database) {  
        this.encoder = encoder;          // 构造函数中，必加this,用以区分Field字段和参数字段  
        this.database = database;  
    }
```

使用场景：
1、区分局部变量和成员变量，当方法提供的参数名与成员变量名相同，使用this关键字明确这个是类的成员变量！
2、在构造方法中调用另一个构造方法。当一个类有多个构造方法时，可以使用this关键词在一个构造方法中调用另一个构造方法。
##### 类的通用方法
[[标准包.类#通用方法]] 
###### equals()
重写时，同时重写hashCode()
不要小看这个方法，有很多花头！

【强制】所有整型包装类对象之间值的比较，全部使用equals方法比较。 #Java编程规约 ^0524d7

对于 Integer var = ? 在-128 至 127 之间的赋值，Integer 对象是在 IntegerCache.cache 产生， ==会复用已有对象==，这个区间内的 Integer 值可以直接使用`==`进行判断，但是这个区间之外的所有数据，都会在堆上产生，==并不会复用已有对象==，这时对于包装类来说，就是两个对象，即便value都是1000，这是一个大坑，推荐使用 equals 方法进行判断。
[`为何1000==1000为false，而100==100却为true？`](https://mp.weixin.qq.com/s?__biz=MzU0OTk3ODQ3Ng==&mid=2247614530&idx=2&sn=47a721bbf6266828e765137800bcea55&chksm=faa151c89fc07c1f5638c1c11d22a8aff51b81f53bde869acf2ab57706c5d68d8cd6da1bcddd&from=industrynews&version=4.1.27.6032&platform=win&nwr_flag=1#wechat_redirect)


 【强制】Object的equals方法容易抛空指针异常，应使用常量或确定有值的对象来调用equals。 #Java编程规约
 
![[Java_01_基础#^3ee420]]
```
正例:"test".equals(object);    // 假如object是空，返回false!

反例:object.equals("test");    // 假如object是空，报空指针异常

```

###### toString()

###### hashCode()

###### getClass()

##### 内部类
内部类对象持有对外部类对象的引用，并且可以直接访问外部类的成员和方法。

![[#^876f5b]]

另一个经典的内部类的例子：一个购物车类，里面有购物车项类（包含一个商品和对应数量）
```
public class ShoppingCart {
    private List<CartItem> cartItems;
    ……
	private class CartItem {
	    private Item item;
	    private int quantity;
		// 构造方法
        public CartItem(Item item, int quantity) {
            this.item = item;
            this.quantity = quantity;
        }
    }
```



##### enum 枚举类
定义一组常量，将一组相关的常量捆绑在一起。
例如Web工程中的状态码+信息

枚举类在声明的时候，就已经写死了“枚举常量”，也就是只有这几个实例，不能有其他的！这就限制了调用者传递的数据范围。

Q：枚举类，为什么不能用`final int MAX_VALUE` ?
A：用常量只能限制数据类型为int，传递的值不能限制；enum的话，传递的值固定就是这几个，其他都不行！

枚举类的实例的唯一性。每个枚举常量就是该枚举类型的单一实例。每个枚举常量都有自己的属性和方法。
构造函数，默认是private，仅类内部构造，外面直接使用就行了！

手写枚举类的步骤：
1、enum关键字，创建枚举类
2、列出所有枚举常量（大写），已逗号分割
3、每个枚举常量可以括号添加属性值。
（思考：每个枚举常量就是一个实例对象，对象当然可以用Field值拉！）
4、添加构造函数、方法、属性！

```
public enum Color {
	RED,
	GREEN,
	BLUE;
}

RED 是这个枚举类的一个实例，称之为：枚举常量！
```

```
public enum ResultCodeEnum {  
    SUCCESS(200,"success"),  
    USERNAME_ERROR(501,"usernameError"),  
    PASSWORD_ERROR(503,"passwordError"),  
    NOTLOGON(504,"notlogin"),  
    USERNAME_USED(505,"userNameUsed");  
  
    private Integer code;  
    private String message;  
   
    ResultCodeEnum(Integer code, String message) {  
        this.code = code;  
        this.message = message;  
    }  
  
    public Integer getCode() {  
        return code;  
    }  
    public String getMessage() {  
        return message;  
    }  
}
```

#### 对象 Object
##### new 声明对象
分为三步：
1、声明一个对象，包括对象类型，对象变量名。
2、使用new关键字来实例化一个对象。
3、将实例化的对象，赋给声明的对象。也就是建立Stack内存与Heap内存的引用。
4、使用new关键字时，调用[[#构造方法 Constructor]]初始化

注意：声明对象的类型，和实例化的对象的类型，不一定是相同的哦。也可能是父类与子类。详见：[[#原则4：多态性]]
```
TextBox textBox1 = new TextBox();

Person p1 = new Man();     // 等式左右的类型是不一样的。
```

可以只声明一个变量，也可以声明+初始化。取决于你是否已经有合适的初始化值。
```
private MortgageCalculator calculator;      //声明语句，在使用前必须初始化它
calculator = new MortgageCalculator(principle, annualInterest, years)；

private MortgageCalculator calculator = new MortgageCalculator(principle，annualInterest, years);                    // 声明+初始化，一套带走
```

```
Person p1 = new Man();

> p1能执行的方法，看的是Person类中定义了哪些方法
> 加入Person类中的某个方法被Man类重写了，那么会执行子类中被重写的。
> 
```

Q：如果只new一个最小的Object，这个对象多大？
A：16字节（只有对象头）[[JVM#对象头]]

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B44%E6%9C%8817%E6%97%A5%2022_36_33.PNG)

关联：[[#堆内存 Heap]]  [[#栈内存 Stack]]


总结：除了new造一个对象，还有其他方式，也可以获取对象哦！

| XX    | XX                                                  |
| ----- | --------------------------------------------------- |
| new   | Dog dog = new Dog();                                |
| 多态    | Animal a= new 鸡鸭猫狗;<br>List list = new ArrayList(); |
| IOC容器 | Spring容器+池化技术                                       |
| 设计模式  | 通过工厂模式，传递参数从工厂模式中return一个实例                         |

##### var关键字
var关键字，让Java根据右边的new实例化的类，决定这个变量的类型。

如果想要查看这个变量是什么类型的，IDEA：鼠标点在变量上，View/Quick Documentation
就能显示出它被判断为什么数据类型！

```
var textBox1 = new TextBox();
```
等价于
```
TextBox textBox1 = new TextBox();   // 这里左右都有TextBox类，有点重复，用var关键字
```

##### 创建对象的方式
###### 1、构造器实例化 -> new
new关键字，其实就是调用构造器来实例化

有些情况下，类的构造器会被声明为private，无法调用构造器实现实例化，这种情况下，一般会提供XxxBuilder类（或XxxFactory类）的静态方法，来执行实例化的功能！这种就叫：工厂模式
[[设计模式#Factory Method]] ^e14ede
###### 2、[[设计模式#Factory Method 工厂模式]]实例化  -> return
**静态工厂方法**：是一种基于class创建对象的方式，通常用于提供更灵活、更清晰的创建对象的实现。 ^096eea
```
// 静态工厂模式  
public class ClientService {  
    // Field  
    private static ClientService clientService = new ClientService();  
    // 私有静态Field，单例模式  
    
    // 构造器  
    private ClientService() {}      // 将构造器设置为private，外部无法通过new实例化，毕竟是工厂模式！  
  
    public static ClientService createInstance(){   // 公共静态方法，利用无参静态方法来实例化  
        return clientService;  
    }  
}
```

List接口中的of，就是静态工厂方法！可以快速创建不可变的集合对象 ^23f9f5
```
List<String> list = List.of("apple", "banana", "orange");
```

of工厂方法的源代码
```
// 构造器
public static <E> List<E> of(E... elements) {  
	return new ImmutableCollections.ListN<>(elements);  
}
```

实例工厂方法：
Q：基于实例化对象？？已经实例化对象了，怎么还要工厂方法呢？
A：该类的构造器声明为private，所以需要实例工厂方法！
其实实例工厂方法创建实例化对象的过程是先new一个实例工厂对象，然后在工厂对象基础上调用方法来创建真正需要的实例！
###### 3、ioc容器
加入ioc容器，需要的时候，由ioc容器管理并提供（单例模式），并组织管理它的生命周期

###### 4、反射 -> getInstance()
利用反射，基于类模版，获取一个实例对象

##### 属性 Field
属性/状态 
1、通过访问修饰符来控制Field的可见性。
2、必须指定数据类型
3、Field可以在声明时不初始化，，延后到构造函数中进行初始化。
`private NumberFormat currency;`
4、Field的作用域是整个类，可以在类的任何地方被访问和修改。
5、每个实例化对象都有自己的Field值。除非声明为static，那就是类的Field

经验：对属性的值的修改，通常用setter方法来修改，不要用num++！
[[设计模式#迪米特法则]]
![[设计模式#^fa7c20]]

符合面向对象的[[#原则1：封装]]
![[#^53100d]]

恍然大悟：所以在贷款计算器中，将重复的逻辑，Extract成为Field
```
String mortgageFormatted = NumberFormat.getCurrencyInstance().format(mortgage);

System.out.println(NumberFormat.getCurrencyInstance().format(balance));
```

```
private final NumberFormat currency;    //声明时可以不初始化，延后到构造函数中进行初始化

// 构造函数，进行初始化！
public MortgageReport(MortgageCalculator calculator) {  
    this.calculator = calculator; 
    currency = NumberFormat.getCurrencyInstance();  
}
```
##### 实例化与初始化的区别
![CleanShot 2024-07-25 at 23.55.57@2x.png](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-07-25%20at%2023.55.57%402x.png)

实例化：在堆中开辟一块空间，属性都是默认值 ^c02f1c

初始化：
	1、给属性完成赋值操作！填充属性(赋值操作)
	2、==调用具体初始化(前后)方法==(initMethod)  ^1cda5e

python中对实例化和初始化是分开的：[[类#`__new__`和`__init__`]]

补图！

### 内存分配机制
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B44%E6%9C%8810%E6%97%A5%2011_25_46.PNG)

我们在前文知道：
![[Java_01_基础#^3916a6]]
![[Java_01_基础#^fa8365]]

| 名称     | 内存分配           | 存储方式                         | 复制方式            |
| ------ | -------------- | ---------------------------- | --------------- |
| 基元数据类型 | 不用new创建实例，直接赋值 | 直接存储在栈内存中                    | 真的复制一份          |
| 引用类型   | 需要先new创建实例     | 存储在堆内存中，变量本身存储在栈内存，指向堆内存中的对象 | 引用地址复制一份，值并不复制。 |

```
var textBox1 = new TextBox();
var textBox2 = textBox1;  
textBox2.setText("Hello World");  
System.out.println(textBox1);  
System.out.println(textBox2);

>>输出:
>>me.shen1993.TextBox@4eec7777     textBox1的内存地址 和 textBox2的内存地址是完全一样的！
>>me.shen1993.TextBox@4eec7777
```

所谓垃圾回收机制：
进程1：当退出方法时，会自动将Stack（[[数据结构_栈]]）上相关变量全部清除掉；
进程2：Heap（[[数据结构-堆]]）上的对象，一段时间没有被引用，也会被清楚掉。
这就是Java垃圾回收机制，通过garbage collector类实现！

思考：为什么Java用栈来存放短期变量，用堆来存放类的实例化对象？
回答：堆空间大，栈空间小；栈的运算速度比堆快；栈空间管运算，堆空间管存储。
引用数据类型比较大，不太可能浪费宝贵的栈空间，只保存一个地址；
基本数据类型有限且固定，直接存在栈空间中。

#### 堆内存 Heap
用于==**存储动态分配的对象**==的内存区域

堆的优点是可以**动态地分配和释放内存**，但是缺点是对象的存储和访问速度相对较慢。
	对象的大小和生命周期不确定，更需要动态分配和释放内存的特性

堆是一个大的、共享的内存区域，可以被多个线程访问。**共享的**。

#### 栈内存 Stack
用于==**存储局部变量和方法调用**==的内存区域。

每个方法的调用都会创建一个栈帧（包含该方法的：局部变量、参数、返回地址）

LIFO（后进先出），符合方法调用的递进逻辑！栈顶的数据可以直接访问

局部变量在栈上分配内存、**访问速度较快**。
	局部变量需要访问速度快的特性。

每个线程都有自己独立的栈内存。**私有的**。

### 面向对象的基本原则
![[编程范式#^c5c708]]

#### 原则1：封装
Encapsulation：Bundle the data and methods that operate on the data in a single unit.
将==数据和对数据操作的方法==，打包成一个单元！

下面这段简单的计算工资的代码，虽然用到了类和对象，但并没有实现面向对象编程的思想！
这也是普通Java程序员容易忽略的点！

经验：如果方法需要传递5-10个以上的参数Field（将多个相关参数，抽象成一个类的Field），证明你没有在使用面向对象编程，而是过程性编程。
	将baseSalary、extraHours、hourlyRate参数封装成一个类，叫做employee
	将参数[[抽象]]成一个单独的类，class Employee
```java
public class Main {
	public static void main(String[] args) {
        int baseSalary = 50_000;  
        int extraHours = 10;  
        int hourlyRate = 20;  
        
        int wage = calculateWage(baseSalary, extraHours, hourlyRate);  
        System.out.println(wage);  
    }  
  
    public static int calculateWage(
	    int baseSalary,
	    int extraHours,
	    int hourlyRate) {  
        return (baseSalary + extraHours * hourlyRate);  
    }  
}
```

```Employee.java
public class Employee {  
    public int baseSalary;     // 不给fields设定具体值，因为是类的蓝图，具体值在实例化对象时传递！  
    // public int extraHours;  // 加班时长每月都不一样，作为参数比较合适，而不是状态！  
    public int hourlyRate;  
  
    public int calculateWage(int extraHours) {    
    // 注意，封装数据和对数据操作的方法，可以减少参数！  
	    return (baseSalary + extraHours * hourlyRate);  
    }  
}
```

```Main.java

public class Main {  
    public static void main(String[] args) {  
        Employee employee = new Employee();  
        employee.baseSalary = 50_000;       // 通过给类的fields赋值的方式传参！
        employee.hourlyRate = 20;  
        int wage = employee.calculateWage(10);  
        System.out.println(wage);  
    }  
}
```

另一个问题，`employee.baseSalary = -1;` 如果有人给fields传递不符合要求的值怎么办？我们可以改为 `employee.setBaseSalary()` 在这个方法中实现数据验证的逻辑。

```
在Main类中，使用set
// employee.baseSalary = 50_000;  
employee.setBaseSalary(-1);


在 Employee类中实现
private int baseSalary;

public void setBaseSalary(int baseSalary) {  
    if (baseSalary <= 0)  
        throw new IllegalArgumentException("Salary can not be 0");  
    else  
        this.baseSalary = baseSalary;  
}  
  
// 由于baseSalary已经被定义为private，外部类中无法访问到，所以需要实现一个getBaseSalary方法  
public int getBaseSalary() {  
    return this.baseSalary;  
}
```

==**总结：**==
1、传递5-10个以上的参数Field（将多个相关参数，抽象成一个类的Field）

2、**将Field设置为Private，并搭配setter和getter来使用**（因为可以在setter函数中加入验证数据的逻辑）  ^53100d


#### 原则2：[[抽象]]
Abstraction：Reduce complexity by hiding unnecessary details
隐藏不必要的细节，降低复杂性。

例如我们使用电视机遥控器，按一下按钮可以换台，under the hook，有电子元器件在工作，我们不用关心电子元器件的状态是什么，如何运转。我们只关心按钮换台这个功能，这就是抽象！ ^08aab9

**耦合 Coupling**
The level of dependency between classes
耦合，就是类与类之间的依赖程度。

通过抽象原则，隐藏不必要的实现细节，只对外暴露必须的一小部份。就可以降低耦合度！
类之间，**不可能是零耦合的**，因为我们把它们创建在一个Module里，就是希望它们合作完成任务的，**我们希望的是减少耦合**！
	Interface接口，在抽象的基础上，进一步降低耦合度！

不使用的方法，去设置为private，不然暴露的方法越多，产生的耦合点就越多。
Field也设置为Private，同样也是隐藏不必要的细节！
![[#^53100d]]


```
private int getBaseSalary() {  
    return this.baseSalary;  
}

private int getHourlyRate() {  
    return this.hourlyRate;  
}
```

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/0dcbe6692126f560e0adb31dcb3d5492.PNG)

对于一个浏览器类，实现了：`findIpAddress(String address)、sendHttpRequest(String ip)、navigate(String address)` 。但是从用户来讲，根本不关注findIpAddress和sendHttpRequest()，只关心最后的navigate()。
```
public class Browser {  
    public void navigate(String address) {  
        String ip = findIpAddress(address);  
        String html = sendHttpRequest(ip);  
        System.out.println(html);  
    }  
  
    private String sendHttpRequest(String ip) {    //隐藏起来，降低耦合度
        return "<html></html>";  
    }  
  
    private String findIpAddress(String address) {   //隐藏起来
        return "127.0.0.1";  
    }  
}
```
好处就是，如果我们想要修改findIpAddress()方法，加一个参数cache，在Main类中的方法不受影响，因为这个方法是private，不暴露给外面，只在类内部生效！

总结：
抽象就是隐藏外部不需要关注的细节！降低与外界的耦合点（设置为private）！只把必要的耦合点与外部接触。

#### 构造器 Constructor
联想到：Python中的 [[类#`def __init__`]]   基于类进行实例化时的函数方法！

构造器：用来初始化对象，给对象的属性赋值！如果不写，JVM自动生成无参构造器，Field值默认Null
Lombok的注解：@AllArgsConstructor 和 @NoArgsConstructor

**==思考：全参和无参构造器的作用？==**
当 `Person p1 = new Person();`时，有了全参和无参构造器，你可以填参数到`new Person(int age, String name)`的括号中，你不知道，不填也没关系。否则就会报红！

构造器发生在new的这句话！
```
Container<String> strContainer = new Container<>();
```

构造器的特点
==1、必须和类同名
2、不需要声明返回类型，连void都不用写！因为它返回的就是构造的实例对象（最特殊）
3、如果不重写，Java会自动生成无参构造器，所有Field为默认值null ==

思考，下例就是重写了构造器，构造器传递一个参数，赋值给Field。
Lombok的@AllArgsConsrtructor全参构造器；@NoArgsConstructor无参构造器
```
@AllArgsConsrtructor
@NoArgsConstructor
public class Container <T> {
    private T t;

    public Container(T t) {
        this.t = t;
    }
}
```

相当于在基于类实例化对象时，进行自动初始化的方法！
通常会使用构造方法给一个==**类的实例变量赋初值**==，或者执行其它必要的步骤来创建一个完整的对象。

![[#^e14ede]]

`var employee = new Employee()`   Employee()这个方法，我们在创建class Employee时并没有创建，是Java自动创建的，用于构造一个新的Employee对象。
	初始化的过程就是**自动设置fields的值**的过程，如果没有重写，java会自动生成，默认值就是primitive Type和Reference Type的默认值。（number默认值是0，Boolean默认是False，Reference Type默认是Null)

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/a5fe2370825ea55a388d9a56a4c464f7.PNG)

```
// 重写构造函数，自定义初始化过程  
public Employee(int baseSalary, int hourlyRate) {    // 构造函数没有返回类型，连void都不用  
    setBaseSalary(baseSalary);  
    setHourlyRate(hourlyRate);  
}


// 在Main类中，new构建实例的时候，通过传递参数，给一个类的实例变量赋初值
var employee = new Employee(50_000,20);


```


![[#^6d67a1]]

##### 构造器前置方法 @PostConstruct
javax.annotation包下定义，JDK自带的注解。
会在构造函数完成之后立即执行被标注了本注解的方法，不需要手动调用。
要求：public void + 无参

#### 方法重载 Overload
==方法：同名不同参，就是重载！==
针对传递不同组合的参数，提供不同的方法。就是相同方法名，重复Copy写多个！

例如：`int wage = employee.calculateWage(extraHour:10)` 加入这个月没有加班，可不可以不传递参数。需要方法重载，来实现不传递参数的方法。

```
public int calculateWage(int extraHours) {    
    return (baseSalary + extraHours * hourlyRate);  
}  
  
public int calculateWage() {    // 针对不传递参数的方法重载  
    return (baseSalary);  
}
```

不要过多使用方法重载，适合的场景是：针对不同数据类型的参数进行方法重载。

同样也可以重载构造，针对有参数和无参数的情况，重载不同的方法！

IDEA的快捷键：ctrl + P 查看针对哪几种参数进行了重载。
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/e3494ee87a8c4d376ac0c99aee1f6a39.PNG)


#### 静态 static
##### 静态类
一个类，有两种选择：
- instance members ：需要进行实例化成具体的对象
- static members：属于类的，直接通过类调用（例如Math类中绝大多数方法，可以通过类直接调用）
```
// 直接通过类Math调用abs()方法
int result = Math.abs(-10);

// 无需实例化成math,再基于对象调用，静态方法完全不用！
var math = Math();  
int result = math.abs(-10);  
```

###### 静态内部类
普通内部类不能隐藏起来，但是静态内部类对外部类的引用是隐式的（不与外部类的实例关联），静态内部类隐藏在内部发挥作用，一般都是对外部类的一种==补充刻画==。 ^876f5b

静态内部类，一般用private进行修饰，private static InnerClass{}

联想到：
	1、链表的类，里面都有节点静态内部类；链表对象内部的节点类。我们只需要实例化链表，内部的节点如何构造我们不关心，可以把节点类设置为静态内部类，隐藏起来！
	2、ThreadLocal类，里面的ThreadLocalMap静态内部类

##### 静态变量 
[[Java_01_基础#静态变量]]
静态字段，直接通过类名来调用，而不是实例。 ` Employee.numberOfEmployee`

static fields用来表示一种概念，它不属于类下任何一个特定实例。
	例如class Employee，有一个fields叫做numberOfEmployee，雇员的数量，显然这个fields不属于任何一个Employee()实例，而属于类本身的。

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/e344632afcbf80f3f68df860aad44eba.PNG)


联想：静态Field还是很有用的，当我们需要整个类只有唯一一个监视器作为同步锁🔒时，就需要用到static关键字声明唯一的一个Field。[[Java_03_高级特性#同步代码块]]
##### 静态方法
基于类名来调用的方法，用在[[SpringMVC#Result类]]，基于`Result.ok(data)`

注意：静态方法是通过类名直接调用的，不需要实例化对象。因此，==静态方法只能访问类级别的变量和方法，不能获取到实例变量（成员变量）。==

*从类加载过程来看，静态变量在Initialization阶段就已经完成赋值，也就是在创建当前类.class这个对象的时候就已经确定下来类，当然不能读取到之后才创建的成员变量啦*

注意1：即使在类中，在静态方法的代码块中，也无法调用instance member的方法。
```
public static void printNumberOfEmployees() {  
    System.out.println(numberOfEmployees);  
    calculateWage // 这里会报错!因为calculateWage是instance member，不是static member

	// 要是用instance member的方法，就必须先实例化
    new Employee(50_000,20).calculateWage();
}  
  
public int calculateWage(int extraHours) {
    return (baseSalary + extraHours * hourlyRate);  
}
```

注意2：**==所有添加到class Main里的方法，都必须声明为static，这样直接通过类去调用，不用实例化！==**
```
System.out.println()
Interger.parseInt()    // 这些都是直接通过类调用的静态方法！
```

Q：**==Static静态成员，有什么好处？==**
1、代码更加简洁和直观。不需要先创建对象实例，可以直接使用类名调用静态方法，减少了不必要的代码。  
2、静态方法在类加载时就已经存在的，不依赖于对象的创建，所以执行静态方法的速度通常比实例方法要快。这对于一些在应用程序中频繁调用的公共方法非常有用。   
3、不能访问对象状态：静态方法只能访问静态成员变量和其他静态方法，它们不能直接访问实例变量和实例方法。因此，静态方法适用于那些不需要访问对象状态的操作，仅依赖于输入参数的情况。  
4、静态方法常常用于实用程序类和工具类中，提供一些公用的功能方法。  
比如java.lang.Math类中的静态方法提供了一系列的数学计算，而不需要创建Math对象。

#### 原则3：继承性
Inheritance：在一个类中定义Common Behaviors，就可以重用这些行为。

我的总结：上模糊，方法少； 下清晰，方法多
子总是有父所有的技能，也有自己独有的技能。
所以子可以隐式扮演父（Upcasting），但是父不能隐式扮演子（Downcasting） ^a8d5f1

注意：
@mosh提醒：不要创建深度继承层次结构。最多2次继承！

python中有多重继承的特效（C可以继承自A和B），但是Java没有实现这个功能（只能1对1继承）
![[#^ba85df]]


![](https://raw.githubusercontent.com/rockyshen/blog-img/master/86c4a02dbb08a74cf1ac937c2b8e5038.PNG)

##### extends 关键字
表示从基类（Base/Super/Parent Class）中继承！
```
public class TextBox extends UIControl {}   //表示TextBox类继承UIControl类
```

所有的类都继承自==**Object类**==。

注意：如果基类中，我们重写了构造函数需要传递参数 ，在子类实例化的时候需要首先通过super(参数)给基类传递构造函数所需的参数，否则编译报错。
例子1：
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/%E6%88%AA%E5%B1%8F2024-01-28%2014.42.47.png)

例子2:
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/%E6%88%AA%E5%B1%8F2024-01-22%2023.47.51.png) ^6d67a1

**==抽象类，必须被继承，且@override其中的方法；**
**接口，必须被实现，也就是@override其中的方法；==** ^ccbc10

##### super
B类继承自A类，在B类中使用爸爸的方法，就用super来代替
```
public class LotteryServiceImpl2 extends LotteryServiceImpl1{
	public LotteryResult lottery(Long userID, String userName, String ip){
		……
		RLock lock = super.redissonClient.getLock(lockKey);  // 调用v1版（父亲）的锁
	}
}
```

#### 方法重写
方法覆写是指，当从子类继承基类时，基类的方法不符合我们的要求，对方法进行重写。
联想到：
[[Django-2#【难点】==视图集-方法重写==]]   例如重写def get_queryset方法等；
[[Django-2#【难点】==（反）序列化器-方法重写==]]

例如Object类自带的toString()方法，输出的是：`me.shen1993.TextBox@5ca881b5`这样的。我们需要重写为我们需要的样子，需要重写toString()方法

##### @Override
注解 annotation，触发编译器的检查机制：检查重写的方法传递的参数和之前的是不是一致。不一致的话，不给你通过编译。
	*既然你要重写我，传递的参数必须得一样才行吧！

Q：Java的注解（@Override）和Python中的[[函数#装饰器 Decorator]] ，虽然长得一样，但是用法有
A：区别为：
1、Java注解，用于标识这个方法是重写的，Java编译器触发检查机制
2、python装饰器，是更通用的机制，增强函数的功能。[[函数#装饰器的案例]]

除了在方法重写中用@Override进行检查，在interface的实现时，也需要在具体类中利用@Override进行检查。![[#^1a2669]]
```
重写Object类中的toString()方法

@Override  
public String toString() {  
    return text;  
}
```

比较对象的方法重写：一般equals方法和hashCode方法，一起重写
![[标准包.类#^22d0ce]]

##### super关键字
加入你在子类中重写了方法，但是你又想调用父类的方法看看，就用super关键字调用父类方法。 ^789894

#### 类型转换
Casting 转型是指讲一个对象从一个类的类型转换为另一个类的类型。

`User user = new Instructor(20);`
例如`Instructor` 是 `User` 的子类，将一个 `Instructor` 对象赋值给一个 `User` 类型的变量 `user`，我们可以将 `Instructor` 对象视为 `User` 类型的对象。

向上转型通常是隐式的，因此在代码中没有显示地使用类型转换运算符。在运行时，虽然 `user` 是 `User` 类型的变量，==**但它实际上引用的是一个 `Instructor` 对象**==，因此可以调用 `Instructor` 类中的方法和属性。

向上转型 Upcasting  -> [[Java_01_基础#隐式转换 Implicit Casting]]
向下转型 Downcasting  -> [[Java_01_基础#显式转换 Explicit Casting]]

**==upcasting和downcasting是类型转换的概念，explict casting和implict casting是具体实现类型转换的方式。==** ^e7e2ba
##### 向上转型 Upcasting
upcasting：Casting an object to one of its super types   
向上转型：子类的实例，在需要的时候，会自动扮演成父类。
![[#^a8d5f1]]

联想到：[[Java_01_基础#隐式转换 Implicit Casting]]

```
class Animal {     // 父类
	public void eat() {
		System.out.println("Animal is eating");
	}
}

class Dog extends Animal {      //继承自Animal的子类
	public void bark() {
		System.out.println("Dog is barking");
	}
}


// Upcasting 向上转型
Animal animal = new Dog();  子类的对象，赋值给父类的引用。多态性！

animal.eat()    // 可以调用父类方法

// animal.bark()    父类无法调用子类的方法
// 除非将父类强制转化为子类（但是需要进行instanceof类型检查）
if (animal instanceof Dog) {
	Dog dog = (Dog)animal;
	dog.bark()
```

`Animal animal = new Dog();`   基于上述结论，animal变量只能调用Animal类里的方法！

注意点：
- 这样是安全的，因为子类的实例具有父类的所有属性和方法；
- 父类不能强制类型转化为子类（因为父类是更Generic的存在，子类更Specialized(更具体)
- 父类无法调用子类的方法！除非将父类强制类型转换为子类（进行instance of类型检查），然后才能调用。也就是[[#向下转型 Downcasting]]
##### 向下转型 Downcasting
downcasting：Casting an object to one of its sub types  
向下转型：父类的实例，强制让它去扮演子类。这个操作需要在编译时进行类型检查！
联想到：[[Java_01_基础#显式转换 Explicit Casting]]
```
class Animal {
	public void eat() {
		System.out.println("Animal is eating");
	}
}

class Dog extends Animal {
	public void bark() {
		System.out.println("Dog is barking");
	}
}

public class Main {
	public static void main(String[] args) {
		Animal animal = new Dog();
		
		// 向上转型 把animal对象，转成Dog类型！
		if (animal instanceof Dog) {    // 必须类型检查，否则报错！
			Dog dog = (Dog)animal;   //DownCasting
			dog.bark();
		}
	}
}
```

^f0bd0b
##### 类型检查 instanceof
判断这个实例对象，是不是属于这个类。

```
object instanceof class
```
其中,object 是需要检查的对象,class 是要检查的类型。
当 object 是指定 class 类型(或其子类型)的实例时,instanceof 操作符返回 true。否则返回 false。


#### 原则4：多态性
Polymorphism。用生活中的例子比喻：
> 孩子：爸爸，我想要只==宠物==！  
> 孩子口中的“宠物”，在这里就具备多态性。可以new成🐶！当然也可以new成🐱！

多态，指的是同一个方法调用，可以在不同对象上产生不同的行为。主要通过继承和override来实现。
方法有多态性，属性没有多态性。

**格式：声明一个父类的类型，new一个子类的对象**
```
Person p1 = new Man();    // 子类的对象Man()，赋给父类的引用p1
```

多态存在的三个必要条件：
1、继承    [[#原则3：继承 Inheritance]]
2、重写    [[#方法重写 Method Override]]
3、父类引用指向子类对象 Parent p = new Child();

多态的运行原理：**编译看左边，运行看右边** 
1、一个对象能执行哪些方法，主要看左边的类型，和右边没关系
2、当使用多态方式调用方法时，首先检查父类中是否有该方法，如果没有，则编译错误；如果有，再去调用子类的同名方法。 ^24ccad

多态的好处及适用场景
当我们只清楚哪个父类，但是具体哪个子类还不清楚，不确定的时候，利用多态性在先声明父类，之后等清楚的时候，再传递一个子类实例就行。
适用场景：在方法的形参中先写父类，之后清楚了再传子类实参！

```
public class AnimalTest {  
    public static void main(String[] args) {  
        AnimalTest test = new AnimalTest();  
        test.adopt(new Dog());  
    }  
  
    public void adopt(Animal animal) {  
        animal.eat();  
        animal.jump();  
    }  
}
```

多态的弊端：
```
Person p1 = new Man();
```
p1具备多态性，只能调用Person类的方法，不能调用Man类方法，但是因为new了，所以内存空间中已经有Man类的方法预载进去了，占空间。

想要用Man类的方法，就要让p1向下转型：
```
Person p1 = new Man();

(Man)p1.earnMoney
```

思考，
上文主要描述了父类与子类继承时发生多态的情况，**接口也能体现多态性**：
![[#^25a213]]

|            |            |                     | 示例代码                                                                                | 使用场景                                                                                                                                                                         |
| ---------- | ---------- | ------------------- | ----------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **类的多态性**  | extends    | 父类引用指向子类实例<br>体现多态性 | // 子类对象Man，赋给父类引用p1<br>Animal animal = new Dog();                                   | 当我们使用`Animal myDog = new Dog();`时，可以通过`myDog`引用调用`Animal`类中的方法，同时也可以调用`Dog`类中重写或者新增的方法。<br>这样做的好处在于，我们可以在不改变引用类型的情况下，方便地替换为其他子类的对象。<br><br>==思考：这块在现实开发中使用的较少，等遇到了一定要回来补充！== |
| **接口的多态性** | implements | 接口与其实现类<br>体现多态性    | // PC和Camera是USB接口的多态体现<br>public Pc implements USB<br>public Camera implements USB | 业务代码中调用接口，下次想要换实现类的时候，业务代码不受影响。<br><br>面向接口编程，这块在实际代码中我感受过好几次，比较能理解！                                                                                                         |

#### 抽象类/方法 abstract关键字
##### 抽象类
抽象类只能作为其他类的父类被继承，无法实例化。==牛逼啊：只当爹，不干活！==
	联想到Python中的[[类#基类]]，但是python中的基类是可以实例化的吧？

```java
abstract class Animal {        // 定义一个叫Animal的抽象类，包含一个会发声音的抽象方法
    public abstract void makeSound();
}

// 常规类继承抽象类，自动具备抽象类中的方法！
class Dog extends Animal {            // extends应该是继承的意思吧？？
    @Override
    public void makeSound() {
        System.out.println("汪汪汪！");
    }
}

public class Main {
    public static void main(String[] args) {
        Dog dog = new Dog();
        dog.makeSound(); // 输出：汪汪汪！
    }
}
```

思考：抽象类 和 接口的区别？
[[#接口与抽象类的区别]]

![[#^ccbc10]]

##### 抽象方法
抽象方法是没有任何实现的方法，具体方法由子类实现。

接口中声明的默认都是public abstract。
```
interface Flyable {
	void fly();           // 默认就是public abstract抽象方法，不包含方法体
}
```
#### final关键字
final class：声明了final关键字的类，就不能被其他类继承了，它已经到终点了，不能往下继承了！String类就是final，不能被extends 继承了！
final method：声明了final关键字的方法，就不能被@Override重写了！

### 接口 - interface 关键字 
**==接口 就是契约，就是规范！！==**
	理解：接口定义抽象方法（例如3个方法，形成约定），所有接在接口上的类，都知道自己只有这三个方法可以调！ ^c0b956

![[JDBC#^6eac4b]]

在[[#原则2： 抽象]]章节中利用抽象的原则（隐藏不必要的实现细节，只对外暴露必须的方法和字段），

“多态+接口”可以使类与类实现松散耦合度！

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/f60f24d2f7b4cf179956eb254c99627d.PNG)

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/IMG_1063.jpg)

接口都是描述功能性的词！所以接口的命名规则：XXXable（例如Comparable、Iterable等）

| Interface | Class |
| ---- | ---- |
| What should be done       Interface                         定义提供什么功能，HOW | How it should be done      Class                              定义具体如何实现这个功能， WHAT |

==对interface的依赖，要远远好于对其他class的依赖，因为interface中没有具体的功能实现！==

#### 接口特性
1、interface声明的方法，==**默认是public abstract；**==
	没有static method，没有private method，~~没有Field~~。
	结果在Java 8和9中把这些都进进来了，但是mosh认为是糟糕的实现。 ^4d21f6

2、mosh说接口中不要有Field！
	尚硅谷说：interface可以声明Field，但是**默认是全局常量：public static final**。

3、接口没有具体函数功能的实现，在implements的具体功能函数中写具体逻辑。
	正因为没有具体函数实现，才使得它与类之间的耦合更松散！
	联想：有点类似于多态中的父类中，也只需要声明与子类的同名方法，以实现多态化）
	接口的多态性：接口名 变量名 = new 实现类对象；

4、interface无法被实例化

5、接口具有多态性！
直观理解，你看USB接口规范，有各种形态的实现类（键盘、鼠标），那不就是多态么！
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B42%E6%9C%8829%E6%97%A5%2016_51_33.jpg) ^25a213

最佳实践：
[[Spring FrameWork#tx 声明式事务]]  --> 事务管理器作为接口提供“开启事务、提交事务、事务回滚”的方法，由不同的持久层框架提供具体实现。
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/df9234ff62b254359a48ca4545f26022.PNG)


6、==接口可以作为一个引用类型来声明和使用==，可以被用作成员变量的类型。（2024-2-5新增，mosh没讲）

#### 接口原则
interface应该尽可能小，不然interface的变更概率变大，会导致接在上面的类崩坏！
interface拆分成尽可能小的功能点，然后用一个大的interface继承extends

**注意：类不能继承自多个父类，但是接口interface可以继承自多个父类！** ^ba85df

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/0b5755a6ac65edd6a2f605bf2f984eda.PNG)


#### implements 关键字
![[#^089d5a]]

你想要接入这个接口，你就去实现接口约定的规范，实现了，我就认为你接入了接口！
接口中的方法是不能在接口中实现的，只能由实现接口的类来实现接口中的方法。

声明接口中定义的这个方法（用implements关键字），需要在具体类中实现（类下面必须是同名方法，用@Override注释）。（联想：继承的extends关键字，区别是is-a 和 has-a）

疑问：接口中定义了多个方法，到具体类中实现，是要分开？还是合并在一个类中呢
回答：实现这个接口的话，都要实现接口中定义的所有方法！

接口说：每一个想要实现我的具体类，应该是public，且与我声明的方法名字完全一致！（利用@Override进行检查。） ^1a2669

#### 接口使用
① 传递接口类型作为参数    `void doSomething(YourInterface yourinterface)`
② 接口类型作为返回值   `YourInterface getSomething()`
③ 接口类型的引用    `YourInterface yourInterface = new YourImplementation()`

##### 情况1：直接在实现类的对象上调用方法
定义接口和实现类，然后直接在使用场景中实例化实现类的对象，并调用它的方法。
```
// 接口中
interface Consumer<T> {                       
	public void accept(T t);
}

// 实现类中
public class Consumer2024 implements Consumer<String>{         
	// Field
	@override
	public void accept(String s) {
		System.out.println(s);
	}
}

// 直接使用实现类的对象con1，调用其方法
public class ConsumerTest {
	public static void main(String[] args) {
		Consumer2024 con1 = new Consumer2024();
		con1.accept("hello");
	}
}
```


思考：什么时候用接口声明变量，什么时候用接口实现类声明变量？
```
//用接口声明变量类型，List
List<Map> list = new ArrayList<>();
```

```
// 用实现类声明变量类型，否则只能调用DataSource的接口方法，不能调用实现类的方法
DruidDataSource dataSource = new DruidDataSource();
```

声明变量类型时，还是能具体尽量具体实现类；接值时，能用接口尽量用接口（给自己留后路）
```
//DruidPooledConnection connection = dataSource.getConnection();  
Connection connection = dataSource.getConnection();
```
在Java中，接口声明的变量只是一个引用，指向实际对象的内存地址。即使我们使用接口Connection声明变量，但实际调用的是DruidDataSource类的getConnection()方法，该方法返回的是DruidDataSource的实例。
因此，尽管我们使用接口Connection声明变量，connection对象仍然是DruidDataSource类的实例，而不是接口的对象。这种情况下，connection对象具有DruidDataSource类所定义的方法和属性。

**==总结，声明变量用具体实现类，变量接值用接口==**
##### 情况2:额外定义一个“接收接口类型的类”
定义了接口和实现类，==还要定义一个“接收接口类型的类”==（public class Computer）。然后在使用场景中，实例化实现类的对象，并将其作为参数传递给接收接口类型的类的实例(public void transferData(USB usb))。
这种情况下，接收接口类型的类可以对传入的对象进行统一处理，而不需要依赖具体的实现类。

```
// 接口中
public interface USB {
    //抽象方法
    public abstract void start();
    void end();
}

//实现类
public class Painter implements USB{
    @Override
    public void start() {
        System.out.println("打印机开始工作");
    }
    @Override
    public void end() {
        System.out.println("打印机结束工作");
    }
}

//接在接口上
public class Computer {
    public void transferData(USB usb) {
        System.out.println("设备连接成功...");
        usb.start();
        System.out.println("传输数据中...");
        System.out.println("传输完成");
        usb.end();
    }
}
// 上面是构造一种场景：一个USB设备，接到Computer上的场景


// 具体使用场景
// 实例化一个USB设备也即打印机实例，然后传递给Computer对象。
public class Main {
    public static void main(String[] args) {
        Computer computer1 = new Computer();
        Painter painter1 = new Painter();
        computer1.transferData(painter1);    }     // 依赖注入
}
```

^9687fd

```
\\ 接口中
interface VideoEncoder {
	void encoder();
}

interface 
```

情况2的三步骤（如图）
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/093c0dba164a9b13c730daa642d51f1b.PNG)

#### 依赖注入
将“~~实现了接口的类的实例化对象~~” 声明为接口类型的变量传入场景类的方法的==构造函数参数中、方法参数中、属性中==。

思考：将接口类型

我们不应该在类中实例化对象，应该在接口中去做这件事。
	实例化对象，和使用这个对象是两码事！

##### 构造函数传参进行依赖注入
最常用
联想到Spring FrameWork的Ioc容器中：构造器传参进行依赖注入的场景
```
public class UserDao {}

public class UserService {  
    // Field  
    private UserDao userDao;  
    private int age;  
    private String name;  
  
    //构造器传参
    public UserService(int age, String name, UserDao userDao){  
        this.age = age;  
        this.name = name;  
        this.userDao = userDao;  
    }  
}
```
![[Spring FrameWork#^7517f2]]

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/82a1e796df47a5ddc3b30fc4c7d6d692.PNG)

##### Setter函数进行依赖注入
更灵活，可以在主函数生命周期中随意更换，调用setter方法即可
联想到Spring FrameWork的Ioc容器中：setter()传参进行依赖注入的场景
```
public class MovieFinder{}

public class SimpleMovieLister{  
    // Field  
    private MovieFinder movieFinder;  
    private String movieName;  
      
    //setter方法  
  
    public void setMovieFinder(MovieFinder movieFinder) {  
        this.movieFinder = movieFinder;  
    }  
  
    public void setMovieName(String movieName) {  
        this.movieName = movieName;  
    }  
}
```


![](https://raw.githubusercontent.com/rockyshen/blog-img/master/00f401fbdc36f82975e2dff672f06ff3.PNG)

3、Method功能方法注入

#### 接口的新（糟糕的）特性
Java8中，针对接口，引入了：静态方法和默认方法！Mosh认为是坏的实践（他的理由是：接口不应该有具体的代码，接口应该体验规约的作用！），警惕使用
- Java8引入，在接口中，可以定义Field，作为公共常量。  public static final
- Java8引入，在接口中，定义Static Method。（接口不应该有具体代码实现！坏的实践）
- Java8引入，在接口中，定义default method。（表面接口中这个default方法，不是必须实现的，不实现就用默认的）联想到[[Spring FrameWork#【高级特性】FactoryBean接口]]中的`isSingleton()`，默认就是true，因为ioc容器中的Bean就是单例的

Q：如果我们想：跨所有实现此接口的类，重用这个接口中的方法，应该怎么办呢？
补图：
A：写一个抽象类去实现接口，在抽象类中写具体逻辑（protected权限，确保其他包不可见，被继承的子类可以用），子类继承抽象类，就能获得接口中共用的方法了！**但mosh认为应该尽量避免，并不是一个好的习惯，多重继承带来复杂性。**
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/50ab1f42d442035da73edb4b4e2ac36e.PNG)

- 给接口实现私有方法（implement private method in interface）
	![[#^4d21f6]]

#### 接口与抽象类的区别
| 抽象类 - Abstract Class             | 接口 - Interface                                                             |
| -------------------------------- | -------------------------------------------------------------------------- |
| partially-completed class 部分共用代码 | contract 合同，协议                                                             |
| 抽象类的目的是：共享代码片段<br>To share code  | 接口的目的是：构建松散的关系<br>To build loosely-coupled extensible testable application |
| 类内部是有功能实现的逻辑代码的，提供给继承子类共用        | 接口内部完全没有逻辑代码！                                                              |
| 可以包含抽象和非抽象方法                     | 只包含抽象方法，不能具体逻辑代码                                                           |
| 可以有构造函数和成员变量                     | 不能有构造方法，，也不能有成员变量                                                          |
| 可以有不同权限修饰符                       | 接口默认是public                                                                |
| 一个子类，只能继承一个抽象类                   | 一个类，可以实现多个接口                                                               |

接口与继承的区别
接口定义一组规则，体现“如果你要……，则必须能……”的思想
继承是一个“是不是”的关系，即“is-a"关系；接口则是”能不能“的关系，"has-a"关系。 ^089d5a

常见官方接口
Comparable    可比较
Cloneable         可复制

#### 接口匿名实现类的匿名对象
1、创建“接口实现类”的对象
```
Computer computer = new Computer();
Printer printer = new Printer();    // Printer是接口实现类

computer.transferData(printer);
```

2、创建“接口实现类”的匿名对象
```
Computer computer = new Computer();

computer.transferData(new Printer());   // 不给接口实现类Printer实例一个名字，直接传进去
```


3、创建“接口匿名实现类”的对象
```
USB usb1 = new USB() {      // 类是匿名的，就用USB类充当一下，接口不是不能被实例化吗
	public void start() {}  // 临时实现一下抽象方法
	public void stop() {}
};

computer.transferData(usb1)
```

Q：`USB usb1 = new USB() {};` USB作为接口，不是不能被实例化吗？这里如何理解？
A：你注意看！`new USB() { }`后面带了花括号，表示创建一个匿名实现类，该匿名实现类实现了USB接口，并提供了具体实现逻辑。在花括号中，你可以提供该接口的所有抽象方法的具体实现。
尽管接口本身不能被实例化，但是可以通过创建一个实现类接口的匿名类对象来使用接口。因为匿名实现类在创建对象的同时为该对象提供了具体的实现。

注意：在接口的匿名实现类中，不需要用@override，直接实现逻辑即可！

**4、【最终形态】创建“接口==匿名实现类==”的==匿名对象==**
使用场景：当我们调用一个方法，方法的参数是一个接口，就使用这种方式去写。
在参数中，直接现场去实现接口的方法，而不是新开一个class，到那边去写。
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B42%E6%9C%886%E6%97%A5%2015_05_03.PNG)
```
computer.transferData(new USB() {   // 相当于现造一个，也不用赋值这么麻烦了。
	public void start() {}
	public void stop() {}
})

```

Q：Java SPI机制是什么
A：Java Service Provider Interface，定义一个接口规范，然后提供多个实现类，动态查找和加载这些实现类，实现模块化和可拓展性。

### 注解 Annotation
JDK 5.0引入的概念，与类、接口、枚举在同一个层次。
可以声明在包、类、字段、方法、局部变量、方法参数的前面

注解就是获取参数，利用反射传递到框架内部，提供功能。

#### 元注解
对自定义注解进行修饰的注解，就叫元注解。
##### @Retention(RetentionPolicy.枚举)
定义该注解的生命周期（有效范围）

| XX                      | XX                             |                                     |
| ----------------------- | ------------------------------ | ----------------------------------- |
| RetentionPolicy.SOURCE  | 该注解只存在于源代码中，编译生成字节码文件时，就不存在了   | @Override;<br>@SuppressWarnings<br> |
| RetentionPolicy.CLASS   | 默认值，注解存在于源代码和字节码中，运行时内存中就没有了   |                                     |
| RetentionPolicy.RUNTIME | 注解存在于源代码、字节码、运行时内存中，程序通过反射获取注解 | ==自定义注解都用这个==                       |

##### @Target(ElementType.枚举)
指定此注解用在哪个位置，如果不指定默认是是任何地方都能用。

| XX                         | XX      |
| -------------------------- | ------- |
| ElementType.TYPE           | 用在类和接口上 |
| ElementType.FIELD          | 用在成员变量上 |
| ElementType.METHOD         | 用在方法上   |
| ElementType.PARAMETER      | 用在参数上   |
| ElementType.CONSTRUCTOR    | 用在构造方法上 |
| ElementType.LOCAL_VARUABLE | 用在局部变量上 |

注解，里面不写成员变量，不写逻辑方法，仅仅做一个标注，当发现这个注解时，
#### 注解属性
注解，可以是空的；不一定要注解的属性。
属性的作用，让用户在使用注解时传递参数，让注解的功能更强大

属性适用的数据类型
8个基元数据类型[[Java_01_基础#数据类型]] 、 String类型、Class类型 、枚举类型 、注解类型

属性的格式：
```
数据类型 属性名();
数据类型 属性名() default 默认值;

// 示例
public @interface Student {
	String name;
	int age() default 18;
	String gender() default "男"；
}
```

如果不加属性，注解格式就是这样： @MyAnnotation
如果加了属性，注解格式就是这样：@MyAnnotation(属性 = “xxx”);

#### 自定义注解
```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MyAnnotation {
	String descript();         // 数据类型 属性()
}
```

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/IMG_2186.PNG)

注解类  -->  加上注解的类，写具体业务逻辑   -->  框架类，利用反射读取注解信息并执行

##### 自定义字段注解
校验字段是否满足要求，例如校验传递给gender字段，必须是“female"和”male“
```
@Target({ElementType.FIELD})         //作用在字段上
@Retention(RetentionPolicy.RUNTIME)
public @interface Check(){
	
}
```

##### 自定义方法/类注解
权限校验，加到Controller类或方法上，只有通过权限校验的人，才能访问这个方法！否则打回去！

```
@Target({ElementType.METHOD, ElementType.TYPE})    // 作用在类和方法
@Retention(RetentionPolicy.RUNTIME)
public @interface PermissionCheck(){
	
}
```


待学习：自定义注解 结合 AOP实现功能！




************************
[[Java_01_基础]]
[[Java_03_高级特性]]
