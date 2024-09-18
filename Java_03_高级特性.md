进入Java Advanced之后，难度陡然提升，很多概念比较抽象，学到卡壳时，可以回去@尚硅谷对应课程里回炉重造一下。
Mosh在Java课程上，尤其在高级篇章中，讲的不好，太快了，很多细节没讲清楚就强行快速推进下去，这种不熟悉，越积累越多，直到Stream中，寸步难行！暂停，转向尚硅谷！

Java Advanced高级特性，是基于基础知识、面向对象的基础上进行。

大师经验：java 看异常不是看最上面的，是要看最后一个 `Caused by:` 后面的异常，这个是根本原因。

## 异常处理 Exceptions
Exceptions是一个类，包含错误的信息，每个执行时的报错都是类的实例化对象。位于Java.lang模块下。
程序调用时的异常，按栈的数据结构，逐层显示。

### 异常类型
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/8e6efec10b61088c7c81aebe10191725.PNG)
#### 错误 Error
JVM虚拟机自身的问题：例如栈溢出、内存溢出等错误。例如JVM用光了所有内存资源时，就会报Error！
[java.lang.Object.Throwable API说明书，包含各种Exception](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Throwable.html)

Stackoverflow    调用栈的空间移除，存在无限递归
OutMemoryError    JVM的堆内存空间用尽！

#### 编译时异常
- Checked Exception
一般IDEA会自动标红的错误。

ClassNotFoundException
FileNotFoundException
IOException

#### 运行时异常 
- Unchecked / Run Time Exceptions
会通过IDEA的检查，但是在编译完成后，运行时报错。例如需要一个string，你给它传递一个null。就会抛出NullPointerException

| 常见RunTimeException        | 描述                                         |
| ------------------------- | ------------------------------------------ |
| NullpointerException      | 传递非法的Null参数，                               |
| ArithmeticException       | 算术异常                                       |
| IllegalArgumentException  | 传递给method的参数不被接收                           |
| IndexOutOfBoundsException | 索引越界异常                                     |
| IllegalStatesException    | 当我们调用一个方法，但是处于不正确状态==？？不清楚啥意思==            |
| InterruptException        | 被阻塞的线程，调用interrupt()时！<br>[[标准包.类#线程中断机制]] |

### 捕获异常
联想到python中的[[标准库.内置函数#try]]
```
public class ExceptionsDemo {  
    public static void show() {  
        try {  
            var reader = new FileReader("file.txt");  
            System.out.println("File Open");  
        }
        catch (FileNotFoundException ex) {      // 将这个异常捕获，赋值给ex变量
            System.out.println("File do not exit");  // 将报错替换为打印字段,更友好
        }  
    }  
}
```

ex.getMessage()   返回报错信息
小技巧：IDEA自动包裹try

![[Spring FrameWork#^7052d2]]

多重捕获
注意：顺序很重要，如果父级Exception类在前，子级Exception就不需要了。（多态！）
```
try {
	// 会产生异常的代码
}
catch (FileNotFoundException e) {       // 捕获1
	e.printStackTrace();
}
catch (IOException e) {                 // 捕获2
	System.out.println("Could not read data.")
}
catch (IOException | ParseException e) {   // 组合多个
	// 自定义异常反馈
}
```

finally 字段会始终执行，一般用于：释放外部资源，例如：文件关闭、关闭数据库连接、关闭网络连接等。
try代码块内的通过了，之后执行finally代码块；
try代码块内的没通过，执行catch内的捕获异常，最后一定会执行finally。

### try-with-resources
最佳方案，处理外部资源的打开和释放。联想到python中的with[[标准库.内置函数#with语句]]
```
public static void show() {  
    try (var reader = new FileReader("file.txt")) {   // 自动添加final代码块
        var value = reader.read();  
        reader.close();  
    }  
    catch (IOException e) {  
        throw new RuntimeException(e);  
    }  
}
```

### 抛出异常
throwing exception  防御性编程
```
public class Account {  
    public void deposit(float value) {  
        // 自定义抛出的异常，当value<=0时，抛出异常  
        if (value <= 0)  
            throw new IllegalArgumentException();  
    }  
}
```

- **throws** 关键字用于在类的方法声明中指定该方法**可能抛出的异常**。在实例化时，用IDEA自动包裹try catch字段时，会自动使用throws关键字声明的异常类型。

re-throw 相当于从功能函数中throw给main，main接住，并执行其他的功能，不要卡主了！
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/2249d72958d419b63c8c70d30676486f.PNG)

异常抛出到方法签名处，编译器强制要求对异常进行处理；
异常抛到方法内部，编译器不强制要求，由程序员自行处理异常！

### 手写一个异常
// Checked -extends-> Exception     如果你想在编译前就IDEA显红，就继承Exception
// Unchecker -extends-> RuntimeException    如果想在运行时报错，就继承Runtime


例如取款，当取的钱大于余额，需要抛出我们自定义的异常：**钱不够异常**

**步骤：**
1、手写一个异常类（继承自Exception类），其中写发送异常时需要打印的信息就行了。具体什么情况下抛出异常的逻辑，在领域类中声明。
2、在领域类中写什么情况发生时，抛出这个手写的异常类，抛出类的实例化对象new InsufficientFundsException()
3、在main中try…catch接住这个异常对象，赋值给e，然后调用e.getMessage()取打印信息。
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/e36f883d73893f164fe10f01a9f5bde4.PNG)

抛出链式异常：大概知道怎么回事，如果用的多再回过来深入了解。
cause 当前异常由哪个异常引起

## 泛型 Generic `<T>`
实现参数化类型的机制，允许在编译时将类型作为参数传递给类、接口或方法，从而实现代码更通用。

泛型：只能传递引用数据类型（包装类 Integer），不能传递基本数据类型（int）。 ^e963da

例如：我写了一个List类，使用了泛型，当我传递一个String类型数据时，这个实例化的对象List就全部存String。当我传递一个integer类型数据时，List里就都是存整数的。

![[#^797406]]

![[Java_01_基础#^f06487]]

### 泛型类型参数
一种符号表示，相当于一种占位符
**泛型即：类型参数**          ==给int得int，给string的string==

T 一般表示任意的泛型类型，Type/Template，更通用，更抽象
E 一般表示集合中的元素类型，Element
K一般表示键，key
V一般表示值,value
N一般表示Number
?  一般表示不确定的java类型，通配符

static `CompletableFuture<Void>` runAsync(Runnable runnable)
![[JUC#^bc52e1]]

**==声明泛型类时：`public class Result<T>`==**
==**声明泛型方法：`protected static <T> Result<T> build(T data) { }`**==

T类型的Result类型

### 泛型类
泛型类就是针对不同数据类型，有一一对应的匹配，而不是乱的。

```
创建一个List泛型的示例

public class GenericList<T> {
// T表示template，表示一一对应关系。每个想要储存在此List中，必须要指定数据类型，这样便于泛型编译时检查！
	//Field
    private T[] items = (T[])new Object[10];  
    private int count;  

	// 功能方法
    public void add(T item) {  
        items[count++] = item;  
    }  
    public T get(int index) {  
        return items[index];  
    }  
}
```

泛型类的类型推断 ^d9fd88
```
HashMap<String, Integer> map = new HashMap<String, Integer>();

//new后面的字段，简化掉，可以不用重复写HashMap的数据类型，这就是类型推断

HashMap<String, Integer> map = new HashMap<>();
```

#### 泛型中使用基元类型（包装类）
在泛型中使用基元类型，需要使用对应的包装类。[[Java_01_基础#包装类]]
// int  ->  Interger
// float  -> Float
// boolean   -> Boolean

Boxing and Unboxing！
```
GenericList<Interger> numbers = new GenericList<>();

numbers.add(1);    // 1是int，会自动转为Interger类的实例，类似装进一个盒子里，Boxing
int number = numbers.get(0)   // 从引用类型的盒子里把数据取出来，叫Unboxing
```

#### 约束
本质就是通过extends继承，来对泛型进行约束！

例如我们只想实现包含Interger和Float的泛型，其他的例如String不包含，就需要约束！
默认是Object类
```
public class GenericList<T extends Number> {}    // 表示约束为数字
```

两个约束字段用 & 连接  `GenericList<T extends Comparable & Cloneable>`

`Generic<T extends Comparable>`   约束为可比较对象。我们手写的User类就不行了，编译不通过了。除非去User类的声明处加上 `implements Comparable`才行！

#### 类型擦除
泛型的示例代码
编译阶段会进行类型检测，一旦通过了编译，编译器就会把泛型擦除掉。  
到了运行阶段，对于JVM来说，就没有泛型类型的对象，所有类型都是Object类！所以返回true
```
List<Integer> integers = new ArrayList<>();
List<String> strings = new ArrayList<>();

println(integers.getClass() == strings.getClass());   //true
```

目的：向低版本兼容！

Java compiler will erase the generic type parameter and replace it with an actual class or interface depending on the constraints约束，如果没有约束，字节码中默认是Object类型
##### 类型擦除弊端
1、泛型只支持引用类型，不支持基本类型，因为泛型最终会被擦除成Object类型，Object类型不能存储基本数据类型，所以泛型只能用引用类型
![CleanShot 2024-07-12 at 12.34.52.png](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-07-12%20at%2012.34.52.png)


2、不能在泛型上用类型检测
```
if (obj instanceOf List<String>){ }   // List<String>被类型擦除后，就是List类型
```

3、泛型不能用在new实例化上
```
T data = new T();    // 编译错误
```

4、泛型不能用在实例化数组上
```
T[] array = new T[2];   // 编译错误
```


![IMG_2214.jpg](https://raw.githubusercontent.com/rockyshen/blog-img/master/IMG_2214.jpg)

![IMG_2215.jpg](https://raw.githubusercontent.com/rockyshen/blog-img/master/IMG_2215.jpg)


注意：在使用泛型时，一定要时刻牢记泛型擦除的特性，规避不必要的故障

泛型编程是非常优秀的编程范式，但是类型擦除让Java开发者承受了许多痛苦，这一切都是Java设计者在1.0版本时的短视造成的！对于缺陷，没必要遮掩，一些烂的设计，就是烂！没有一门语言是完美的！

### 泛型方法
在方法调用时，指定==参数==类型！
我的理解就是，参数类型不确定，用泛型方法！

语法格式
```
public <T> returnType methodName(T parameter) {
	//方法体
}
```

示例
```
public <E> E judgeReturnStatus(ReturnStatus<E> returnStatus) {

<E>  定义泛型类型参数
E    返回值也是这个泛型类型
```


可以在普通类下面写泛型方法，不一定要在泛型类下面写泛型方法。
```
public class Utils {  
// 泛型方法声明放在static后面，返回类型前面
    public static <T extends Comparable<T>> T max(T first, T second) {  
        return (first.compareTo(second) > 0) ? first : second;  
    }  
}
```

##### 多参数
多参数的泛型方法
```
public static <K, V> void print(K key, V value) {  
    System.out.println(key + "=" + value);  
}
```

多参数的泛型类
```
public class KeyValuePair <K, V>{  
    private K key;  
    private V value;  
  
    public KeyValuePair(K key, V value) {  
        this.key = key;  
        this.value = value;  
    }  
}
```

思考一个问题：
泛型方法和方法的Overload有什么区别？

泛型方法，是在初始状态就指定好了什么类型的参数。Overload是在调用方法时根据提供的参数来选择不同方法。
泛型方法是有先见之明，Overload是擦屁股。

ChatGPT给的答案，我还是不理解，先放这里，慢慢再理解
泛型方法用于在方法级别上实现参数类型化，可以适用于不同类型的参数，提供更好的类型安全性和代码通用性。方法的重载则是在同一个类中定义多个同名但参数不同的方法，根据参数的类型和数量来决定使用哪个重载方法。两者的使用场景和目的有所不同。

#### 泛型类的继承
可以用两个关键字：extends，super
- 如果想读这个类，用extends
`(GenericList<? extends User> users)`

- 如果想add给这个类，用super关键字
`(GenericList<? super User> users)`


##### 通配符 ?
父级的`GenercList<User>`的实现，如何应用到子级实例中。
```
public static void printUsers(GenericList<?> users) {    
							// ? 表示不指定，unknown type
  
}
```

有限度的通配
```
public static void printUsers(GenericList<? extends User> users) {    
							// 有限度通配，只能通配User类及其子类
  
}
```

## 集合接口 Collection
[Collection接口文档](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Collection.html#removeIf(java.util.function.Predicate))

java.util.Collection
![[标准包.类#^ac7f96]]

JDK8中引入的[[#流 Stream]]，使得对集合类的操作更加函数式和便捷！

不同数据类型（ArrayList类、LinkedList类、HashSet类等）去实现它！
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/%E6%88%AA%E5%B1%8F2024-01-28%2022.47.00.png)

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B44%E6%9C%885%E6%97%A5%2022_26_51.jpg)

collection是接口，接口是契约。Collection接口通用功能：==**1、迭代功能，2、增删查改**==

==**通用约定：**==
1、大小操作：size()方法获取集合的大小
2、添加元素：通过add()方法向集合中添加元素
3、移除元素：通过remove()方法向集合中移除指定元素
4、清空集合：通过clear()方法清空集合中所有元素
5、判空操作：通过isEmpty()判断集合是否为空
6、包含操作：通过contains()方法判断集合中是否包含指定元素

Q：fail-fast 和 fail-safe两种模式？
A：上述两种模式，主要作用在迭代过程中：
fail-fast，迭代器遍历集合过程中，如果集合结构发生异常，会抛出异常，并终止迭代
fail-safe，迭代器遍历集合过程中，如果集合结构发生异常，，不会抛异常，因为使用的是副本，适合并发场景，但是有内存开销。

补充：所谓集合结构发生变化，是只原本在迭代前应该已经把集合固定下来了，结果在迭代过程中，执行了add等命令，那不是扑街了吗？
```
List<String> list = new ArrayList<>();
list.add("one");
list.add("two");
list.add("three");
Iterator<String> iterator = list.iterator(); 

while (iterator.hasNext()) { 
	System.out.println(iterator.next());
	list.add("four"); // 这会导致 ConcurrentModificationException 
	}
```
### 声明集合
```
Collection<String> collection = new ArrayList<String>();
// 声明一个字符串集合，具有Collection接口功能：1、迭代功能，2、增删查改
// 存放在ArrayList数组中
// 不能new Collection<>()，因为Collection是接口，不能实例化，只能将接口的实现类，也就是ArrayList类进行实例化！

Collection<String> collection = new ArrayList<>();
// ArrayList<>()  声明泛型，留空，让Java编译器进行类型推断
```

`Collection<数据类型>  collection = new 实现类<>( );`

![[标准包.类#^b940bd]]
### Collection接口公用方法
Collection接口通用的方法
#### add(E e)
向集合中添加一个元素
#### size()
返回数组有几个元素
#### isEmpty()
返回布尔值，是否为空集合
#### remove(Object o)
从集合中，删除其中指定元素
#### clear()
清空集合中的所有元素
#### contains(Object o)
当前集合是否包含目标值，返回布尔值
#### stream()
每一个实现了Collection接口的类，都具备创建stream的能力！[[#创建Stream]]
#### iterator()
返回一个迭代器，用于遍历集合中的元素
#### toArray()
List(ArrayList) 转 数组(Array)：使用List自带的toArray()方法
转换为常规数组`[ ]`

可以看到有三个Overload重载
如果不适用泛型，就是Object[ ]对象数组。数组中每个元素只是Object，无法调用String类方法哦。
所以最好使用泛型！

```
collection.toArray();        
```
#### addAll(collection)
将指定集合中的所有元素，都加到当前集合中！

#### removeIf(Predicate<> filter)
基于过滤条件，来删除元素。
可以用在HashMap的`table[index]`，也即单链表中删除指定key的节点！
```
LinkedList<KVNode> list = table[hash(key)];

list.removeIf(node -> node.key == key);
```

#### collection.stream().forEach(Consumer<> action)
[[#`forEach(consumer)`]]
适用范围有限：集合、Stream！
注意：可以直接在集合上调用forEach，也可以作为Stream的终止操作
	函数式接口，lambda表达式的封装！本质上是在`collection.stream().forEach(consumer)	`
在jdk8中，简化成直接在collection上调用forEach也可以的。
在List中的每个元素上执行指定的操作。

补充一个案例使用：
```

```

思考：for循环和forEach的区别？
for 循环是传统的迭代方式，需要手动指定循环条件、循环体和计数器；是单线程的，无法实现并行处理
forEach方法是增强型，代码更间接，是多线程的。适用范围有限：集合、Stream！

联想到[[Java_01_基础#for循环]]

```
public class ForEachExample {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("Apple");
        list.add("Banana");
        list.add("Orange");

        // 使用 forEach 遍历集合并输出每个元素
        list.forEach(element -> {
            System.out.println(element);
        });
    }
}
```
#### Collections类
Collections类是提供给Collection类的实用方法，联想到Arrays类提供的是对数组`[ ]`的各种操作方法。
注意，以下都是静态方法。都是基于Collections类进行调用
##### addAll(collection，element1…)
实现批量将元素添加到目标集合中
```
方法的签名是：public static <T> boolean addAll(Collection<? super T> c, T... elements)

其中参数c代表目标集合，elements代表要添加的元素。
```

```
Collection<String> collection = new ArrayList<>(); 

// 方法1：逐个添加
collection.add("a");
collection.add("b");

// 方法2：借助Collections类提供的addAll方法
Collections.addAll(collection,"a", "b", "c");

Collection<String> other = new ArrayList<>();
other.addAll(collection);        // 注意，这里的addAll不是Collections类下的方法，是ArrayList类下的，不一样！
```
##### sort()


### List接口
继承自Collection接口。

==**List接口的约定**==：有序的集合、允许重复元素、可以根据索引访问元素!

List接口在此基础上增加功能：1、有序性；2、索引定位
```
List<String> list = new ArrayList<>();   // 声明一个字符串集合，具有列表接口功能
```

使用==**静态工厂方法**==  List.of，声明固定长度的List
![[Java_02_面向对象#^23f9f5]]
```
List<String> list = List.of("apple", "banana", "orange");
```

#### List接口公用方法
#####  add(E e) -> @override Collection -> List
###### overload -->> add(index, e)
提供基于index增加值！
不提供index的话，默认在队尾添加
```
List<String> list = new ArrayList<>();  
list.add("a");  
list.add("b");  
list.add(0, "!");      // 此处的add方法，是基于List类的重载，有index的形参
```

Q：list.add() 与 Arrays.asList()有什么区别？
A：Arrays.asList()不能动态变化！
```
List<Long> ids = new ArrayList<>();  
ids.add(1L);  
ids.add(2L);  

List<Long> ids = Arrays.asList(1L,2L,3L);
```

##### remove() -> @override Collection -> List
###### overload -->> remove(int index)
提供基于index，删除元素

##### contains() -> @override Collection -> List

##### iterator() -> @override Collection -> List
返回该集合的迭代器，后续利用hasNext、next搭配while循环，实现遍历输出！

##### toArray() -> @override Collection -> List
List(ArrayList) 转 数组(Array)：使用List自带的toArray()方法 ^851e26
```
// list to array
List<String> list = new ArrayList<String>();
list.add("叶痕秋")；
list.add("的诗情画意");
list.toArray();
```

##### get(index)
返回List中index位置上的元素
##### set(index, e)
给固定位置替换元素
```
list.add(0,"a");    //如果位置上有，往后移一位

list.set(0,"a");    //替换掉位置上的元素
```

##### indexOf(value)
返回List中value第一次出现的index值！找不到就是-1
	lastOf(value)  该元素最后一次出现的index

##### subList(i, j)
返回子List，前闭后开
[[标准包.类#Comparable接口]]
[[标准包.类#Comparator`<T>`接口]]

#### 【经典实现类】ArrayList类
实现了List接口，==按索引，不遵循FIFO==

联想到：[[数据结构-数组]]

![[Java_01_基础#^404672]]

##### 几种创建列表的区别
- 1、【最方便】假如你知道要往列表中放什么元素，可以用Arrays.asList()直接放，但是不可变的固定大小List！
![[标准包.类#^c3031a]] ^9837f2
```
List<String> list = Arrays.asList("1","2","3");
```


2、new ArrayList<>()是不能往里面直接放元素的，需要new完之后，再add
```
ArrayList<String> myArrayList = new ArrayList<>();    //类型推断
myArrayList.add("a");
myArrayList.add("b");
myArrayList.add("c");

myArrayList.size();

输出：size是3；
```

3、基本的数组的方式，直接声明带有初始值的。同1，也是固定数组大小！
```
String[] strList2 = new String[]{"1","2","3"};
```



扩容机制，当add时，每次扩容50%，扩容至原来的1.5倍。
底层原理：计算出需要扩容的容量，然后实例化一个新数组，把旧数组挪过去！

##### 常用方法
[[#List接口公用方法]]
###### add(E e)  -> @override Collection -> List -> ArrayList
###### overload -->> add(index, e)
###### addAll(Collection c) -> @override Collection -> List -> ArrayList
批量集合加到List尾部

注意：和Collections类下的addAll()方法不同！
```
方法的签名是：public boolean addAll(Collection<? extends E> c)，

其中参数c代表要添加的集合。
```
将一个集合的所有元素添加到另一个集合中。
```
other.addAll(collection);   //将collection中所有元素添加到other这个空的ArrayList类实例中
```

###### remove() -> @override Collection -> List -> ArrayList
###### overload -->> remove(int index)

###### iterator() -> @override Collection -> List -> ArrayList

###### get(index) ->  @override List -> ArrayList

###### set(index, E e) ->  @override List -> ArrayList

###### indexOf(E e) ->  @override List -> ArrayList

##### 【子类】【并发】CopyOnWriteArratList
是线程安全的列表，适用于读多写少的场景。
每次写操作都会复制一份新的数组（），写操作比较慢，但是读操作不加锁，具有较高并发性能。

ArrayList线程不安全问题的解决方案！
方案1：[[Java_03_高级特性#*~~【古老实现类】Vector类~~*]]
方案2：[[标准包.类#`synchronizedList(List<T> list)`]]
方案3：`CopyOnWriteArrayList<E>`类
读的时候，支持并发读；
写的时候，独立写；（写时复制）
```
List<String> list = new CopyOnWriteArrayList<>();
```

#### 【实现类】LinkedList类
注意：这是==**双链表**==！
实现了List接口 + Queue接口；（两头都能进出  --> [[#【实现类】ArrayDeque类]]
由于实现了Queue，==遵循FIFO接口==

联想到：[[数据结构-链表]]

##### 常用方法
###### [[#List接口公用方法]]
###### [[#Deque接口公用方法]]
###### poll()
移除并返回链表开头元素，继承自Queue接口的方法
等价于：Deque接口公用方法 -> pollFirst()
###### peek()
返回链表开头元素，但不移除，空链表时返回null，继承自Queue接口的方法
等价于：Deque接口公用方法 -> peekFirst()

思考：[[#【实现类】LinkedList类]] 与  [[#【实现类】ArrayDeque类]]  有什么区别？
回答：ArrayDeque通过ArrayList + 双指针实现，而LinkedList则通过链表实现；
ArrayDeque不支持存储null数据，但是LinkedList支持。
ArrayDeque在JDK1.6引入，LinkedList在JDK1.2时引入。
ArrayDeque插入时可能发生扩容，但是插入操作为O(1)，LinkedList虽然不扩容，但是需要申请新的堆空间，性能慢。
因此，在实现队列时，ArrayDeque更好！

#### *【古老实现类】Vector类*
java.util.Vector
所有的方法，都加上了[[#线程同步机制1： 关键字：synchronized]]，所以可以解决线程安全问题！

Vector与ArrayList的区别？
1、Vector借助synchronized实现线程安全、ArrayList是线程不安全的
2、ArrayList性能优于Vector
3、都能动态扩容，ArrayList每次增加50%，Vector每次增加1倍

源码！
```
public synchronized void addElement(E obj) {  
    modCount++;  
    ensureCapacityHelper(elementCount + 1);  
    elementData[elementCount++] = obj;  
}
```

##### Stack
栈！线程安全的！因为继承自Vector，所有方法都加上了synchronized关键字

#### 循环遍历框架
1、单个列表，遍历每个元素
```
for (int i = 0; i < list.size(); i++){
	list.get(i);
}
```

2、嵌套列表，遍历
```
int i = 0;
int j = 0;
while(j < list.size()){
	for (j = i+ 1; j < list.size(); j++){
		//
	}
	i++;
}
```


******************
### Queue接口
Queue接口在Collection基础上增加：先进先出 (FIFO) 的特性。
这种特性适用于：任务调度、缓冲、消息传递

**==Queue接口的约定==**：1、元素的出队列顺序遵循FIFO 或 优先级排序的规则；

#### Queue接口公用方法
##### boolean add(E e) -> @override Collection -> Queue
将指定元素加入队尾
add与offer的区别是，当队列满时，add会抛异常，offer会返回false
##### boolean offer(E e)
将指定元素加入队尾
add与offer的区别是，当队列满时，add会抛异常，offer会返回false

##### E remove() -> @override Collection -> Queue
移除并返回队列头部元素
remove和poll的区别是，当队列为空时，remove会抛异常，poll会返回null
##### E poll() 
移除并返回队列头部元素
remove和poll的区别是，当队列为空时，remove会抛异常，poll会返回null

##### E element()
返回队列头部元素
element和peak的区别是，当队列为空时，element会抛异常，peak会返回null
##### E peak()
返回队列头部元素
element和peak的区别是，当队列为空时，element会抛异常，peak会返回null

#### 【经典实现类】AbstractQueue
##### 【子类】PriorityQueue类
[PriorityQueue解析](https://pdai.tech/md/java/collection/java-collection-PriorityQueue.html)
最高优先级的优先出队！
底层使用可变长的数组存储数据，利用[[数据结构-堆#二叉堆]]数据结构实现！
默认是[[数据结构-堆#Min Heap 最小堆/小顶堆]]（完全二叉树 = 堆 =某元素一定比其左右子元素小），但可以接收一个Compartor作为构造参数，自定义元素优先级的先后。
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/IMG_2021.PNG)
主要用在手撕算法中，典型例题是：堆排序、第K大的数，带权图的遍历
##### 【子类】【并发】ConcurrentLinkedQueue
线程安全的非阻塞队列，用于多线程并发地插入和删除元素
适用于多线程环境下！

#### 【子接口】BlockingQueue
阻塞队列的接口，下面所有实现类都是线程安全的，因为是阻塞的！
##### BlockingQueue接口公用方法
###### add(E e)  -> @override Collection -> Queue -> BlockingQueue
加入元素，成功返回True，超过容量，抛出IllegalStateException异常
###### offer(e) -> @override Queue -> BlockingQueue
加入元素，成功返回True，超过容量，返回false
###### ==overload -->> offer(e, time, unit)==
==阻塞==超过等待时间，放不进去元素，就结束
###### remove()  -> @override Collection -> Queue -> BlockingQueue
移除元素，没有的话，抛出异常
###### poll() -> @override Queue -> BlockingQueue
移除元素，没有的话，返回false
###### ==overload -->> poll(time, unit) ==
==阻塞==超过等待时间，取不到元素，就结束
###### put(E e)
指定元素插入此队列中，进入阻塞状态
###### take()
获取并移除此队列的头部，取不到的话，进入阻塞状态

| 方法类型 | 抛出异常      | 特殊值      | 阻塞     | 超时                   |
| ---- | --------- | -------- | ------ | -------------------- |
| 插入   | add(e)    | offer(e) | put(e) | offer(e, time, unit) |
| 移除   | remove()  | poll()   | take() | poll(time, unit)     |
| 检查   | element() | peek()   | 不可用    |                      |

##### 【实现类】ArrayBlockingQueue
特点：
1、基于数组Array实现的阻塞队列
2、维护一个==定长的数组==
3、构造函数必须传递一个int,也就是长度！

[[JUC#【实现类】ArrayBlockingQueue]]
##### 【实现类】LinkedBlockingQueue

[[JUC#【实现类】LinkedBlockingQueue]]

#### 【子接口】Deque接口
双端队列，允许在两端进行插入和删除操作的队列。
两头都通，所以可以实现FIFO的队列，也能实现LIFO的栈！双击666！
##### Deque接口公用方法
###### boolean offerFirst(E e)
在队列头部插入一个元素.
addFirst 与 offerFirst的区别：如果队列满了，addFirst抛异常，offerFirst返回false
######  void addFirst(E e)
在队列头部插入一个元素.
addFirst 与 offerFirst的区别：如果队列满了，addFirst抛异常，offerFirst返回false

###### boolean offerLast(E e)
在队列尾部插入一个元素。
addLast 与 offerLast的区别：如果队列满了，addLast抛异常，offerLast返回false
###### void addLast(E e)
在队列尾部插入一个元素。
addLast 与 offerLast的区别：如果队列满了，addLast抛异常，offerLast返回false

###### E pollFirst() 
移除并返回队列的头部元素
removeFirst 与 pollFirst的区别：如果队列为空，removeFirst抛异常，pollFirst返回null
###### E removeFirst()
移除并返回队列的头部元素
removeFirst 与 pollFirst的区别：如果队列为空，removeFirst抛异常，pollFirst返回null

###### E pollLast() 
移除并返回队列的尾部元素
removeLast 与 pollLast的区别：如果队列为空，removeLast抛异常，pollLast返回null
###### E removeLast()
移除并返回队列的尾部元素
removeLast 与 pollLast的区别：如果队列为空，removeLast抛异常，pollLast返回null

###### E peekFirst()
返回队列的头部元素，但不移除
getFirst 与 peekFirst的区别：如果队列为空，getFirst抛异常，peekFirst返回null
######  E getFirst()
返回队列的头部元素，但不移除
getFirst 与 peekFirst的区别：如果队列为空，getFirst抛异常，peekFirst返回null

###### E peekLast()
返回队列的尾部元素，但不移除
getLast 与 peekLast的区别：如果队列为空，getLast抛异常，peekLast返回null
###### E getLast()
返回队列的尾部元素，但不移除
getLast 与 peekLast的区别：如果队列为空，getLast抛异常，peekLast返回null

###### void push(E e)
在队列头部插入一个元素，如果队列满了，会抛出异常，==等价于 addFirst()==
###### E pop()
移除并返回队列头部元素，如果队列为空，会抛出异常，==等价于removeFirst()==

##### 【实现类】ArrayDeque类
双端队列，联想[[标准库.内置函数#deque]]
[[#Deque接口公用方法]]



******************
### Set接口
继承自Collection接口
==**Set接口的约定**==：1、不允许重复元素；2、不保证有序性；

```
collection = ["a", "b", "b", "c"]

Set<String> set = new HashSet<>();    //自动去除重复的

>> 输出： ["a", "b", "c"]
```

#### Set接口公用方法
##### add(E e) -> @override Collection -> Set
向set中添加元素，如果存在，返回false
##### addAll(Collection c) -> @override Collection -> Set
将一个集合中所有元素都添加到set中，只保留不重复的元素。
也就是并集！

##### retainAll(collection) -> @override Collection -> Set
仅保留set中与传入集合相同的元素，移除其他
也就是交集
```
set1.retainAll(set2);
```
##### removeAll(Collection c) -> @override Collection -> Set
移除set中与传入指定集合中相同的元素。
返回第一组有，但是第二组没有的部分
```
set1 = ["a", "b", "c"];
set2 = ["b", "c", "d"];

set1.removeAll(set2);

>>输出：a   set1中有的，set2中没有的
```

#### 【经典实现类】HashSet类
实现了Set接口
底层是HashMap + 保证唯一性；不保证插入和取出顺序
##### 常用方法
###### [[#Set接口公用方法]]

###### removeId(filter)
根据条件删除符合条件的元素，java8引入的，函数式接口

#### 【子类】LinkedHashSet
底层是链表+哈希表，可以保证插入和取出顺序！
[[#【子类】LinkedHashMap]]

#### 【子类】TreeSet
底层是红黑树，元素是有序的，排序方式支持自然排序或自定义排序
[[#【实现类】TreeMap]] ^d2b4f5

#### 【子类】【并发】CopyOnWriteSet
HashSet线程不安全问题的解决方案
```
Set<String> set = new CopyOnWriteArraySet<>();
```

******************
### Map接口
java.util.Map
区别于Collection，存储一对一的数据！

==**Map接口的约定**==：1、键值对存储；2、键的唯一性

底层使用：[[数据结构-哈希表]]实现。

Map是一种常见数据结构，通常用于实现缓存功能。使用键，快速查找到值（O1），非常适合缓存的场景。将需要缓存的数据作为val，缓存数据标识符作为key。  [[redis]]
#### Map接口公用方法
##### put(key, value)
将键值对，写入map

创建一个字典，示例代码：
```
Map<String, Object> employee2 = new HashMap<>();  
employee2.put("name","marry");  
employee2.put("salary",111.33);
```
##### get(key)
返回value
##### remove(key)
删除一整个entry，返回value
##### size()
返回长度
##### keySet()
由于key是唯一的，不能重复的，所以使用Set进行存储！
调用此方法，可以返回包含所有key的Set集，返回Set类型
```
Set keySet = map.keySet();

Iterator iterrator = keySet.iterator();
while (iterator.hasNext()) {
	System.out.println(iterator.next());
}
```

==注意：Map接口没有去实现Iterable接口，所以无法用for循环。可以用keySet来进行循环！==
```
for (var key : map.keySet())
	sout(key)；
```
##### values()
遍历value集，返回的是一个集合
##### entrySet()
一个Entry对象，里面包含一个键值对。
返回Map中所有Entry对象的Set集，返回Set类型
利用Set这一数据类型，来返回 键=值
	entry.getKey()         //基于entry返回key
	entry.getValue()      //基于entry返回value

```
Map map = new HashMap();

Set entrySet = map.entrySet();

for (Object entry：entrySet){
	//将HashMap的Node，强转为Map的Entry，才能调方法
	Object key = (Map.Entry) entry.getKey();
	Object value = (Map.Entry)entry.getValue();
}
```

Entry.getKey()
Entry.getValue()
##### containsKey(key)
查询是否有这个键值对，借助[[标准包.类#hashCode()]] 和 [[标准包.类#equals()]]方法，返回布尔值。
##### clear()
清空当前map中的所有数据
##### replace(key,newValue)
将key位置的value替换掉
##### computeIfAbsent()
from [@SchelleyYuki -> Java教程：如何写出更优雅的Java代码](https://www.youtube.com/watch?v=x4z3K0BsrwM&list=PLnwmMZboPhaiwsrFdm-nGJm9EkMk-WmhI&index=12&t=338s&ab_channel=SchelleyYuki
检查key是否存在，如果不存在，就利用函数式接口创建一个！

复杂Map：指的是value是一个List。这种复杂Map在项目上、算法上大量用到！
![[#^7b4712]]
```
Map<key, List<value>> map = new HashMap<>();
map.get(key).add(val);   //存在一个问题，如果key不存在，就会往null里放数据

 |
 V

if (!map.containsKey(key)) {
	map.put(key, new ArrayList<>());
}
map.get(key).add(val);

 |
 V

Map<key, List<value>> map = new HashMap<>();
map.computeIfAbsent(key, k -> new ArrayList<>());
map.get(key).add(value);

```

#### 【经典实现类】HashMap类
**底层实现
JDK 1.7：数组+单向链表
JDK 1.8：[[数据结构-数组]]+单向[[数据结构-链表]] -> [[数据结构_树#红黑树]]结构存储。**（链表长度大于8时，升级为红黑树） ^c0e653

[[数据结构-哈希表#拉链法]]
##### HashMap的put流程
1、判断数组容量是否足够，如果不够的话，执行resize()进行扩容
2、根据key放入哈希函数计算出对应数组索引：
	情况1:如果该索引没有元素，直接在此索引放入元素，跳转至6
	情况2:如果该索引已经有元素了（证明发生哈希冲突了），跳转至3
3、（修改值的情况）判断索引处的链表头部的key是不是和我们的key一样，如果一样，value直接替换
4、判断索引处元素是否为红黑树节点，如果是红黑树，走红黑树插入键值对流程；如果不是红黑树，证明还是链表，跳至5
5、遍历该链表，如果发现key相同的，更新value，如果不存在就在链表中插入一个节点，并判断链表长度大于8时，转换为红黑树！
6、插入成功后，计算负载因子，如果达到了75%，就扩容1倍。

利用哈希函数计算key的哈希值，并将其转为数组索引，然后将value存在数组该索引处。
万一出现不同key生成相同哈希值，发生哈希冲突时，使用单链表将相同哈希值的值串在一起。 ^40a640

==链表升级为红黑树的规则：链表长度大于8，且数组长度大于64时切换为红黑树==。  ^55a1b5

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B46%E6%9C%883%E6%97%A5%2017_47_38.jpg)

注意，此处输入泛型时，传递两个参数！
```
Map<String, Object> map = new HashMap<>();
```

特性：
1、Key不允许重复；无序。所有key构成一个Set集合。
	---> key所在类，重写[[标准包.类#hashCode()]] 和 [[标准包.类#equals()]]
2、value允许重复，无序；所有value构成一个Collection。
	--> value所在类，重写[[标准包.类#equals()]]
3、一个key-value，就构成了一个entry
4、entry不允许重复，无序。所有entry就构成了一个Set集合。

HashMap中的Entry，叫做Node！
`static class Node<K,V> implements Map.Entry<K,V>`

所以IDEA和lombok都会自动生成：equals和hashCode方法。

1、键值对都可以为null；
2、键值的值传递（深拷贝）
##### 常用方法
###### [[#Map接口公用方法]]

浅拷贝：只拷贝的是栈-局部变量表 <---> 堆内存 引用关系
深拷贝：直接建立与堆内存新的引用关系。

HashMap的特性
1、HashMap可以接受key为null，同时设置Value！因此是线程不安全的！ ^7b4712

key一旦被put进去，就会值传递，堆内存中的对象直接与Map的键关联，而不是和key变量关联了。这种特性就叫值传递。
```
public static void myHashMap() {
	HashMap<Integer, String> map = new HashMap<>();
	Integer key = new Integer(1);
	String value = "HashMap";

	map.put(key,value);
	sout(map);
>> 输出：{1="HashMap"}

	key = null;
	sout(map);
>> 输出：{1="HashMap"}
}
```

Hashtable是线程安全的，性能差，不接受null
**HashMap是线程不安全的**，性能好，可以接收null

在这个例子中，有两个线程分别进行往 HashMap 中添加元素和删除元素的操作。由于 HashMap 不是线程安全的，因此可能会发生以下情况之一：
1. 线程1添加元素的同时，线程2也在删除元素，可能导致数据丢失或者覆盖。
2. 线程1添加元素时，由于没有同步机制，可能导致 HashMap 的内部数据结构发生错误，使得最终 HashMap 的大小不一定是期望的大小。
所以，HashMap是线程不安全的。
```
public class HashMapThreadUnsafeExample {
    static HashMap<Integer, String> map = new HashMap<>();
    public static void main(String[] args) {
	    //实现Runnabl接口的方式，创建一个进程！
        Runnable task1 = () -> {
            for (int i = 0; i < 1000; i++) {
                // 线程1往HashMap中添加元素
                map.put(i, "A");
            }

        };
        
        Runnable task2 = () -> {
            for (int i = 0; i < 1000; i++) {
                // 线程2往HashMap中删除元素
                map.remove(i);
            }
        };

        Thread thread1 = new Thread(task1);
        Thread thread2 = new Thread(task2);
        thread1.start();
        thread2.start();

        try {
            thread1.join();
            thread2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("HashMap size: " + map.size());
    }

}
```

| XX        | XX    |                                                        |                 |
| --------- | ----- | ------------------------------------------------------ | --------------- |
| Hashtable | 线程安全  | 不能使用null作为键，也不允许null作为值                                | 继承自Dictionary类  |
| HashMap   | 线程不安全 | 可以使用null作为键，也可以null作为值<br>注意：ConcurrentHashMap不允许用null | 继承自AbstractMap类 |

注意：HashMap不是线程安全的，多线程环境下同时对 HashMap 进行读写操作，可能会导致数据不一致或者出现竞态条件的情况。

##### 【子类】LinkedHashMap
是HashMap的子类！
新增对插入顺序的维护，保证键值对插入的顺序。

**底层实现： [[数据结构-数组]]+单向[[数据结构-链表]]+[[数据结构_树#红黑树]]  +   双向[[数据结构-链表]]结构存储。**
	单链表：一个bucket中的节点，按单链表添加（同HashMap原理)
	双链表：所有节点按双链表连接，可以维护先后顺序

开发中，对于频繁的遍历操作，建议使用此类！

##### 【子类】WeakHashMap
一般在缓存时使用这个WeakHashMap
##### 【子类】【并发】ConcurrentHashMap
HashMap线程不安全问题的解决方案
```
Map<String,Object> map = new ConcurrentHashMap<>();
```

底层：数组（每个元素是bucket）+ 链表/红黑树
![[#^55a1b5]]

HashMap的演进过程
![[#^c0e653]]

ConcurrentHashMap的演进过程与HashMap的升级保持同步：
**JDK 1.7 ：** 由16个Segment分段数组组成（也就是允许16个线程）+ 单向链表
每个Segment数组中的每个元素叫：HashEntry（是链表结构，解决哈希冲突），Segment继承ReentrantLock。当有线程需要对数据进行修改时，必须获得对应Segment的锁，对同一个Segment的写入会被阻塞，不同Segment的写入时并发执行的。
**JDK 1.8**：~~取消了Segment分段锁的设计~~，同步升级JDK 1.8中HashMap的底层数据结构（数组+链表/红黑树）
并发控制采用synchronized + [[JUC#CAS机制]]。比1.7版本锁的颗粒度更细，synchroinzed加到每个元素的链表首节点上，谁要修改必须持有这个链表头节点的锁，才能对这个元素中的链表进行修改，越精细，发生抢锁的概率越低，效率大大提升

###### ConcurrentHashMap的put过程
#Java八股/JUC 
1、初始化bucket数组
2、如果待插入的元素所在bucket为空，把此元素放入bucket的头节点
3、如果整个bucket数组正在扩容，则把当前线程排在扩容后面
4、待插入的元素所在bucket不为空且没在迁移元素，则锁住这个bucket的头节点
5、如果当前bucket为链表，则在链表中寻找该元素或插入该元素
6、如果当前bucket为红黑树，则在树中寻找该元素或插入该元素
7、如果元素的key存在，覆盖value
8、如果元素不存在，整个Map元素个数+1，检查是否需要扩容。^157e78

思考：ConcurrentHashMap的Key、Value是否可以为Null？HashMap的Key、Value是否可以为null？
ref：[HashMap 可以存 null，ConcurrentHashMap 不可以为什么](https://xie.infoq.cn/article/430746f01ca50357646db3e88)
回答：ConcurrentHashMap不允许kv为null，无法解释到底是存储本身null，还是不存在（二义问题）
HashMap允许kv为null，因为在单线程下使用，可以用[[#containsKey(key)]]来排除二义问题。多线程环境下用containKey排除是不准确的，因为可能被其他线程锁住的地方，你查不到！

#### 【实现类】TreeMap
**底层实现：[[数据结构_树#红黑树]]结构存储。**
可以按照添加键值对的，Key元素指定属性的大小顺序进行遍历。
需要考虑排序方法：自然排序、定制排序。

因为基于Key排序，所有Key必须是同一个类的对象，同类才能比较。

#### *【古老实现类】Hashtable类*
**底层实现：[[数据结构-数组]]+单向[[数据结构-链表]]**
Java 中的一个旧的（初代目）、==线程安全的==哈希表实现，它继承自 Dictionary 类，实现了 Map 接口
1、线程安全的，但是由于它是同步的，因此性能通常不如 HashMap。所有公共方法都使用了 synchronized 关键字进行同步。更常用的是 [[#【实现类】ConcurrentHashMap]]
2、键值都不允许为 null
##### Properties
继承自HashTable
特点：键值对都必须是String类型！
常用来处理配置（属性）文件！

xml、jdbc、JSON都用到了Properties！
注意：key=value，千万不要有空格

Spring微服务启动时加载的配置文件。bootstrap.properties
![[SpringCloud#^249dc0]]

补充一个技巧，通过[[JVM#类的加载器]]，获得指向字节码根路径下指定文件的输入流！ ^766ed9
```
Properties properties = new Properties();

InputStream ips = DruidUsePart.class.getClassLoader().getResourceAsStream("druid.properties");

properties.load(ips);

// properties对象中就可以调用Field来获取值了！
```
#### [[标准包.类#Pair类]]
和Map没有直接关系啦，只不过它是简单的键值配对。
有三个包提供：javafx.util、apache.commons.lang3.tuple
我的IDEA中导入了：apache commons lang3.tuple中的Pair类


### 并发容器
是指能在高并发场景下，安全使用的Collection，在此做一个汇总
[[#【子类】【并发】ConcurrentHashMap]]
[[#【子类】【并发】CopyOnWriteArratList]]
[[#【子类】【并发】ConcurrentLinkedQueue]]
[[#【子类】【并发】CopyOnWriteSet]]


## 函数式接口 -> lambda表达式
Java中，正是因为当接口只有一个抽象方法时，利用lambda表达式（= 接口的“匿名实现类”的对象），此时lambda表达式虽然是函数（包含逻辑），但被看做一个对象，可以被传递，被返回。这就符合[[编程范式#函数式编程]]的特点！

虽然Java是面向对象的语言，但是在lambda表达式（替换接口”匿名实现类“的对象）时，进行了[[编程范式#函数式编程]]的尝试。
![[JavaScript#^b74943]]

### ==_总结函数式接口的应用场景==

| 使用场景               | 方法                                 | 接收参数                           | 接口唯一抽象方法               | 备注                                    |
| ------------------ | ---------------------------------- | ------------------------------ | ---------------------- | ------------------------------------- |
| 流式计算               | stream().filter( ( )->{ } )        | [[#Predicate -> test()]]       | test( )                | (a) -> {一个判断语句 return true or false;} |
|                    | stream().map( a->b )               | [[#Function -> apply()]]       | apply( )               | map( a -> b).collector(toList)        |
|                    | forEach( ( )->{ } )                | [[#Consumer -> accept()]]      | accept( )              | (a) -> {无返回}                          |
| 线程                 | new Thread( ( )->{ } )             | [[#方式二：【常用】实现Runnable接口]]      | run( )                 | Runnable task1 = () -> {线程处理的业务逻辑}    |
|                    | new Thread( ( )->{ } )             | [[#方式3：实现Callable接口]]          | call( )                |                                       |
| 比较                 | compare( )                         | Compartor`<T>`接口               |                        | (o1, o2) -> {自定义比较逻辑}                 |
|                    | compareTo( )                       | Comparable`<T>`接口              | (a,b) -> {return a-b;} | stream.sort(Comparable)               |
| mybatis_plus       | and( )                             | [[#Consumer -> accept()]]      | accept( )              | (a) -> {无返回}<br>无返回，只是构造wrapper条件     |
| 原子操作增强类            | new LongAccumulator( )             | applyAsLong(left , right)      | applyAsLong( )         | 构造一个LongAccumulator必须传递函数式接口          |
| 事务工具类              | transactionTemplate.execute()      | doInTransaction( (status)->{}) | doInTransaction()      | TransactionTemplate工具类实现编程式事务！        |
| Mybatis_Plus中and方法 | queryWrapper.and(w -> {w.eq(...)}) | [[#Consumer -> accept()]]      | accept()               | Mybatis_Plus中and方法接收一个Consumer接口      |

##### stream().map(Function函数式接口)
映射
```
.map( (xx) -> 一顿处理; return xx)
```
示例代码
```
list.stream()
	.map((netMall) -> String.format(productName + "in %s price is %.2f",netMall.getNetMallName(), netMall.calcPrice(productName));)
	.collect(Collectors.toList());
	
// map中是隐式的返回结果，没有return，不能用{}
```

```
memberPrice.stream()
	.map(item -> {
		MemberPriceEntity memberPriceEntity = new MemberPriceEntity();
		memberPriceEntity.setSkuId(skuReductionTo.getSkuId());
		memberPriceEntity.setMemberLevelId(item.getId());
		memberPriceEntity.setMemberLevelName(item.getName());
		memberPriceEntity.setMemberPrice(item.getPrice());
		emberPriceEntity.setAddOther(1);
		return memberPriceEntity;})    // 有return，必须用{ }包裹
	.filter()
```
##### stream().filter(Predicate函数式接口)
```
.filter( (xx) -> 条件 )
```

#### 优化格式
1、无参数，无返回值时，参数列表的`( )`不能省略，不然缺东西了。
2、lambda只需要一个参数时，参数的小括号也可以省略。
3、有参数时，参数列表的引用类型可以省略，因为编译器会类型推断。
4、lambda体只有一条语句，return和{ }可以省略

### 函数式接口
位于：java.util.function这个包中
一个接口中只声明一个抽象方法，就叫函数式接口！
常见的函数式接口，主要就是下面四大类：

| 函数式接口                | 唯一抽象方法      | 返回值类型   | lambda表达式              | 使用场景                                                                   |
| -------------------- | ----------- | ------- | ---------------------- | ---------------------------------------------------------------------- |
| 消费型 `Consumer<T>`    | accept(T t) | void    | (t)->{具体如何消费}          | forEach(Consumer)<br>[[JUC#whenComplete(`BiConsumer<V, E>`)]]          |
| 供给型 `Supplier<T>`    | get()       | T       | () ->{ 供给一个结果}         | [[JUC#supplyAsync(Supplier supplier) 有返回值]]                            |
| 函数型 `Function<T, R>` | apply(T t)  | R       | (t) -> "Result:"+ t +1 | stream.map(Function)                                                   |
| 断言型 `Predicate<T>`   | test(T t)   | boolean | (t) -> t >7000         | 1、stream.filter(Predicate)<br><br>2、File类中：list(FilenameFilter filter) |


#### Consumer -> accept()
**Consumer Interface**
consumer：接收一个参数，返回void。之所以被称为consumer，因为它消耗了一个参数！ ^00e6e4

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/%E6%88%AA%E5%B1%8F2024-02-05%2021.00.42.png)

accept(T t)
接收一个参数，返回void。之所以被称为consumer，因为它消耗了一个参数！

链式调用consumer
```
List<String> list = List.of("a", "b", "c");  
Consumer<String> print = System.out::println;  
Consumer<String> printUpperCase = item -> System.out.println(item.toUpperCase());  
list.forEach(print.andThen(printUpperCase.andThen(print)));
```

consumer的lambda案例
```
Arrays.stream(list).forEach(n -> System.out.println(n));
```

##### `BiConsumer<V, E>`
传递两个输入参数，但是没有返回值！
联想到：[[JUC#whenComplete(`BiConsumer<V, E>`)]]

#### Supplier -> get()
**Supplier Interface**
supplier：不接收任何参数，返回一个值。所以叫Supplier ^e2a31d

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/%E6%88%AA%E5%B1%8F2024-02-05%2021.45.17.png)

**get()**
不接收任何值，但是返回一个值

supplier的lambda案例
```
Stream.generate(() -> Math.random())   不接收参数，返回一个随机数
```

#### Function -> apply()
**Function Interface**
接收一个值，返回一个值。y = 1+x

注意，有的时候，并没有显式的用return，而是直接一行代码会产生返回值，也行的
这时候，没有用return，就不能用{ }

**apply(T t)**
接收一个参数T t，返回结果R r

链式调用Function
```
Function<String, String> replaceColon = str -> str.replace(":","=");  
Function<String, String> addBraces = str -> "{" + str + "}";  

// andThen的链式调用
var result = replaceColon.andThen(addBraces).apply("key:value");

// compose的链式调用，顺序不同
var result = addBraces.compose(replaceColon).apply("key:value");

```

#### Predicate -> test()
**Predicate Interface**
接收一个值，判断这个值是否符合要求，返回布尔
```
@FunctionalInterface
public interface Predicate<T> {
	boolean test(T t);    # 只有一个test抽象方法，输入一个条件，返回True or False
}
```

^f2bb59

**test(T t)**
链式调用Predicate
```
Predicate<String> hasLeftBrace = str -> str.startsWith("{");  
Predicate<String> hasRightBrace = str -> str.endsWith("}");  
Predicate<String> hasLeftAndRightBrace = hasRightBrace.and(hasLeftBrace);  
var result = hasLeftAndRightBrace.test("{key:value}");
```

predicate的lambda案例
```
movies.stream().filter(movie -> movie.getLikes() > 10) 
```

#### UnaryOperator
一元操作器，接收一个值，返回一个值。

### lambda表达式
python中lambda就是匿名函数！[[标准库.内置函数#lambda(参数 表达式)]]

java中，lambda表达式用于将：接口的“匿名实现类”的对象 转换为 lambda表达式
因此，==**在Java中，lambda表达式是对象**，而不是函数==，必须依附于函数式接口！
	函数式接口，也即接口中只有一个抽象方法！ ^e118d1

注意：lambda简写，只有在对官方方法非常熟悉时，才会高效使用。初学的话，先老老实实用更直观的！
凡是确定的东西，都可以删掉。突出一个懒！懒才是代表最先进生产力！

#### 适用情况
接口中只有一个抽象方法的情况，也就是函数式接口，就是使用lambda表达式的唯一场景。 ^4ea6db

lambda表达式的本质是：充当接口实现类的对象！
本质也是：匿名函数，没有给实现类的方法取名字。

->  lambda操作符，或叫箭头操作符，goes to
左边：lambda形参列表，也即接口中需要重写的抽象方法的形参列表。
右边：lambda体，也即接口的实现类中重写的方法体（具体你要调的函数名）。

注意：关于lambda表达式，不一定全部都是 () -> {}
如果只有一个参数，可以不用括号， a -> {}
如果方法块中没有显式的用return返回值，就不能加{} , a -> b
但是不代表没有返回值，一行代码有可能是隐式的返回结果的

### 方法引用
看作是：lambda表达式的进一步简化！

这么理解：(str) -> { return str.toUpperCase()}   ==假如方法体就是一个现成的方法==，那还要手写一边调用吗？不用了，直接引用这个方法不就行了吗！就是这个意思
String :: toUpperCase
#### 格式
类(或对象）`::` 方法名

#### 适用的三种情况
当满足lambda表达式，且接口实现类的：1、形参列表；2、返回数据类型一致时，可以用方法引用
如果lambda表达式==仅仅只调用一个已经存在的方法==时，可以使用方法引用。
直达灵魂深处！

方法引用
```
Consumer<String> consumer = System.out::println;
```


##### 实例方法引用
当lambda主体与某个实例的方法完全一致时，可以使用实例方法引用，格式：**对象 :: 实例方法**

函数式接口中的抽象方法`get()` 与其内部实现时调用**对象的某个方法**getName()的==形参列表和返回值类型都相同时==。可以考虑使用getName()对get()的替换、覆盖。

注意：内部方法不能是静态方法！是对象调用的。
```
Employee emp = new Employee(1001, "马化腾"， 34， 6000)；

Supplier<String> sup1 = new Supplier<String> {
	@override
	public String get() {
		return emp.getName();
	}
}

//2.lambda表达式
Supplier<String> sup1 = () -> emp.getName();

//3.实例对象的方法引用
Supplier<String> sup1= emp :: getName();   //get方法是String类型，emp.getName()也是String类型，满足！


```

##### 静态方法引用
假如lambda主体和某个静态方法完全一致时，可以用静态方法引用。格式：**类名  :: 静态方法**


##### 参数1为调用者
当lambda参数列表中第一个参数作为调用者，参数列表的后续参数作为实例方法的参数时，可以使用本方法引用。格式： **类  ::  实例方法**
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/%E5%90%8D%E7%89%87.PNG)


函数式接口中的抽象方法a与其内部实现时调用**对象的某个方法**b的返回值类型都相同时。同时，抽象方法a中有n个参数，方法b中有n-1个参数，且抽象方法a的第一个参数作为方法b的调用者，且抽象方法a的后n-1个参数与方法b的n-1个参数的类型一致。可以替换。
注意：此方法b是非静态方法，需要对象调用，但是形式上，写对象a所属的类！

外函数有n个参数，内函数有n-1个，剩下一个是调用主体o1。可以用本方法
```
Camparator<String> com1 = new Comparator<String>() {
	@override
	public int compare(String o1, String o2) {
		return o1.compare(o2)
	}
}

// lambda表达式
Camparator<String> com1 = (o1,o2) -> o1.compare(o2);

// 类::实例方法
Camparator<String> com1 = String :: compareTo;
```

##### 构造器引用
当lambda的主体与某个构造器方法完全一致时，使用构造器引用，格式：**类名 :: new**

```
Function<Integer, Employee> func2 = Employee :: new;  //调用的是Employee类中参数是Integer类型的构造器
```

说明：调用了类名对应的类中某一个确定的构造器。具体调用的是类中哪一个构造器，取决于函数式接口的抽象方法的形参列表。

[[Java_02_面向对象#接口匿名实现类的匿名对象]]  ->  [[#lambda表达式]]   ->   [[#方法引用]]
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B42%E6%9C%886%E6%97%A5%2023_37_23.jpg)

## 流式计算 Stream
Stream API是真正把函数式编程风格引入到Java中，是Java8最重要的新特性。
[阿里云技术：总结｜Stream技术在真实案例中的应用](https://mp.weixin.qq.com/s/mZbejfgxLVwO7pu4nkRjIg)

stream流中的方法，之所以能利用lambda表达式，是因为他们都是接口，且都只有一个抽象方法需要实现，满足函数式接口的条件，具体的：

| 方法     | 参数（接口）        | 接口的实现类  |
| ------ | ------------- | ------- |
| filter | new Predicate | test()  |
| map    | new Function  | apply() |
|        |               |         |
|        |               |         |


集合Collection，是存储数据的，面向内存
流Stream，是从Mysql接收过来的集合数据，进一步==操作数据==的（过滤、排序、切片等），面向CPU。联想到[[mysql.DQL]]
	虽然mysql等关系型数据库具备（过滤、排序、切片等功能），这时我们用不到Stream！但是当引入了非关系型数据库，例如[[redis]]时，需要我们在Java中利用Stream对拿来的数据进行操作，然后再返回给前端。

Collection看做是储水罐，stream看做是接在储水罐上的水管，可以接各种各种的水管（过滤芯、排序型等等）

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/IMG_1154.PNG)

### 基本步骤
1、实例化一个流的对象
2、一系列中间操作 (如果没有终止操作，中间操作不会执行：惰性操作）
3、终止操作          ->  forEach( )

注意：1、一系列操作不会影响原本数据！
2、建议使用==链式编程==，否则终止操作后就不能用了！

### 创建stream
方式1：通过collections
```
movies.stream()  
        .filter(movie -> movie.getLikes() > 10)    
        // 只有满足了条件的元素，才会从collection储水罐中被抽水出来！  
        .count();
```

方式2：通过Arrays
```
int[] list = new int[]{1,2,3};  
Arrays.stream(list).forEach(n -> System.out.println(n));
```

方式3：通过任意数量对象，静态工厂方法
```
Stream.of(1, 2, 3)；
```

方式4：有限/无限流
![[#^e2a31d]]

**利用generate创建无限流**
`Stream.generate(Supplier<?> s)`  传递一个供给器（不需要参数，返回一个值），生成无限流
```
var stream = Stream.generate(() -> Math.random());

// 惰性，不会马上生成随机数，只有当调用stream时才生成！
stream.limit(3).forEach(n -> System.out.println(n));
```

**利用iterate创建无限流**
`Stream.iterate(seed, UnaryOperator)`
```
Stream.iterate(1, n -> n+1).limit(10).forEach(n -> System.out.println(n));
```

创建并行流
parallelStream  
只有当任务是独立的时，它才有用。如果任务不是独立的，并且必须共享相同的资源，则必须使用锁（主要是Java中的synchronized关键字）来保证它们的安全，此时它们的运行速度慢于正常的迭代。

当你使用并行流时，流中的元素被拆分成多个子任务，这些子任务被提交给`ForkJoinPool`进行并行处理
[[JUC#Fork/Join 分支合并框架]]

### 中间操作
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%202%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B43%E6%9C%8827%E6%97%A5%2012_50_04.PNG)
#### 筛选
##### filter(判断语句)
`filter( (a) -> {一个判断语句 return true or false;} )`
有一个集合，集合中的每一个a，当满足条件的a，返回一个新的集合。

filter的lambda中不要加{ }，否则你还要return true or false，你烦不烦！
直接写判断式！！

提供一个[[#判断型 Predicate Interface]]，返回符合条件的值。
filter有且仅有一个抽象方法需要实现（test)

```
movies.stream()  
	    .filter(movie -> movie.getLikes() > 10)    
        .count();
```
可以链式调用

以下是开发过程中总结的适用场景
1、条件筛选：根据指定条件对流中的元素进行筛选，例如筛选大于某个阈值的元素。
```
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6);  

list.stream()  
        .filter(i -> i > 5)     # 直接写判断式，不要加{ }
        .forEach(i->{
            System.out.println(i);  
        });
```

2、非空过滤：过滤掉流中的空值或空字符串。
```
List<String> strings = Arrays.asList("apple", "", "banana", "cherry", "");

List<String> nonEmptyStrings = strings.filter(s->{
	!s.isEmpty()
}).collect(Collectors.toList());
```

3、对象过滤：根据对象的属性值来过滤对象流，例如筛选出满足某个条件的用户对象。
```
List<User> users = Arrays.asList(Alice,Bob,Charlie);

List<User> filterUsers = users.stream().filter(user->
	user.getAge() > 25
).collect(Collectors.toList());
```

##### map(return)
映射，提供一个[[#函数型 Function Interface]]，返回一个值，体现映射关系
	可以用lambda表达式写
`map( (a) -> {a.func(); return 一个东西 } )`

map接收一个函数作为参数，该函数会==应用到每一个流元素==上，并将==函数的返回值作为新的流元素==

以下是开发过程中总结的适用场景
1、数据类型转换：将一个类型的数据转换为另一个类型的流，例如将一个字符串流转换为对应的整数流。
```
# 这里就用Arrays.asList()快速创建一个固定大小的List，不用new ArrayList再add了
List<String> strList = Arrays.asList("1","2","3");

strings.stream().map(str -> {
	return parseInt(str);
}).collect(Collections.toList());
```

2、对象转换：将一个对象流转换为另一个对象流，例如，对一个包含用户姓名的流进行转换，生成一个包含用户对象的流
```
List<String> names = Arrays.asList("Alice","Bob","Charlie");

List<User> users = names.stream().map(name ->{
	return new User(name);
}).collect(Collectors.toList);
```

3、属性提取：从一个对象流中提取指定属性的流，例如从一个包含学生对象的流中提取学生姓名的流。
```
List<Student> students = Arrays.asList(Alice,Bob,Charlie);

List<String> names = students.stream().map(student ->{
	return student.getName;
}).collect(Collectors.toList());
```

举例：获取员工姓名长度大于3的员工
```
employees.stream()
		.filter(emp->emp.getName().length()>3)
		.forEach(System.out::println)
```

4、flatMap()   将二维数组，打平成一维
```
var stream = Stream.of(List.of(1,2,3),List.of(4,5,6));

stream.flatMap(list -> list.stream())
		.forEach(n -> System.out.println(n));
```

紧接着，获取员工姓名长度大于3的==员工的姓名==（当我们需要元素的某一部分值时，需要用到映射）
```
employees.stream()
		.filter(emp->emp.getName().length()>3)
		.map(Employee::getName)
		.forEach(System.out::println)
```

把数据流中每个元素应用到函数上，结果生成返回一个新的数据流。
```
movies.stream()
		.map(movie -> movie.getTitle())
		.forEach(name -> System.out.println(name));
```

********************
#### 切片
##### limit(n)
限制输出前n个
##### skip(n)
跳过前n个元素
```
分页器,假设我们想要第三页的数据

movies.stream()
		.skip((page-1)*pageSize)
		.limit(pageSize)
		.forEach(m -> System.out.println(m.getTitle()));

```
##### takeWhile(lambda)
只有满足[[#判断型 Predicate Interface]]时，才允许通过本管道
takeWhile会在遇到false时直接停，filter是全局找所有满足条件的！

##### dropWhile(lambda)
满足[[#判断型 Predicate Interface]]之前的（包含自己）全部都丢掉，只保留之后的。

***************************
#### 排序
##### sorted(定制Comparator)
1、如果集合中的每个元素对应的类中自带了实现Comparable接口，那直接调用sort()进行排序即可。
```
# 字符串自己就实现了Comparable接口

String[] strArr = new String[]{"GG","DD","MM","SS","JJ"};  
Arrays.stream(strArr).sorted().forEach(System.out::println);
```

2、如果集合中的每个元素对应的类中没有Comparable接口，就要在Sort(定制Comparator)
lombda  ==> [[标准包.类#Comparator`<T>`接口]] 实现 compare(o1, o2)方法
```
arr1.stream()
	.sort( (o1,o2)->{
	return o1.getAge - o2.getAge;
	})
```

有一个集合，集合中的抽取a 和 b，当利用（a.getSort()的值）compateTo排序，返回一个加工过的新集合。

假如自定义的类，没有实现Comparable接口的抽象方法(compareTo)，就不具备比较特性，所以你调用sorted()必然报错，除非你调sorted方法时传递一个外部比较器Comparator！
带参数的，是传入外部比较器，进行定制比较。
```
list.stream().sorted((o1,o2)->o1.getAge()-o2.getAge()).forEach(System.out::println)

其中(o1,o2)->o1.getAge()-o2.getAge() 就是外部比较器的lambda表达式
```

```
movies.stream().sorted()  
        .sorted((a, b) -> a.getTitle().compareTo(b.getTitle()))

转换为：
movies.stream().sorted()  
        .sorted(Comparator.comparing(Movie::getTitle))

相当于基于Movie类创建一个，基于title的比较器
```

.reversed()  进行倒序，在sorted方法内部！   `.sorted(Comparator.comparing(Movie::getTitle).reversed)`

##### distinct()
去重，返回唯一值
	基于hashCode()和equals()去除重复元素
```
movies.stream().filter(Moive::getLikes)
		.distinct()
		.forEach(System.out::println);
```
##### peek()
用来检查哪个语句出现问题，将每个队列元素显示出来
```
movies.stream()
		.filter(m -> m.getLikes() > 10)
		.peek(m -> sout("filtered:" + m.getTitle()))
		.map(Movie::getTitle)
		.peek(t -> sout("mapped:" + t))
		.forEach(System.out::println)
```
*************************
终止流，以下这些方法一旦调用，整个流就结束了！
### 终止操作
#### `forEach(consumer)`  

[[#collection.stream().forEach(Consumer<> action)]]
用来终止流的！返回void，所以是最后一部终止操作！
![[#^00e6e4]]

```
.forEach(System.out::println)
```

思考：forEach和map都可以利用lambda表达式，迭代每一个item进行操作，有什么区别？
ChatGPT回答：确实，两个方法都是常见的用于迭代集合中元素并进行操作的方法，都可以利用lambda表达式实现。
- forEach用来遍历元素，并对每个元素进行操作，没有返回值，不会返回修后的结果！
- map用来遍历元素，对每个元素进行操作，并将操作结果收集到一个新的Stream中！

#### anyMatch(predicate)
判断stream中是否有任一满足判断的元素，返回布尔值

- allMatch(predicate)   所有元素都满足才返回True
- noneMatch()

#### findFirst()
返回stream中第一个元素，返回的是Optional类型的元素，必须接一个`get()`方法才能拿到匹配到的元素。
可以先接排序操作，然后再接findFirst方法，获取到排完序之后的第一个元素
```
movies.stream.findFirst().get();
```
#### count()
返回stream中的个数
#### max(Comparator)
基于自定义比较器，对元素进行比较，返回最大的

返回最高工资的员工
```
employees.stream().max((e1,e2)->Double.compare(e1.getSalary(),e2.getSalary()));
```

返回最高工资员工的工资（基于Employee的映射）
```
employees.stream()
		.map(Employee::getSalary)
		.max((salary1,salary2)->Double.compare(salary1,salary2));
```

```
movies.stream
		.max(Comparator.comparing(Movie::getLikes))   // 按点赞数进行比较，返回最大值
		.get()
```

#### reduce(seed, BinaryOperator)
规约！
```
list.stream().reduce((x1,x2) -> x1+x2);
```

reduce(BinaryOperator)
```
employeeList.stream().map(emp::getSalary).reduce(Double::sum);
```

#### collect(Collectors.toList())
收集。固定下来
由于stream调用的一系列方法不影响原来的数据，所以需要收集将结果存起来！
```
list.stream().filter(emp->emp.getSalary() > 6000).collect(Collectors.toList());
```
toList()    将stream过滤过的数据，固定下来，收集到List中
toSet()
toMap()
```
//key (title)
//value (likes)

movies.stream().filter(m->m.getLikes>10)
			.collect(Collector
			.toMap(Movie::getTitle,Movie::getLikes);
```

summarizingInt()        将整数的平均数，最大数、最小数都返回出来，生成一个报告
joining(delimiter)        将数组的元素，用符号拼接。 a,b   而不是`[a, b]`
groupingBy(Function)     按函数逻辑进行分组
```
movies.stream().
```


思考：Stream.map 和 Stream.forEach的异同
```
map + collect to list方式
        chapterList.stream().map(chapter -> {  
            //得到课程每个章节  
            ChapterVo chapterVo = new ChapterVo();  
            BeanUtils.copyProperties(chapter,chapterVo);  
            finalChapterList.add(chapterVo);  
            return chapterVo;  
        }).collect(Collectors.toList());  

forEach方式
        chapterList.stream().forEach(chapter -> {  
            ChapterVo chapterVo = new ChapterVo();  
            BeanUtils.copyProperties(chapter,chapterVo);  
            finalChapterList.add(chapterVo);  
        });
```

都是遍历chapterList中每个chapter，然后对每个chapter进行处理。
map.collect(Collectors.toList()) 会收集到一个全新的List中，而不是finalChapterList；
forEach是终止操作，直接进入finalChapterList
那map的使用场景是什么呢？
利用map遍历数组，原数组不会改变，利用.collect返回一个新数组，不对原数组做改变
利用forEach遍历数组，直接对原数组中每个元素进行处理，没有返回值！

| XX  | map                               | forEach          |
| --- | --------------------------------- | ---------------- |
| 返回值 | 返回一个新Stream对象                     | void             |
| 2   | 遍历后，返回到原数组                        | 遍历后，不返回到原数组      |
| 3   | 适用于：对每个元素，加点减点的操作<br>映射好之后，收集成新数组 | 适用于：直接在当前数组上进行操作 |

## 多线程 Multi-threading
ref :  [[并发与并行]]  、 [[JUC]]
### 创建线程的四种方式

^874c89

#### 方式一：继承Thread类（不推荐）

^b08324

缺点：继承的单继承，接口可以多实现！
所以更推荐[[#方式二：【常用】实现Runnable接口]]

1、创建一个类，继承Thread类
2、然后==重写Thread类的run()==，将此线程要执行的操作，写在重写方法体中。
3、实例化继承了Thread类的子类的对象
4、通过该对象调用start()方法 ^ef90e2
```
//继承Thread类，然后重写run
public class EvenNumberTest extends Thread{  
    @Override  
    public void run() {  
        for (int i = 1; i < 100; i++) {  
            if (i % 2 == 0) {  
                System.out.println(i);  
            }  
        }  
    }  
}

// main方法中，实例化，并start
EvenNumberTest thread1 = new EvenNumberTest();  
thread1.start();

for循环逻辑加到main中

>输出：
>从输出结果可以看到，两个线程在交替使用CPU资源，一会Thread，一会main交替出现，美妙！
Thread-0:2
Thread-0:4
main:2
main:4
```

^6a7a1e

注意：1、不能用run()替换start()，因为如果用run，想当于一个线程
      2、线程start之后，不能再调用start

也可以用继承Thread类的子类的匿名实现类
创建的是Thread类的子类的对象


Q：为什么继承Thread类的方式无法共享数据，但是实现Runnable接口的方式可以共享数据？
A：因为继承Thread类，将会产生多个实例进程，之间是不容易共享数据的（除非声明static），但是实现Runnable接口的话，可以共享同一个Runnable实例给多个线程，他们可以读取到相同的数据！

##### 共享数据
继承Thread类的方式创建线程，共享数据
1、声明一个共享数据的类，其中field
2、在继承Thread类的类中传递共享数据的类，然后赋值
3、new线程时，都使用相同的共享数据的类即可。
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B42%E6%9C%8826%E6%97%A5%2014_32_14.jpg)
```
class Clerk{  
    private int productNumber = 0;

class Producer extends Thread{  
    // Field  
    private Clerk clerk;
    
    // 自定义构造器  
	public Producer(Clerk clerk){  
	    this.clerk = clerk;    //继承Thread类的方式，共享数据的方法！  
	}  
	// 重写run方法
	@Override  
	public void run() {}
```

^63b4a7

#### 方式二：【常用】实现Runnable接口
1、创建一个实现Runnable接口的类
2、==实现接口中的run方法==，将此线程要执行的操作，写在重写方法体中。
3、创建当前实现类的对象
4、将==此对象作为参数传递到`new Thread(传到这里)`的构造器==中，创建Thread类的实例
5、通过Thread类的实例，调用start方法

因为只会实例化一个Runnable接口，所以同步监视器用this即可~

```
// 实现Runnable接口
public class EvenNumberTest2 implements Runnable{  
    @Override  
    public void run() {  
        for (int i = 1; i < 100; i++) {  
            if (i % 2 == 0) {  
                System.out.println(Thread.currentThread().getName()+ ":" + i);  
            }  
        }  
    }  
}

// main方法中
// 实现了Runnable接口的对象，作为参数传递到new Thread(）构造器中!
EvenNumberTest2 p1 = new EvenNumberTest2();  
Thread t1 = new Thread(p1);  
t1.start();
```

start方法运行时，如果Thread类里重写了run就执行Thread类里面的方法，如果没重写，再运行Runable接口中实现的run方法。

总结：实现Runnable接口的方式，是使用[[设计模式#Proxy 代理模式]]的思想
Runnable接口下，有两个实现，一个是**Thread类**的实现，一个是**你自定义线程类**的实现（重写run()方法），Thread类和自定义线程类是拜把子兄弟，Thread类也是自定义线程类的代理，替你完成线程的其他事情，你只要关注核心的run方法
补图！
。 ^b32dc0

| 继承Thread类                        | 实现Runnable接口                     |
| -------------------------------- | -------------------------------- |
| 创建的是Thread类的子类的对象                | 创建的是Thread类自身的对象，并传递参数：实现接口的类的实例 |
| 继承只能单继承，存在局限性                    | 接口可以多实现（更推荐）                     |
| 不能共享数据！继承Thread类的话，会产生独立的不同线程实例。 | 可以共享数据！因为多个线程共享同一个Runnable实例。    |
| 其实Thread类本质上也实现了Runnable接口       | 直抵核心，这样更直接！                      |

##### Runnable使用匿名实现类
runnableTask示例代码，只有一个需要实现的方法run()，因此满足lambda表达式的使用，简化为：
```
//方式1
Runnable runnableTask = new Runnable() {
    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            map.put(i, "A");
        }
    }
};

//方式2
Runnable runnableTask = () -> {for (int i =1; i<100, i++){
	// for循环内部执行语句
	}
}

//结合线程池使用
ThreadPoolExecutor executor = new ThreadPoolExecutor(参数1,2，3，4，5，6，7)；

Runnable runnableTask = () -> {};   //同上，方式2

executor.execute(runnableTask);

executor.shutdown();

```

^75c1f3

#### 方式3：实现Callable接口

![[JUC#Runnable接口与Callable接口对比]]

思考：Thread类的构造函数只接收Runnable接口，不接受Callable接口，那如何才能做到通过实现Callable接口创建线程呢？
回答：FutureTask实现类，即实现了Future接口，又实现了Runnable接口，将这个类的对象传递给Thread类的构造函数，它一定是认的！完美解决上面的问题！ ^27e445

new一个线程，并传递FutureTask类，示例代码！
```
//Thread类的构造函数
// 只能接收Runnable接口
Thread(Runnable task, String name);

public class Demo1 {  
    public static void main(String[] args) throws ExecutionException, InterruptedException {  

        FutureTask<Integer> futureTask1 = new FutureTask<>( ()-> {  
            sout(Thread.currentThread().getName()+"进入callable接口");  
            return 1024;  
        });  

		//将FutureTask对象，传递给Thread构造函数
		//因为FutureTask实现类，实现了Runnable接口
        new Thread(futureTask1,"luck").start();  

		// 从FutureTask返回值中获取异步任务的返回结果
        System.out.println(futureTask1.get());  
    }  
}
```

##### Callable使用匿名实现类
```
//结合线程池使用
ThreadPoolExecutor executor = new ThreadPoolExecutor(参数1,2，3，4，5，6，7);

Callable callableTask =  () -> {
	for(int i = 1; i<100; i++){
	//循环内容
	}
	return i;
};

//方式2:使用submit()方法提交一个Callable任务，并返回Future对象，获取任务结果
Future<Integer> future = executor.submit(callableTask);
Integer result = future.get();

executor.shutdown();
```

#### 方式4：线程池
频繁创建和销毁线程占用大量资源，利用线程池复用线程。
手机的消息瀑布流，在线程池中开启5个线程，顶上那个用完之后，释放给后一个用。
详见 [[JUC#【实现类】ThreadPoolExecutor]]  -> 自定义线程池

### Thread类的常用方法
[[标准包.类#Thread类]]

#### 四种构造器
| 四种构造器 | 备注 |
| ---- | ---- |
| public Thread() | new一个新的线程实例 |
| public Thread(String name) | new一个指定名字的线程实例，显示在终端上，getName() |
| public Thread(Runnable target) | new一个线程实例，传递实现Runnable的实例，也即创建thread 方式二 |
| public Thread(Runnable target, String name) |  |
```
Thread.activeCount()         查看当前进程中被激活的线程数
Runtime.getRuntime().availableProcessors()        查看当前进程中可以使用的线程数

>输出:
>2        // main主线程，JVM的垃圾回收线程
>8        // 总共有8个线程可以使用！也即4个CPU
```
#### start()
本质是调用
```
// Thread.java源码中的！

public sychronized start() {

	private native void start0();   # 内部调用的是native start0，就是C语言的方法
}
```
start0是native方法，C语言编写
![[JVM#^9d4248]]

启动线程，调用线程的run()
#### run()
将线程要执行的事情，声明在run()方法中
#### currentThread()
==静态方法==，获取到当前线程，并获取到这个线程的名字，也可以是ID：`.getId()`
#### .getName()
获取线程名称，public Thread(String name)
#### setName()
设置线程名，`Thread.currentThread().setName("新的线程名")`
#### sleep(毫秒)
==静态方法==，使当前线程睡眠指定毫秒数。需要用try-catch处理异常
#### yield()
==静态方法==，一旦执行此方法，主动释放本线程的CPU使用权，让给别的线程！  大方！！
#### join()
在线程a中通过线程b调用join()，意味着线程b就进入阻塞状态，直到线程b执行结束，才会继续执行线程a。需要用try-catch处理异常
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B42%E6%9C%888%E6%97%A5%2010_22_25.jpg)
#### isAlive()
判断当前线程是否还存活
#### isDaemon()
判断是否是后台守护线程

不用的方法
- stop()方法，导致内存回收出现问题！
- suspend() 和 resume()   导致死锁问题

***************
线程优先级相关 ^a7bcad
#### getPriority()
==静态方法==，获取当前线程的优先级
NORMAL_PRIORITY = 5           默认优先级5        
MAX_PRIORITY = 10                最高优先级
MIN_PRIORITY = 1                  最低优先级
#### setPriority(Thread.MAX_PRIORITY)
设置优先级
*********************
### [[并发与并行#线程的状态]] 
线程的生命周期
源代码：Thread类的State内部类中定义了各种状态 public enum State

优化完善了阻塞状态，拆分成三个：锁阻塞、无限等待、计时等待
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/IMG_1166.jpg)

### 线程安全问题
多个线程同时访问并修改同一个资源。

最经典的例子就是：卖票。
具体代码请看：juc-part/saleticket包里的SaleTicket类和Window类。里面写了不加锁造成重复卖、超卖的现象展示；也写了加锁解决线程不安全问题的代码！

线程①通过if判断，进入if结构体，还未执行if内代码块时，线程②也进入if结构体（由于线程①还没执行代码块，所以if状态还是可以被线程②满足的），此时两个线程同时执行，妈蛋，那就麻烦了！

![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202407261534067.gif)

解决方法：必须保证线程①在操作ticke整个过程中，其他线程必须等待！——>线程同步机制，也叫做==互斥锁==！
联想到：[[并发与并行#悲观锁]]

#### synchronized关键字
注意，**synchronized关键字不能加到类上！**
只能加载方法上，或者代码块上（需要指定同步监视器，也就是锁）
	synchronized(同步监视器){需要被同步的代码块}
	是[[并发与并行#悲观锁]]
同步监视器：哪个线程获得了同步监视器，哪个线程就可以执行下面的代码，俗称：锁![[05B3F94B.png]]
必须是一个对象，并且对于多个线程来说，这个对象必须是唯一的，这样多个线程才能公用同一个锁！



##### 锁的规则
synchronized关键字加的地方：
	加到同步代码块上，锁的是一小段逻辑，🔒需要手动指定（this，或当前类.class）  
	加到实例方法上，锁定一整个方法，🔒默认是this，也就是当前对象。  
	加到静态方法上，锁定一整个静态方法，🔒默认是当前类.class（如果类不适合转成静态对象也不能硬转） ^9c8a7c

参考具体案例：juc-part/advanced/lock/Lock8Demo
1、一个普通方法被加了synchronized，锁的是整个资源类的**实例对象**，其他线程进入同步方法前要获得当前实例对象的锁，否则就不能调用这个被锁定的**实例对象的其他同步方法**，普通方法不受影响。
- 反编译中：flag: ACC_SYNCHRONIED

2、一个静态方法被加了synchronized，锁的是这个资源类的**类对象**`.class`（不管你new了多少实例，全部无视），其他线程进入静态同步方法时要获得类的锁，否则都不能调用其他静态同步方法。
- 反编译中：flag: ACC_STATIC   ACC_SYNCHRONIED

3、一个代码块被加了synchronized，锁的是括号里的对象！
- 反编译中：一个monitorenter，两个monitorexit（正常退出释放锁，异常退出释放锁），包裹业务逻辑代码！

静态同步方法与普通同步方法不存在竞态关系，各运行各的！

#Java编程规约 阿里规约：高并发时，同步调用应该去考量锁的性能损耗。能用无锁数据结构，就不要用锁；能锁区块，就不要锁整个方法体；能用对象锁，就不要用类锁。
尽可能使加锁的代码块尽可能的小，避免在锁的代码中调用[[SpringCloud#RPC]]方法（慢啊） ^ee22a2
##### 同步代码块
同步代码块，用obj作为锁！
线程去获取任意一个对象（的owner字段），其中任意一个对象，因为任意一个对象是独一无二的，它的存在就能保证独占性，它是什么不重要，它是独一无二的比较重要。让我联想到[[redis#V2：SETNX]]实现分布式锁，锁的值是什么不重要，只要它是独一无二存在的就行！

Q：在同步代码块中`synchronized{Object o}`为什么任何实例化对象，都能成为锁？
A：Object类是所有类的祖宗类，阅读底层C++的源码，Object对象（及所有对象中）天生都带着一个Monitor（也称为管程、也曾为同步监视器），Monitor里面有个字段：`_owner = null`  记录的是当前持有本对象的是哪个线程！
底层依赖的是操作系统层面用户态与内核态的切换，依赖于操作系统的互斥量（Mutex Lock）实现，成本非常高！
![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202407311253680.jpg)
 ^b9a4bb

```
class SaleTicket implements Runnable {  
    // Filed  
    int ticket = 100;  
  
//    Object obj = new Object();   // 同步监视器，锁，确保唯一  
  
    @Override  
    public void run() {  
        while (true) {            // 同步不能放在循环条件外，不然要等循环结束之后，才解锁，那就没有其他线程啥事了  
            synchronized (this){  
                if (ticket >0) {  
                    try {  
                        Thread.currentThread().sleep(3);  
                    } catch (InterruptedException e) {  
                        e.printStackTrace();  
                    }  
                    System.out.println(Thread.currentThread().getName() + "：售票，票号为" + ticket);  
                    ticket--;  
                }else{  
                    break;  
                }  
            }  
        }  
    }  
}  
  
public class WindowTest {  
    public static void main(String[] args) {  
        SaleTicket s = new SaleTicket();  
  
        Thread t1 = new Thread(s);  
        Thread t2 = new Thread(s);  
        Thread t3 = new Thread(s);  
  
        t1.setName("窗口1");  
        t2.setName("窗口2");  
        t3.setName("窗口3");  
  
        t1.start();  
        t2.start();  
        t3.start();  
    }  
}
```

##### 同步方法
如果操作共享数据的代码，完整声明在一个方法中，我们就可以将此方法声明为同步方即可。

```
public synchronized void show(){     // 在方法前加上synchronized关键字
	if (ticket > 0) {}
}
```

- 如果非静态方法，默认同步监视器就是this。在继承方式时，需要注意，this会产生三个，也会出现问题。
- 如果是静态方法时，==默认同步监视器是：当前类本身，即为Window1.class.==

注意：在进入synchronized进行同步代码块时，其实就是串行的！性能会降低，但是没办法！

Q：说一下synchronized底层原理 #Java八股/JUC 
A：底层通过一个monitor（监视器锁）完成，当被修饰的地方被线程占用时，monitor被设置为1，synchronized是隐式可重入锁，之后每次重入加1，其他线程看到monitor为1，就会进入阻塞状态，该线程完成后，设置为0，其他线程可以占用。

Q：synchronized锁升级的过程 #Java八股/JUC 
A：
jdk1.6之前，只有重量级锁（互斥），导致线程的上下文切换和用户态到内核态的切换，开销较大。
jdk1.6开始引入了：锁升级的过程（偏向锁 ->  轻量级锁 -> 重量级锁），利用对象头中threadid字段实现，主要是优化加锁和撤销锁带来的性能消耗！
偏向锁：只有一个线程时，就在厕所直接贴上我的名字
轻量级锁：当有多个线程竞争时，升级为轻量级锁，采取自旋等待，避免频繁的阻塞和唤醒（相当于在厕所门口转一转）
重量级锁：如果自旋一段时间（在厕所门口等了好久），进入阻塞状态，等正在用的线程完毕后，唤醒我！

###### 利用sychronized关键字解决懒汉式线程不安全问题
在单例模式下，解决懒汉式线程安全问题，需要同步机制来处理！ ^2cbec8

![[设计模式#^a79491]] ^f241ca

![[设计模式#^811de0]]

##### 隐式可重入性
衍生，JUC中的Lock接口，可以更精细化操作，手动上锁解锁

![[JUC#^a2d37a]]
```
synchronized(obj) {
	sout(Thread.currentThread.getName() + "外层调用");

	// 如果synchronized没有可重入性，此时就会产生死锁！
	// 外层持有锁，中层需要同一把锁，等外层释放才能获取，
	// 这些僵持住了！

	synchronized(obj){
		sout(Thread.currentThread.getName() + "中层调用");

		synchronized(obj){
			sout(Thread.currentThread.getName() + "内层调用");
		}
	}
}
```

##### synchronized锁升级的过程
![[#^ee22a2]]

在“安全性”和“性能下降”，取得平衡！

无锁  --> 偏向锁  -->  轻量级锁（[[JUC#CAS]]）  -->  重量级锁
001           101                    000                                       010

/Users/shen/Library/CloudStorage/OneDrive-个人/文档/02-学习/Learn_Java_Mosh/JUC并发编程/synchronized锁升级过程.jpg

依赖于JVM - 对象头信息 - Mark Word的标志位的不同，来体现锁升级的过程

![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202408011539872.jpg)

hashcode的位置被占用了，GC分代年龄被占用了，去哪里了？
1、第一次调用hashcode，就会存到31位hashcode位置

2、一个对象如果调用过hashcode，这个对象就不能被设置为偏向锁（这个位置被hashcode占用了），直接升级为轻量级锁！ ^d7ce71

3、轻量级锁会有拷贝；重量级锁在C语言的Object里也有记录！
4、本应该是偏向锁，但是计算了hashcode，会跳过偏向锁，直接升级为轻量级锁
5、在偏向锁过程中，遇到hashcode()计算，立马撤销偏向锁，立刻变成重量级锁！

###### 偏向锁
Java 6中引入，目的就是尽量减少用户态和内核态的切换

Mark Word中存储的是，指向==线程栈==中偏向的线程ID

大规模并发访问情况下，有可能某个线程一直都能抢到锁，==**利用偏向锁就不需要频繁“用户态->内核态“的切换。**==
	Java 15 取消和禁用了偏向锁，因为高并发下，开启后stw的开销更大！😂
	JEP 374  Disable and Deprecate BiasedLocking

你看抢票程序的运行结果就知道了！
```
a卖出第：1 号票；
a卖出第：2 号票；
a卖出第：5 号票；
a卖出第：3 号票；
b卖出第：4 号票；   // 产生竞争关系
a卖出第：7 号票；
```

偏向锁会偏向于第一个访问锁的线程，如果在接下来的运行过程中，没有被其他线程抢到，第一个线程（持有偏向锁的线程）永远不需要出发同步，消除忽略同步语句，懒得连CAS都不做！直接提高性能！
	怪不得记录的是第一个访问的线程ID

==**偏向锁只有在遇到竞争时，才会释放锁；否则不会主动释放！**==

如果不相等，表明发生了竞争，尝试使用CAS来替换Mark Word中的线程ID！锁不会升级，只不过是偏向A改为偏向B

如果竞争失败，也就是B替换不进去，就需要升级为轻量级锁！持有偏向锁的继续持有轻量级锁。

需要偏向锁的撤销：等到全局安全点（A线程完成离开的时候）（类似STW），撤销！

|                    |                                                                                                                   |
| ------------------ | ----------------------------------------------------------------------------------------------------------------- |
| +PrintFlagsInitial | java -XX:+PrintFlagsInitial<br>不用运行任何java进程<br><br># 打印偏向锁相关的参数<br>java -XX：+PrintFlagInitial \| grep BiasedLock* |
Java 6开始，默认是开启的，有4秒钟延时！
`-XX:BiasedLockingStartupDelay=0`  改为0，才可以显示出来！

如果关闭偏向锁，直接进入轻量级锁
`-XX:-UseBiasedLocking`


###### 轻量级锁
Java 6中引入，目的就是尽量减少用户态和内核态的切换

Mark Word中存储的是，指向==线程栈==中Lock Record的指针

将“原持有偏向锁的线程ID” 替换掉 “Mark Word中线程ID”。

竞争线程数量不多，CAS可以搞定，就用轻量级锁！（本质就是CAS自选锁）

如果A线程在持有轻量级锁过长，线程B在进行CAS达到一定次数后，会直接升级为重量级锁！

多少次数呢？
Java6之后：不再以固定数，而是自适应次数（同一个上一次自旋的时间和拥有锁的状态决定的）
	这次线程A，自旋了8次成功了，自旋次数加一点，9次，那大概率就会成功
	这次线程A，自旋了7次都不成功，减少次数或不自旋了，直接升级到重量级锁。
	避免CPU空转

###### 重量级锁
java 5之前，默认synchronized就是重量级锁，性能比较差！

![[计算机操作系统#^f1e0bb]]

Mark Word中存储的是，指向==堆中==monitor对象的指针。

大量的线程参与竞争了！前面都搞不定了！

底层实现：在同步块的指令前后插入：monitor enter 和 monitor exit指令。并获取对象的Monitor的控制权（`_owner`)


#### 死锁
联想mysql的死锁  [[mysql.DML#死锁]]

两个（或多个）线程互相持有对方需要的同步资源不放弃，都在等待对方放弃自己需要的同步资源，就形成了线程的死锁。打个比方：两个人都要吃饭，每个人都握有一只筷子🥢，都在等待对方让出这只筷子。 ^c37634

循环等待！

解决方法：
1、让线程1，一次性把需要的锁都拿到，用完全部释放，再给线程2去拿。
2、占用部分资源的线程在进一步申请其他资源时，如果申请不到，就主动释放掉已占用的资源。
3、将资源改为线性顺序。申请资源时，先申请序号小的，再申请序号大的。

注意：当调用同步方法时，一定要意识到，你这时候握着一把锁，避免形成死锁。

示例代码@阳哥
```
public class DeadLockDemo {  
    public static void main(String[] args) {  
        Object objectA = new Object();  
        Object objectB = new Object();  
  
        new Thread(()->{  
            //线程需要执行的逻辑  
            // 持有o1对象的同步代码块  
            synchronized (objectA){  
                System.out.println(Thread.currentThread().getName()+"\t 持有objectA锁，希望获得objectB锁");  
                try {TimeUnit.MILLISECONDS.sleep(1000);} catch (InterruptedException e) {throw new RuntimeException(e);}  
                synchronized (objectB){  
                    System.out.println(Thread.currentThread().getName()+"\t 成功获得objectB锁");  
                }  
            }  
        },"A").start();  
  
        new Thread(()->{  
            //线程需要执行的逻辑  
            synchronized (objectB){  
                System.out.println(Thread.currentThread().getName()+"\t 持有objectB锁，希望获得objectA锁");  
                try {TimeUnit.MILLISECONDS.sleep(1000);} catch (InterruptedException e) {throw new RuntimeException(e);}  
                synchronized (objectA){  
                    System.out.println(Thread.currentThread().getName()+"\t 成功获得objectA锁");  
                }  
            }  
        },"B").start();  
    }  
}
```

示例代码@宋红康老师
```
/*
s1和s2形成死锁的演示
 */

public class DeadLockTest {
    public static void main(String[] args) {
        StringBuilder s1 = new StringBuilder();
        StringBuilder s2 = new StringBuilder();

        new Thread() {
            @Override
            public void run() {
                synchronized (s1) {
                    s1.append("a");
                    s2.append("1");
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized (s2) {
                        s1.append("b");
                        s2.append("2");
                        System.out.println(s1);
                        System.out.println(s2);
                    }
                }
            }
        }.start();

        new Thread() {
            @Override
            public void run() {
                synchronized (s2) {
                    s1.append("c");
                    s2.append("3");
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized (s1) {
                        s1.append("d");
                        s2.append("4");
                        System.out.println(s1);
                        System.out.println(s2);
                    }
                }
            }
        }.start();
    }
}
```

##### 排查死锁
jps 查处java程序允许的进程ID

jstack 进程ID
```
……
Found 1 deadlock.
```
#### Lock接口
[[JUC#可重入锁 ReentrantLock]]
[[JUC#读写锁 ReentrantReadWriteLock]]

注意：new ReentrantLock(); 一定要写在Runnable接口的实现类的属性上，千万不能写到重写的run方法内部，这样每个线程去创建锁有什么意义呢？切忌切忌！

```
public class SaleTicket implements Runnable{  
    Integer ticket = 100;  
  
    // 创建的锁，必须在类顶层，不能在run方法里！  
    private ReentrantLock reentrantLock = new ReentrantLock();

	/**  
	 * V3：利用Lock接口的中最常用的实现类：ReentrantLock，上锁！  
	 */  
	@Override  
	public void run() {  
	    while(true){  
	        try {  
	            reentrantLock.lock();  
	            if (ticket > 0) {  
	                System.out.println(Thread.currentThread().getName() + "售票，票号为：" + ticket);  
	                ticket--;  
	            } else {  
	                break;  
	            }  
	        } finally {  
	            reentrantLock.unlock();  
	        }  
	  
	        // 线程休眠10毫秒，别的线程即便进来了，也要在if边界处等待  
	        // 锁不锁也都没关系了  
	        try {  
	            Thread.sleep(10);  
	        } catch (InterruptedException e) {  
	            throw new RuntimeException(e);  
	        }  
	    }  
	}
}
```
### 线程的通信
#### wait() 与 sleep()对比

^3f108e

线程一旦执行此方法，就进入阻塞状态，并会释放同步监视器（锁），但是sleep不会释放。

|     | wait()                                                      | sleep()                               |
| --- | ----------------------------------------------------------- | ------------------------------------- |
| 相同点 | 都能使线程进入阻塞状态                                                 | 都能使线程进入阻塞状态                           |
| 不同点 | 声明在Object类中  <br>-->  [[标准包.类#wait()]]                      | 声明在Thread类中 <br>--> [[#sleep(毫秒)]]    |
|     | 只能在synchronized声明的代码块中                                      | 任何位置都能用                               |
|     | 释放锁                                                         | 不释放锁                                  |
|     | 1.通过指定时间，自动结束(timed waiting)；<br>2.通过notify唤醒，结束阻塞(waiting) | 1.只能通过指定时间，自动结束阻塞。<br>(timed waiting) |
|     | ==在哪里睡，就在哪里醒！==                                             |                                       |
特殊的点：线程处于阻塞状态下，调用interrupt()方法，会立刻退出阻塞，并抛异常！
![[标准包.类#^619a87]]
#### [[标准包.类#notify()]]

#### [[标准包.类#notifyAll()]]

### volatile关键字
volatile通常运用在高并发场景下。
具有两个特点：可见性、有序性(禁止重排，听我的顺序)；
注意：**==volatile不支持原子性==**！   [[JVM#三大特性]]

当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值==立即刷新回==主内存中； -> 可见性
当读一个volatile变量时，JMM会把该线程对应的本地内存设置为无效，重新回到主内存中读取最新共享变量

总结：volatile的写内存语义是直接刷新到主内存中；读内存语义是直接从主内存中读取。
![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202407312130904.png)


[[JVM#先行发生原则 happens-before]]在volatile关键字的运用：
规则：
1、volatile写之前，都禁止重排序到volatile之后
2、volatile读之后，都禁止重排序到volatile之前

Store                        +       Load
volatile的操作是写            后面的（普通）操作 

| volatile读                     | XX                      |
| ----------------------------- | ----------------------- |
| 在每个volatile读操作后面插入LoadLoad屏障  | 禁止把上面volatile读与下面普通读重排序 |
| 在每个volatile读操作后面插入LoadStore屏障 | 禁止把上面volatile读与下面普通写重排序 |

| volatile写                      | XX                                |
| ------------------------------ | --------------------------------- |
| 在每个volatile写操作前面插入StoreStore屏障 | 保证volatile之前的所有普通写操作都已经刷新到内存中     |
| 在每个volatile写操作后面插入StoreLoad屏障  | 禁止把上面volatile写与下面可能的volatile读写重排序 |

#### 适用场景
1、单一赋值，复合运算不可以用（i++)
不适合依赖当前值计算的场景（如 i=i+1、i++）
```
volatile int i = 10;     OK

volatile i++             绝对不行
```
![[Java_01_基础#^a2d163]]


2、状态位标识，使用volatile
要想保证原子性，必须还要依赖于：锁！光靠volatile搞不定！
适合用在保存某个状态的boolean值或int值  [[设计模式#State 状态模式]]

适用于某些多线程共享的属性，共同维护的状态！
[[标准包.类#线程中断机制]]中利用volatile关键字修饰的中断标识位，让不同线程更新状态

volatile关键字的作用，示例代码：
```
public class VolatileSeeDemo {  
//    static boolean flag = true;  
    static volatile boolean flag = true;  
  
    public static void main(String[] args) {  
        new Thread(()->{  
            System.out.println(Thread.currentThread().getName()+"\t come in");  
            while(flag){  
                // 主内存中的标识位，一旦被修改为false，就跳出循环  
            }  
            System.out.println(Thread.currentThread().getName()+"\t 检测到flag被修改，结束循环");  
        },"t1").start();  
  
        try {  
            TimeUnit.MILLISECONDS.sleep(1000);} catch (InterruptedException e) {throw new RuntimeException(e);}  
  
        new Thread(()->{  
            flag = false;  
            System.out.println(Thread.currentThread().getName()+"\t 将标识位修改为false"); 
        },"t2").start();  
    }  
}
```

3、先用volatile修饰value，读时不加锁（利用volatile的可见性），写时加锁（利用synchronized的原子性）
张弛有度！ ^6118f0
```
public class Demo{
	private volatile int value;

	public int geyValue(){     // 读方法，不加锁，利用volatile保证可见性
		return value;
	}

	public synchronized int increment(){   // 写方法，加锁，保证操作上线程安全
		return value++;
	}
}
```

^1ae4c9

4、Double Check Lock双端锁
待补充 ^5d3c83


## File类与IO流
一个常识：文件以字节形式存储在磁盘中；读取到内存时转为字符形式；
当文件被保存到磁盘时，编码方式，转为字节形式，存储在磁盘上；
当文件被读取到内存时，解码方式，转为字符形式，放在内存上；
### File类
java.io.File类
File类的对象：存在磁盘中的某一个具体文件（或文件夹），就可以看做是File类的一个具体实例化对象；

public File(String pathname)          以pathname为路径创建File对象，可以是相对或绝对路径
public File(String parent, String child)   以parent为父路径，child为子路径创建File对象
public File(==File== parent, String child)       根据一个父File对象，和子文件路径创建File对象

Test方法  -->  相对路径  -->   对应到Module
main方法  -->  相对路径  -->   对应到Project

#### 常用方法
getName()   获取文件或文件夹的名称
getPath()     获取路径
getAbsolutePath()   获取绝对路径（带盘符），返回String
getAbsoluteFile()     获取绝对路径表示的文件，返回File对象
getParent()               获取上层文件目录路径，返回String 父路径
length()            获取文件长度（字节数），目录不能获取长度
lastModified()    获取最后一次修改的时间，毫秒值

list()               返回一个String数组，表示File目录中所有子文件或目录
list(FilenameFilter filter)    传递FileNameFilter接口的匿名实现类。

listFiles()       返回一个File对象的数组，表示File目录中所有子文件或目录

renameTo(File file)     把文件重命名为指定路径。移动+重命名

exists()         判断当前文件或目录是否实际存在物理磁盘上
isDirectory()
isFile()
canRead()
canWrite()
isHidden()

createNewFile()      创建文件
mkdir()                     创建文件目录
mkdirs()                      创建文件目录，上层一并创建
delete()                     删除文件或文件夹

```
练习：创建一个与hello.txt文件在相同文件目录下的另一个名为abc.txt文件

File file1 = new File("hello.txt");
String parentPath = file1.getAbsoluteFile.getParent();

File file2 = new File(parentPath, "abc.txt");
```

注意：File类声明了新建、删除、获取名称、重命名等方法，并没有涉及对文件内容的读写操作。==**要想实现文件内容的读写，就需要IO流！**==
File类对象，通常最为IO流操作的文件端点出现。代码层面，将File类的对象作为参数传递到IO流相关类的构造器中。

### IO流
*注意：这里的IO流，和操作集合Collection的Stream没有直接关系！是两个完全不同的概念！*

 IO流有两个角度的解释：
 1、我们站在：==程序或内存的角度==
	Input：表示从磁盘，读取数据，到代码中；
	Output：表示从代码中，向磁盘写数据；

2、我们站在：Server和Client的角度   [[Java_网络编程#Socket]]
==**注意：正是有了Socket通过网络建立数据连接通道，才让Server和Client的IO成为可能！**==
![[Java_网络编程#^6f674d]]
	Input：表示从Client向Server发起请求
	Output：表示从Server响应给Client ^b166ff

存储单位：字节流、字符流（1个字符=2个字节）

| **文本文件** | **txt、java代码** | 用字符流 |
| -------- | -------------- | ---- |
| 非文本文件    | doc、xlsx、jpg   | 用字节流 |

流的API分类
Java的IO流涉及40多个类，都是从下面4个抽象基类派生出来的

| 抽象基类 | 输入流         | 输出流          | 备注                   |
| ---- | ----------- | ------------ | -------------------- |
| 字节流  | InputStream | OutputStream | 其他文件例如图片jpg等，都使用字节流！ |
| 字符流  | Read        | Write        | 只能处理文本文件，txt         |
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/%E5%9B%BE%E5%83%8F2024-5-8%2014.03.JPG)

![[#^766ed9]]


#### InputStream / OutputStream

InputStream  输入流，就是可以往这个流里写数据的

OutputStream  输出流，就是可以从这个流里读数据的！


#### 文件流
##### FileReader
常用方法：
read()，返回int类型数字，表示读入的字符个数；-1，表示读到末尾了！一般搭配len来循环！
`
``read(char[] cbuf)`

```
//1 创建File类的对象，对应着hello.txt文件
File file = new File("路径/hello.txt");

//2 创建输入型的字符流，用于读取数据
FileReader fr = new FileReader(file);

//3 读取数据，并显示在控制台
//方式1:一个个读，效率低
int data = fr.read();
while(data != -1){
	sot((char)data);
	data = fr.read();
}

//方式2:利用char buffer小车，一次装5个字符
char[] cbuffer = new char[5];
int len;
while((len=fr.read(cbuffer)) != -1){
	for (int i = 0; i < len; i++){
		sout(cbuffer[i]);
	}
	
}

// 4 流的关闭，否则内存泄露
fr.close();
```
##### FileWriter
```
// 创建File类的对象，指明写出文件的名称
File file = new File("路径/info.txt");

// 创建输出流
FileWriter fw = new FileWriter(file,true);  # append=true 表示追加

// 写出的具体过程
fw.writer("hahaha");

// 关闭输出流
fw.close();

```
##### FileInputStream
向文件中写入数据！

示例代码
```
public void test1() {  
    FileInputStream fis = null;  
    FileOutputStream fos = null;  
  
    try {  
        File srcFile = new File("rockyshen.jpg");  
        File desFile = new File("rockyshen_copy.jpg");  
  
        fis = new FileInputStream(srcFile);  
        fos = new FileOutputStream(desFile);  
  
        byte[] buffer = new byte[1024];   // 1kb  
        int len;  
        while ((len = fis.read(buffer)) != -1){  
            fos.write(buffer,0,len);  
        }  
        System.out.println("复制成功");  
    } catch (IOException e) {  
        e.printStackTrace();  
    } finally {  
        try {  
            if(fis != null){  
                fis.close();  
            }  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
        try {  
            if(fos != null){  
                fos.close();  
            }  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
    }  
}
```
##### FileOutputStream
将文件中的数据，往外输出！

#### 缓冲流
当文件大时，效率明显优于文件流！(一个几百M的视频，文件流要11秒，缓冲流只要0.4秒)
一般包在文件流外面

示例代码：
```
@Test  
public void test01(){  
    File srcFile = new File("rockyshen.jpg");  
    File desFile = new File("rockyshen_copy.jpg");  
  
    FileInputStream fis = new FileInputStream(srcFile);  
    FileOutputStream fos = new FileOutputStream(desFile);  
      
    // 在文件流基础上，包一层缓冲流  
    BufferedInputStream bis = new BufferedInputStream(fis);  
    BufferedOutputStream bos = new BufferedOutputStream(fos);  
  
    byte[] buffer = new byte[1024];   // 1kb  
    int len;  
    while ((len = fis.read(buffer)) != -1) {  
        bos.write(buffer, 0, len);  
    }  
    System.out.println("复制成功");  
      
    //先关外层，再关内层  
    bos.close();  
    bis.close();  
    //fos.close();  
    //fis.close();  
}
```
##### BufferedInputStream
`read(byte[] buffer)`
##### BufferedOutputStream
`writer(byte[] buffer, 0, len)`
##### BufferedReader
`read(byte[] cBuffer)`

readLine()      一读一整行（不包含换行符，读完时返回null）
```
String data;
while ((data = br.readLine() != null){
	sout(data);
}

br.close();
```
##### BufferedWriter
`writer(byte[] cBuffer, 0, len)`
newLine()     表示换行操作
flush()           刷新方法，每当调用此方法，主动将内存中数据写入磁盘
#### 转换流
字节  <---> 字符

字符编码：我们能看得懂的（字符）  ---->   我们看不懂的（字节）
字符解码：我们看不懂的（字节）    ----->    我们能看懂的（字符） ^78e871
##### InputStreamReader(fis, "utf-8")
将输入型的字节流外，转换为输入型的字符流。

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B45%E6%9C%888%E6%97%A5%2013_43_45.jpg)

```
File file1 = new File("dbcp_gbk.txt");
File file2 = new File("dbcp_gbk_to_utf8");

FileInputStream fis = new FileInputStream(file1);
InputStreamReader isr = new InputStreamReader(fis, "GBK");

FileOutputStream fos = new FileOutputStream(file2);
OutputStreamWriter osw = new OutputStreamWriter(fos,"utf-8");

char[] cBuffer = new char[1024];   //快递小车，1kb
int len;
while( (len = isr.read(cBuffer)) != -1 ) {
	osw.writer(cBuffer, 0, len);
}

osw.close();
isr.close();
```
##### OutputStreamWriter(fos, "gbk" )
将输出型的字符流外，转换为输符型的字节流。，
#### 对象流
将Java对象的属性，保存起来，就需要用到数据流 或 对象流

对象的序列化机制：将Java对象转为二进制流，并持久保存到磁盘上，或通过网络传输！其他程序获取到这个二进制流，可以恢复成Java对象。
##### 数据流
数据流只能处理：基元数据类型 和 String类型；自定义类型处理不了！
DataInputStream
	byte readByte()
	……
	String readUTF()
	`readFully(byte[] b)`

DataOutputStream

**自定义的Java类，要向实现序列化，需要满足以下要求：** ^edd6f3
1、实现Serializable接口
2、接上，是Serializable接口的要求，要求声明一个全局常量：`static final long serialVersionUID = 59933885;` 用来唯一标识当前类模板
	说明：如果不声明全局常量，系统会自动生成一个针对于当前类的serialVersionUID，修改此类会导致ID变化，进而导致反序列化时出现InvalidClassException异常
3、Field也必须可序列化的。
	- 基元数据类型，默认就是可以序列化
	- 引用数据类型，必须实现Serializable接口 ^80c05e

##### ObjectOutputStream
序列化：将内存中的Java对象，持久化到文件中
```
@Test  
public void test01() throws Exception {  
    //1 造file对象，造流对象  
    File file = new File("object.txt");  
    ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));  
  
    //2 写出数据  
    oos.writeUTF("江山");   //String对象  
    oos.flush();  
  
    oos.close();  
}
```

##### ObjectInputStream
反序列化：获取一个文件，在内存中还原为Java对象
```
@Test  
public void test2() throws Exception {  
    // 1  
    File file = new File("object.txt");  
    ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));  
  
    //2  
    String str1 = ois.readUTF();  
    System.out.println(str1);  
  
    ois.close();  
}
```

#### 其他流
##### 打印流
PrintStream
println()，就是打印流下的方法！ ^cb4566
##### 标准输入输出流
System.in      标准的输入流，默认从键盘输入
System.out    标准的输出流，默认是从显示器输出（控制台）



## 反射机制
java.lang.class
java.lang.reflect.Method           代表类的方法
java.lang.reflect.Field                代表类的成员变量
java.lang.reflect.Constructor     代表类的构造器

正常使用new基于类创建实例，在一开始就明确知道操作哪个类。
**反射就是在运行时才知道要操作的类是什么，并且可以在运行时获取类的完整构造，并调用对应的方法**

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-05-16%20at%2023.29.00%402x.png)
### 基本概念
很多时候，对象的编译时类型和运行时类型不一致。 [[Java_02_面向对象#原则4：多态性]]
```
Object obj = new String("hello");

>> 编译时类型 ——> Object类型
>> 运行时类型 ——> String类型
```
如果我们想用Object类型的obj，调用compareTo()方法（显然compareTo()不是Object类型的方法，但是是（运行时类型）String类型的方法），怎么办？

**方法1**：运用[[Java_02_面向对象#向下转型 Downcasting]] instanceOf进行判断，然后再利用强制类型转换符将其转换为运行时类型的变量即可。
相当于==手动==强转的方式！
```
Object obj = new String("hello");

if (obj instanceOf String) {
	Integer result = (String) obj.compareTo("world");
	return result;
}
```

**方案2**：希望程序运行过程中，动态获取obj所属的类型。及时==自动==将你转成对应的类！
此时就需要利用反射技术。
见下图，对比实例化对象方式 和 反射方式

![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B45%E6%9C%8816%E6%97%A5%2023_18_46.jpg)

思考：反射和泛型有点像，区别是什么呢？
我的答案：
1、==泛型只关心类型整体==，相当于占个位置，不关心类型里有什么方法，也无法调用类中的方法。
2、==反射则需要具体操控未知类里面的方法==，以实现预知性的操作（非常适合框架，因为你不知道会有什么类的什么方法过来） ^797406

### 应用场景
补充具体代码，辅助理解：

#### 开发通用框架
例如Spring框架

补充AsyncFlow异步任务调度框架中利用反射执行方法的代码示例：

#### 动态代理
利用中介来控制内部实际对象的访问

#### 注解
只要我们通过反射拿到Class，Method，Field，就能通过下面的方法拿到注解，并取值！
boolean isAnnotationPresent()
getAnnotation()
getAnnotations()
getDeclaredAnnotation(Class)
getDeclaredAnnotations()

### 特点
1、可以调用运行时类中**任意的**构造器、属性、方法，包括private的属性、方法、构造器（-->暴力反射）

2、运行时类的**动态性**（可以在运行时动态获取对象所属的类，动态调用相关方法），设计框架的时候，会大量使用反射。如果需要阅读框架的源码，那么就需要学习反射。
框架 =  注解  +  注解  + 设计模式

体现==**反射的动态性**==的示例代码
```
public class Reflect {
	public <T> getInstance(String className) {    // 全类名
		Class clazz = Class.forName(className);
		Constructor constructor = clazz.getDeclaredContructor();
		constructor.setAccessible(true);
		return (T) constructor.newInstance();    // 强转
	}
}
```

思考：反射是否打破了类的封装性？
封装性：体现的是否建议我们调用内部API问题。声明了private，意味着不建议调用；
反射：    体现的是我们能否调用的问题，因为类的完整结构都加载到内存中，所以我们就有能力调用。

### 反射的应用
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/reflect.jpg)

#### 1、创建运行时类的对象

1、基于类实例clazz，调用~~clazz.newInstance()~~方法（JDK9之后被弃用）
clazz.newInstance()要求很严格：
	1、要求类里必须有空参构造器。否则抛出：InstantiationException
	2、要求类的空参构造器的权限必须为public。否则抛出：IllegalAccessException
```
Class clazz = Person.class();
Person per = (Person) clazz.newInstance();  //要求必须是空参构造器！
```

2、基于类实例clazz，调用constructor.newInstance()创建
Class.forName("全类名")    
Class类的静态方法，加载指定全类名的类，并返回对应Class对象clazz。
constructor.newInstance()就没有那么严格
	可以接收空参、也可以接收全参；
```
Class clazz = Class.forName("LearnJava.advanced.reflect.Orange");  
Constructor constructor = clazz.getDeclaredConstructor();  
constructor.setAccessible(true);  
Fruit fruit = (Fruit) constructor.newInstance();  
```

3、类的加载器（生僻）
```
方式1：调用运行时类的静态属性 .class
Class clazz1 = User.class;

方式2：调用运行时类的对象的getClass()
User u1 = new User();
Class clazz2 = u1.getClass();

方式3：调用Class的静态方法forName
String className = "com.atguigu._class";
Class clazz3 = Class.forName(className);

方式4：使用类的加载器
Class clazz4 = ClassLoader.getSystemClassLoader().loadClass("com.atguigu._class.User");

```

![[JDBC#^48c513]]
![[JDBC#^36259c]]
#### 2、获取运行时类的所有结构
内部结构1：所有属性、所有方法、所有构造器
内部结构2：父类、接口们、包、带泛型的父类、父类的泛型

|        | XX                                    | XX                       |
| ------ | ------------------------------------- | ------------------------ |
| Field  | 运行时类本身及父类中声明为public的属性                | getFields()              |
|        | 运行时类声明的所有属性（不包含父类）                    | getDeclareFields()       |
|        | (基于field对象) 获取权限修饰符、类型等               | getModifiers()、getType() |
| Method | 运行时类本身及父类中声明为public的方法                | getMethods()             |
|        | 运行时类声明的所有方法（不包含父类）                    | getDeclaredMethod()      |
|        | (基于method对象)获取注解、权限修饰符、返回值类型、方法名、形参列表 | getParametersTypes       |
| Class  | (基于clazz对象)获取运行时类的父类                  | getSuperclass()          |
|        | (基于clazz对象)获取运行时类实现的接口                | getInterfaces()          |
|        | (基于clazz对象)获取运行时类所在的包                 | getPackages()            |
|        | (基于clazz对象)获取运行时类带泛型的父类               | getGenericSuperclass()   |
|        | (基于带泛型的父类)获取运行时类的父类的泛型                | getActualTypeArguments   |

getFields()
获取运行时类本身及其所有父类中声明为public的属性
```
Class clazz = Person.class

Field[] fields = clazz.getFields();    //本身类和父类的public
for (Field f : fields) {
	sout(f);
}
```

getDeclareFields()
获取当前运行时类中声明的所有属性（不包含父类）
```
Field[] declareFields = clazz.getDeclaredFields();   
//本身类所有属性（含private
```

可以基于field对象，获取权限修饰符、类型等

getModifiers()
返回一个int类型的数

| Modifier  | int | 二进制   |
| --------- | --- | ----- |
| public    | 1   | 1     |
| private   | 2   | 10    |
| protected | 4   | 100   |
| state     | 8   | 1000  |
| final     | 16  | 10000 |
```
int modifier = field.getModifiers();
Modifier.toString(modifier);

>> private
```

getType()
获取类型
```
Class type = f.getType();

>> java.lang.String  全类名
```

getMethods()
获取运行时类本身及其所有父类中声明为public的方法

getDeclaredMethod()
获取当前运行时类中声明的所有方法（不包含父类）

可以基于返回的method对象，进一步获取注解、权限修饰符、返回值类型、方法名、形参列表getParametersTypes

- 获取运行时类的父类
- 获取运行时类实现的接口
- 获取运行时类所在的包
- 获取运行时类带泛型的父类
- 获取运行时类的父类的泛型
应用场景：`public class CustomerMapper extends Mapper<Customer>`  利用此方法，获取到CustomerMapper的父类（也就是Mapper）的泛型（也就是Customer），我们能获取到Customer类，就不需要额外再传递了。自动判断是Customer类！ ^445344
```
Class clazz = Class.forName("全类名")；
Type superclass = clazz.getGenericSuperclass();    //获取运行时类带泛型的父类
ParameterizedType paramType = (ParameterizedType)superclass;
Type[] arguments = paramType.getActualTypeArguments();//获取运行时类的父类的泛型
```

#### 3、调用指定的结构
指定的属性、方法、构造器
和应用2类似，getFields   --->  getField(String field)

|        | XX                    | XX                                                   |
| ------ | --------------------- | ---------------------------------------------------- |
| Field  | 运行时类指定名的属性（仅public）   | getField(String field)                               |
|        | 运行时类指定名的属性（无视private) | getDeclareField(String field)<br>setAccessible(true) |
|        | (基于filed对象)向指定名的属性，写入 | set(person, "Tom")                                   |
| Method | 同理，不写了                |                                                      |
|        |                       |                                                      |

调用指定属性的示例
private String name;
```
Class clazz = Person.class;
Person per = (Person)clazz.newInstance();
Field nameField = clazz.getDeclaredField("name");
nameField.setAccessible(true);
nameField.set(per,"tom");
```

调用静态属性的示例
private static String info；
```
Class clazz = Person.class;
Field infoField = clazz.getDeclaredField("info");
nameField.setAccessible(true);
infoField.set(Person.class,"我是一个人");    // Person.class也可以写成null
```

调用私有方法的示例
```
Class clazz = Person.class;
Person per = (Person)clazz.newInstance();

Method showNationMethod = clazz.getDeclaredMethod("showMethod",String.class,int.calss);
// 注意：int类型参数，必须写int.class 不能写 Integer.class
// 值可以自动装箱，类型不能自动装箱
showNationMethod.setAccessible(true);

Object returnValue = showNationMethod.invoke(per,"CHN",10);
```

静态方法和静态属性类似，不写了！

调用指定的构造器
```
Class clazz = Person.class;
Constructor constructor = clazz.getDeclaredContructor(String.class,int.class);
constructor.setAccessible(true);
Object per = constructor.newInstance("Tom",12);
|
|  强转
V
//Person per = (Person) constructor.newInstance("Tom",12);
```

使用Constructor的实例，调用newInstance()创建运行时类的实例
```
Constructor constructor = clazz.getDeclaredContructor(String.class,int.class);
Object per = constructor.newInstance("Tom",12);
Object per = constructor.newInstance();    //空参构造器
```
Person per = clazz.~~newInstance~~()
对比基于类的对象调用newInstance()要求比较高，基于contructor调用newInstance()更灵活。

#### 4、获取类声明上的注解中的信息
框架中大量使用！
```
Class clazz = Customer.class;
Annotation annotation = clazz.getDecalredAnnotation(Table.class);
|
|  强转
V
Table annotation = (Table) clazz.getDecalredAnnotation(Table.class);
annotation.value()
```

### 反射的使用示例
@马士兵总结：反射的使用步骤

1、先获取Class对象
	方式1：Class clazz = Class.forName("类的全限定名")
	方式2：Class clazz = 对象.getClass();
	方式3：Class clazz = 类.class;

2、基于clazz（类模板对象），创建实例化对象
~~clazz.newInstance()~~    这种方法已经过时了
```
Constructor constructor = clazz.getDeclaredConstructor()
Object obj = constructor.newInstance();
```


### 反射的案例
**一道重要的练习题**
榨汁机榨水果汁，水果分别有苹果（Apple），香蕉（Banana），橘子（Orange）等
提示：
1、声明（Fruit）水果接口，包含榨汁机抽象方法：void squeeze()
2、声明榨汁机（Juicer），包含运行方法：public void run(Fruit f)，方法体中，调用f的榨汁方法squeeze()
3、声明各种水果类，实现水果接口，并重写squeeze()；
4、在src中，建立配置文件，config.properties，并在配置文件中配上fruitName=xxx（xxx为水果的全类名）
5、在FruitTest测试类中：
	(1) 读取配置文件，获取水果类名，并用反射创建水果对象
	(2) 创建榨汁机对象，并调用run()方法

详见：`C:\Users\61750\Learn_Java\IdeaProjects\JavaSE-part\src\LearnJava\advanced\reflect`
```
public class JuicerTest {  
  
    @Test  
    public void test01() throws Exception {  
        //1 读取properties，获得全类名  
        Properties properties = new Properties();  
  
        File file = new File("src/fruit.properties");  
        FileInputStream fis = new FileInputStream(file);  
        properties.load(fis);  
        String fruitName = properties.getProperty("fruitName");  
  
        //2 通过反射，创建指定全类名的类的实例  
        Class clazz = Class.forName(fruitName);  
        Constructor constructor = clazz.getDeclaredConstructor();  
        constructor.setAccessible(true);  
        Fruit fruit = (Fruit) constructor.newInstance();  
  
        //3 通过榨汁机对象调用run()  
        Juicer juicer = new Juicer();  
        juicer.run(fruit);  
    }  
}
```

![[JDBC#^e728fd]]
![[JDBC#^ff4d82]]

