其实，就是讲这个包里的内容：Java.util.concurrent  缩写：JUC 
ref : [[并发与并行]]  、 [[Java_03_高级特性#多线程 Multi-threading]]

JUC是Java高阶编程内容！后面再补！2024.5.23 面试必备技能，补了@王泽的JUC基础；
2025.7.26 来补@阳哥的JUC高级和源码了！

由Doug Lea（纽约州立大学教授），基于JSR-166，提案由Doug教授亲自完成了util.concurrent包的编写工作。在Java界是举足轻重的大神！

java.util.concurrent（jdk 1.5 加入）
java.util.concurrent.locks
java.util.concurrent.atomic

[[并发与并行#进程 Process]]
[[并发与并行#线程 Thread]]
[[并发与并行#管程 Monitor]]
[[并发与并行#并行 Parallel]]
[[并发与并行#并发 Concurrent]]

[[Java_网络编程#BIO]]
[[Java_网络编程#NIO]]

注意：main方法，本身也是一个线程哦！！！new出来的线程，都是从main方法的线程分支出去的！！
[[Java_02_面向对象#线程 -操纵-> 资源类]]

## 多线程步骤
1、创建资源类，属性，方法
2、在资源类的操作方法上加锁！   
	a. 判断
	b. 干活
	c. 通知
3、创建多个线程，调用资源类的操作方法  [[Java_03_高级特性#创建thread的方式]]
4、防止虚假唤醒问题；

## Lock接口
java.util.concurrent.locks.Lock接口

较synchronized关键字，提供更丰富的功能（手动开关锁），当竞争激烈时，Lock接口远远优于synchronized关键字

| Lock                              | synchronized |
| --------------------------------- | ------------ |
| 不是Java内置的<br>util.concurrent中的一个类 | Java内置的关键字   |
| 手动上锁和释放锁                          | 自动上锁和释放锁     |
Lock接口中最常用的是这个实现类：[[#可重入锁 ReentrantLock]]

可重入锁  +  [[并发与并行#互斥锁]]
1、在 Field区域，new ReentrantLock对象
2、在方法前后，加锁和解锁
```
//Field
//创建可重入锁  
private final ReentrantLock lock = new ReentrantLock();

public void sale(){  
    //手动上锁  
    lock.lock();  
      
    try {  
        System.out.println(Thread.currentThread().getName()+" 卖出： "+(number--)+" 剩下： "+number);  
    }catch(Exception e) {  
        e.printStackTrace();  
        throw e;  
    }finally {  
        //手动解锁  
        lock.unlock();  
    }  
}
```

### 接口方法 @Override
实现Lock接口，需要重写以下6个方法
#### lock()
#### unlock()

#### tryLock()

#### tryLock(long time, TimeUnit unit)

#### lockInterruptibly()

#### newCondition()

## Condition接口
基于lock实例调用。==condition是用于线程通信的对象==。
联想到 Object类的方法：[[标准包.类#wait()]] 和 [[标准包.类#notify()]]

应用场景：
![[#^b67eb9]]

注意：一个condition对象，对应一个等待和唤醒条件！
不是对应线程！是对应条件！

condition可以实现：更灵活地控制线程的等待和唤醒，而不需要像使用 synchronized 时那样只能使用 wait() 和 notify()。 通过 Condition，我们可以实现更复杂的线程通信模式，例如生产者-消费者模型，读写锁等。
#### 常用方法
##### await()
等价于 [[标准包.类#wait()]]

1、使用await()和signal()必须包在锁块之间，否则报错 ！
2、wait()必须前，notify()必须在后，否则wait持续在那边，卡住！ ^f41349
##### signal()
等价于 [[标准包.类#notify()]]
##### signalAll()
等价于 [[标准包.类#notifyAll()]]

## 线程间通信
所谓线程间通信，就是：多个线程，按我们要求的顺序执行！一个wait，一个notify

[[#多线程步骤]] 中部：判断、干活、通知
```
public synchronized void sub() throws InterruptedException {  
    //判断  
    if (number != 1) {  
        this.wait();  
    } else {  
        //干活  
        number--;  
        System.out.println(Thread.currentThread().getName() + " :: " + number);  
        //通知  
        this.notifyAll();  
    }  
}
```

### 虚假唤醒问题
判断，始终应该加到**while循环中，而不是if判断中**！
wait()方法的特点，在哪里睡，就在哪里醒！就会跳过if判断，接着往下执行！

### 定制化通信
之前只能确保多个线程，我睡，你们醒的过程，现在更深层要求，按约定的顺序执行！

利用flag标志位 和 condition进行线程的自定义调度！
```
class ShareResource {  
    private int flag = 1;   //约定flag=1给AA线程，2给BB线程，3给CC线程  
  
    private Lock lock = new ReentrantLock();  

	//每个condition，负责处理一种等待条件
    private Condition c1 = lock.newCondition();   //c1负责flag是否等于1
  
    private  Condition c2 = lock.newCondition();  //c2负责flag是否等于2
  
    private Condition c3 = lock.newCondition();  //c3负责flag是否等于3

	// AA线程的方法，打印5次  
	public void printFive(int loop) throws InterruptedException {  
	    lock.lock();  
	    try {  
	        while(flag != 1){    //标志位判断  
	            c1.await();  
	        }  
	        for (int i = 0; i < 5; i++) {  
	            Sout(Thread.currentThread().getName()+" :: "+ i + " :轮数: "+ loop);  
	        }
	        this.flag = 2;   //修改标志位  
	        c2.signal();     //通知BB线程  
	    }catch(Exception e) {  
	        e.printStackTrace();  
	        throw e;  
	    }finally {  
	        lock.unlock();  
	    }  
	}
```

## 集合的线程安全
集合中，各种操作方法（例如ArrayList的add方法），并没有加上synchronized关键字，所以是线程不安全的。多线程同时写入会发送：==ConcurrentModificationException==  并发修改异常。
### [[Java_03_高级特性#【子类】CopyOnWriteArratList]]

### [[Java_03_高级特性#【子类】CopyOnWriteSet]]

### [[Java_03_高级特性#【子类】ConcurrentHashMap]]

## 多线程锁

### 锁的位置
![[Java_03_高级特性#^9c8a7c]]

### 公平锁与非公平锁
ReentrantLock中传递true参数，将默认的非公平锁改为公平锁。 ^7e8d64
```
new ReentrantLock(true);   // 传递参数true!   公平锁
// 默认是false,非公平锁
```

公平锁：部分线程饿死，效率高；
非公平锁：所有线程都参与，效率低

### 可重入锁 ReentrantLock
也称为：递归锁。
可重入性：一个线程进入一个方法，如果该方法嵌套了其他方法（恰好也加锁了），可以直接进入，不需要重新排队！

当前被加锁的方法内，一个线程进来，拿到锁，这个方法内部递归了一个方法，内部方法也被加了锁，可重入锁情况下，进入递归内部方法，这个线程不需要再申请另一把锁！

sychronized关键字，也称为（隐式）可重入锁！
Lock接口，是显式可重入锁（手动） ^a2d37a

ReentrantLock的可重入性的示例代码
```
Lock lock = new ReentrantLock();

lock.lock();
try {
	sout(sout(Thread.currentThread.getName() + "中层调用");
	// 如果没有可重入性，这里一定会死锁 -> 中层等待外层解锁！
	lock.lock();
	try{
		sout(sout(Thread.currentThread.getName() + "中层调用");
		
		lock.lock();
		try {
			sout(sout(Thread.currentThread.getName() + "内层调用");
		}finally {
			lock.unlock;
		}
	}finally {
		lock.unlock();
	}
}finally {
	lock.unlock;
}
```

#### 常用方法
##### lock()
获取锁
##### unlock()
释放锁

注意：上了几次锁，必须要释放几次锁！否则后面的线程拿不到！
这一点在redis分布式锁，实现可重入性的时候，是个难点 ^883699

##### lockInterruptibly()
获取锁，但可以被中断
##### tryLock(10，TimeUnit.MILLISECONDS)
尝试获取锁，等待10毫秒，获取不到则返回false
也可以传递等待时间的参数，在指定时间内获取不到就返回！


## Callable接口
复习：创建线程的四种方式  --> [[Java_03_高级特性#创建线程的四种方式]]
	a. 继承Thread类
	b. 实现Runnable接口（常用）
	c. ==实现Callable接口==
	d. 通过线程池方式

### Runnable接口与Callable接口对比

| Runnable接口 | Callable接口          |
| ---------- | ------------------- |
| 无返回值       | 有返回值 Future类型，可以抛异常 |
| lang包下     | concurrent包下        |

^bb7a38

### [[Java_03_高级特性#方式3：实现Callable接口（拓展）]]

## Future接口
jdk 1.5引入！
本接口定义了操作异步任务执行的一系列方法，例如：获取异步任务的执行结果、取消异步任务的执行、判断异步任务是否被取消、判断任务是否完成了等等。
	所谓异步，就是有些业务不仅仅要执行过程，还要获取执行结果，这时候就需要Future接口。Future接口就是封装异步方法的返回值！通过ExecutorService类的submit()方法提交任务，返回一个Future对象

Callable接口创建的线程，可以有返回值，这个返回值就被封装在Future类型中
	Runnable接口，无返回值；

如果主线程需要执行一个很耗时的任务，我们可以通过Future接口把这个任务放到一个单独的子任务异步执行它！

### 常用方法
#### boolean isDone()
反复询问，会耗费CPU资源

#### V get()
一旦调用，死心眼，非要等到结果才会离开，不管是否计算完成，都会==阻塞==！
泛型，获取异步计算结果
所以，get()一般都是放在程序后面再调用
#### 【重载】V get(long timeout, TimeUnit unit)
如果还没有获得结果，我再等一段时间！

### 【实现类】FutureTask类
1、实现了Runnable接口、Callable接口、RunnableFuture接口
2、不允许空参构造，构造函数要么传Runnable、要么传Callable！
联想到：[[Spring FrameWork#有参构造器 = 依赖注入]]
#### 构造函数
即实现了Future接口，又实现了Runnbale接口
##### FutureTask(Runnable runnable, V result)
##### FutureTask(`Callable<V>` callable)

![[Java_03_高级特性#^27e445]]

示例代码详见：[[Java_03_高级特性#方式3：实现Callable接口（拓展）]]

#### FutureTask的缺点
未完成的情况下，会阻塞

要获得异步结果，通常要用轮询的方式，尽量不要阻塞。

```
// 轮询的示例代码
// 联想到自旋等待  
while(true){  
    if(futureTask.isDone()){  
        System.out.println(futureTask.get());  
        break;  
    }else{  
        // 间隔500毫秒，旋一次，不要太频繁  
        try {  
            TimeUnit.MILLISECONDS.sleep(500);  
        } catch (InterruptedException e) {  
            throw new RuntimeException(e);  
        }  
        System.out.println("正在处理中，不要再催了，越催越慢");  
    }  
}
```

^21e932

## CompletableFuture接口
==是Future接口的升级版！==
1、异步结果。从轮询改为回调通知
2、可以将多个异步任务组合在一起。（两个线程相互独立，但是后一个依赖前一个结果）
3、一组异步任务中，返回最快完成的！

继承自：Future接口 和 CompletionStage接口
CompletionStage接口：代表异步计算过程中的某一个阶段，一个阶段完成后可能会触发另一个阶段

### 四个重要静态构造方法
不推荐使用new的方式创建对象，而是用静态方法
![[#^bb7a38]]

既发起多线程、又获取结果，都在一个里面！
#### runAsync 无返回值
static `CompletableFuture<Void>` runAsync(Runnable runnable)
如果不传递线程池，使用默认的线程池：ForkJoinPool.commonPool-worker-1

static `CompletableFuture<Void>` runAsync(Runnable runnable, Executor executor)
如果传一个线程池

Void对象，是一个Java内部定义的特殊对象，就是null ^bc52e1
#### supplyAsync(Supplier supplier) 有返回值
static `<T>` `CompletableFuture<T>` supplyAsync(`Supplier<T>` supplier)
[[Java_03_高级特性#Supplier -> get()]]

static `<T>` `CompletableFuture<T>` supplyAsync(`Supplier<T>` supplier, Executor executor)

利用多线程查询商城中特定商品的价格，[[Java_03_高级特性#流式计算 Stream]]
```
/**  
 * V2版本，每个netMall对象，放入一个多线程中，万箭齐发，最后异步把价格收集起来  
 * List<NetMall>  ---> List<CompletableFuture<String>  ---->  List<String>  
 * 每一个netMall，都生成一个异步线程，单独去查，收集成 List<CompletableFuture<String>  
 * 再用stream从异步任务的结果.join  
 */
 public static List<String> getPriceByCompletableFuture(List<NetMall> list, String productName){  
    return list.stream()  
            .map(netMall -> CompletableFuture.supplyAsync(() ->  
            String.format(productName + "in %s price is %.2f", netMall.getNetMallName(), netMall.calcPrice(productName))  
    ))  
            .collect(Collectors.toList())  
            .stream().map(s -> s.join()).collect(Collectors.toList());
            //第二次流式计算，从completableFuture对象中取结果  
}
```

### 常用方法

#### 获得结果和触发计算
##### 获得结果
```
CompletableFuture completableFuture = CompletableFuture.supplyAsync(()->{});

completableFuture.get();
```

###### V get() -> @Override Future接口
获取返回值，必须抛异常
###### V get(long timeout, TimeUnit unit) -> @Override  Future接口
###### T join()
这个方法，Future接口没有，是CompletableFuture接口独有的！
编译期间，不抛异常。
###### T getNow(T valueIfAbsent)
如果完成，返回结果；否则，返回一个指定值！valueIfAbsent是备胎值
##### 主动触发计算
###### boolean complete(T value)
打断分支线程的计算过程，直接拿value值返出去。
true  表示打断了
false  表示没打断

#### 对计算结果进行处理
##### thenApply(Function)
计算结果存在依赖关系，使两个结果串行化;

当前步骤错误，就不会走下一步！

```
CompletableFuture.supplyAsync(()->{
	TimeUnit.SECONDS.sleep(1);
	return 1;
}).thenApply(f -> {   // 基于上一步的结果，依赖上一步的结果，进行本次的计算
	return f+2;
}).whenComplete(v,e){
	if(e == null){
		sout(v)
	}.exceptionally(e -> {
		e.printStackTrace();
		return null;})
}
```
##### handle()
有异常也可以往下走，带着异常参数，进一步处理。

类似于：try catch finally

#### 对计算结果进行消费
接收任务的处理结果，并消费掉，无返回结果
##### thenAccept(Consumer)

| XX         | XX                          |                    |
| ---------- | --------------------------- | ------------------ |
| thenRun    | 任务A执行完执行B，并且B不需要A的结果        |                    |
| thenAccept | 任务A执行完执行B，B需要A的结果，但是任务B无返回值 | Consumer -> accept |
| thenApply  | 任务A执行完执行B，B需要A的结果，同时任务B有返回值 | Function -> apply  |

##### whenComplete(`BiConsumer<V, E>`)
这就是回调函数！一旦完成，自动触发此方法！
也能处理Exception
注意：不要用默认线程池，ForkJoinPool线程池被看做是守护线程，main一结束，它也会关掉，就拿不到异步结果了，一定要用自己的线程池

#### thenXxxAsync()
没有传入线程池，大家都用默认的ForkJoinPool
如果执行第一个supplyAsync()的时候传入了一个自定义的线程池，
	之后调用thenRun()，则第二个跟随第一个，都用自定义的
	如果调用thenRunAsync()，会另起炉灶，使用另一个线程池

#### 对计算速度进行选用
##### applyToEither
谁快用谁！
```
playA.applyToEither(playB, f -> f + "\t" + "is winner").join();
```
#### 对计算结果进行合并
##### thenCombine()
对两个分支的计算结果进行合并

```
completableFuture1.thenCombine(completableFuture2,FunctFunction)
```

## 并发工具类
辅助并发环境下的工具类！
### 【最常用】CountDownLatch 计数器
在多线程环境下，可以用来通过计数（等于线程池数量），等待所有线程完成，然后去获取每个线程执行的结果。
有点类似于放出去的猎鹰，然后挨个收线回来，从猎鹰手里获取猎物的过程！

1、该类构造器，需要传入一个int值作为初始值；2、其他线程调用countDown()方法，使计数器减1；3、await()方法阻塞线程，当计数器为0，唤醒被await()的线程
面向任务数，所以构造器中传递的参数是这个任务执行的次数
```
CountDownLatch (int count)   计数器的数！
```

多线程执行，拦着，不要着急走，等所有线程都返回结果了，你再走！
```
for (int i = 1; i <= threadNumber; i++) {  
    new Thread(()->{  
        //线程需要执行的逻辑  
        try {  
            for (int j = 1; j <= _1W; j++) {  
                clickNumber.clickBySynchronized();  
            }  
        } finally {  
            countDownLatch1.countDown();   // 50个线程，做完一个，减个1  
        }  
    },String.valueOf(i)).start();  
}  
countDownLatch1.await();                  // 拦在这里，不要着急进入下一步，等全部线程都完成了，才能走！
```

```
public static void main(String[] args) throws InterruptedException {  
    CountDownLatch countDownLatch = new CountDownLatch(6);  
  
    for (int i = 1; i <= 6 ; i++) {  
        new Thread(()->{  
            //线程需要执行的逻辑  
            System.out.println(Thread.currentThread().getName()+"号同学离开了教师");  
  
            countDownLatch.countDown(); //计数器-1  
  
        },String.valueOf(i)).start();  
    }  
    //等待  
    //注意：main方法本身就是一个线程！  
    countDownLatch.await();  
    System.out.println(Thread.currentThread().getName()+"班长锁门离开了");  
}
```

### CyclicBarrier 循环栅栏
循环栅栏，集齐7颗龙珠就可以召唤神龙🐲！
面向线程，所以构造器中传递的是线程数量
```
//构造函数
CyclicBarrier (int parties,Runnable barrierAction)

参数1：循环次数，触发时机   集齐7颗龙珠，就是7
参数2：Runnable接口创建线程
```

```
private static final int NUMBER = 7;  
  
public static void main(String[] args) {  
    CyclicBarrier cyclicBarrier = new CyclicBarrier(7,new Thread(()->{  
        //线程需要执行的逻辑  
        System.out.println("集齐7颗龙珠……召唤神龙");  
    }));  
  
    // 改成6，循环栅栏会一直等待  
    for (int i = 1; i <=7 ; i++) {  
        new Thread(()->{  
            //线程需要执行的逻辑  
            try {  
                System.out.println(Thread.currentThread().getName()+"星龙珠被收集到类");  
                cyclicBarrier.await();  
            } catch (Exception e) {  
                e.printStackTrace();  
            }  
        },String.valueOf(i)).start();  
    }  
}
```

### Semaphore 信号量
信号量，颁发许可
用来控制并发量
```
Semaphore(int permits)

//permits 表示颁发的许可数量，例如3个停车位就是3！
```
6辆汽车，停在3个停车位。

acquire()   从Semaphore获取到一个许可
release()    释放一个许可，返回给Semaphore

联想到：[[SpringCloud#SemaphoreBulkhead]]

## 读写锁 

### ReentrantReadWriteLock类
在可重入锁的基础上，提出更精细化的锁类型！

读锁：多人可以一起读，共享锁
写锁：只能一个人写，独占锁

| 无锁      | 添加锁                                    | 读写锁                                              |     |
| ------- | -------------------------------------- | ------------------------------------------------ | --- |
| 多线程抢占资源 | synchronized<br>ReentrantLock          | ReentranReadWriteLock                            |     |
| 乱       | 全部独占<br>每次只能一个人操作                      | 读可以共享<br>写只能一个人操作                                |     |
|         | 读读    只能1人<br>读写    只能1人<br>写写    只能1人 | 缺点：<br>a. 有可能一直读，写不了<br>b. 读的时候，不能写；<br>写的时候，可以读 |     |

#### 读写锁的降级 
1、在线程持有Read锁时：该线程不能获取Write锁。（不能把Read锁升级为Write锁，可能有其他线程也在读）

2、在线程持有Write锁时，该线程可以继续获取Read锁。（该线程持有Write锁，证明它一定持有ReadWrite锁，可以让它获取读锁。并且Write锁释放后，继续持有Read锁也没问题）

写锁降级为读锁
获取写锁 -->  获取读锁  -->  释放写锁  --> 释放读锁
（读锁不能升级为写锁，向下兼容）

## 阻塞队列 BlockingQueue接口
就是运用在多线程环境下，提供给线程等待的容器！用[[数据结构_队列]]这个数据结构实现！
在队列基础上，增加了两个附加操作：
1、支持阻塞的插入方法：队列满时，组织插入，直到队列不满时；[[#overload -->> offer(e, time, unit)]]
2、支持阻塞的移除方法：队列为空时，获取元素的线程会等待队列变为非空；[[#overload -->> poll(time, unit)]]

思考：基于上述描述，队列空时，消费者一直等待，当生产者添加元素时，消费者是如何知道队列有元素的？
回答：使用通知模式，[[#Condition接口]]]方式实现。 ^b67eb9

### 【实现类】ArrayBlockingQueue
特点：
1、基于数组Array实现的阻塞队列
2、维护一个==定长的数组==
3、构造函数必须传递一个int,也就是长度！
#### 常用方法
[[#BlockingQueue接口公用方法]]
##### add(E e)  -> @override Collection -> Queue -> BlockingQueue -> ArrayBlockingQueue

##### offer(e) -> @override Queue -> BlockingQueue -> ArrayBlockingQueue

##### overload -->> offer(e, time, unit)
##### remove()  -> @override Collection -> Queue -> BlockingQueue -> ArrayBlockingQueue
##### poll() -> @override Queue -> BlockingQueue -> ArrayBlockingQueue

##### overload -->> poll(time, unit) 
##### put(E e) -> @override BlockingQueue -> ArrayBlockingQueue

##### take() -> @override BlockingQueue -> ArrayBlockingQueue

### 【实现类】LinkedBlockingQueue
特点：
1、基于链表LinkedList实现的阻塞队列
2、长度默认是 ==integer.MAX_VALUE== （无限大）

## 线程池
Thread Pools
复习：创建线程的四种方式  --> [[Java_03_高级特性#创建线程的四种方式]]
	a. 继承Thread类
	b. 实现Runnable接口（常用）
	c. 实现Callable接口
	d. ==通过线程池方式==
联想到：[[JDBC#连接池]]

### 【❄️经典白雪 】默认线程池
基于Executors类，调用的静态方法
#### newFixedThreadPool(int)
固定数量的线程，超过数量，就要等待
底层用：[[#【实现类】LinkedBlockingQueue]]

联想到：[[SpringCloud#FixedThredPoolBulkhead]]
#### newSingleThreadExecutor()
只有一个线程的线程池

#### newCachedThreadPool()
可扩容的线程池

```
ExecutorService threadPool = Executors.newFixedThreadPool(5);

ExecutorService threadPool = Executors.newSingleThreadExecutor();

ExecutorService threadPool = Executors.newCachedThreadPool();

```

说明：Executors创建线程池对象的弊端：
1、newFixedThreadPool(int)和.newSingleThreadExecutor()，允许的请求队列，底层用[[#【实现类】LinkedBlockingQueue]]，固定队列长度是Integer.MAX_VALUE，容易造成OOM
2、newCachedThreadPool() 和 newScheduledThreadPool ，允许创建的线程数量为Integer.MAX_VALUE，容易造成OOM

#### newScheduledThreadPool()
专门用于执行 "延迟" 任务和 "周期性" 任务

使用 `schedule()` 方法来延迟执行一个任务，

也可以使用 `scheduleAtFixedRate()`  指定时间后，开始执行

或 `scheduleWithFixedDelay()` 方法来周期性地执行任务。

### Executor接口
线程池的接口！！

执行多线程任务的接口，将任务提交和执行解耦，根据不同策略制定规则。
![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202408082341334.png)
#### 接口方法
##### execute(Runnable)
开启线程池中的任务，只能执行Runnable接口的任务。==**无返回值**==。
```
Executor executor = Executors.newFixedThreadPool(10);

exectuor.execute(() -> {
	//执行任务的逻辑

})；

executor.shutdown
```

![[Java_03_高级特性#Runnable使用匿名实现类]]

#### ExecutorService接口
ExecutorService是子接口，主要接收Callable接口，有返回值！

##### 接口方法
###### submit(Callable)
接收Callable接口对象，并==**返回Future对象**==，里面封装了返回值！

![[Java_03_高级特性#Callable使用匿名实现类]]
##### 【实现类】ThreadPoolExecutor(7个参数)创建自定义线程池
是ExecutorService接口的实现类，自定义线程池！

开启了线程池，一定要记得关闭
```
ExecutorService threadPool = Executors.newFixedThreadPool(3);

try{
	
}catch (Exception 3){

}finally{
	threadPool.shutdown();
}
```

自定义线程池对象threadPool
	既可以执行execute(Runnable接口)方法，启动整个线程池！
	也可以执行submit(Callable接口)方法，(因为是ExecutorService接口的实现类)，启动线程池，并有返回对象Future

###### 线程池工作流程
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-05-26%20at%2021.18.49%402x.png)
###### 7个参数
ThreadPoolExecutor构造器源码
```
public ThreadPoolExecutor(int corePoolSize,                // 常驻\核心线程数量，这是池里的
                          int maximumPoolSize,               // 最大线程数量，借的
                          long keepAliveTime,                // 空闲线程等待工作的超时时间
                          TimeUnit unit,                     // 时间单位
                          BlockingQueue<Runnable> workQueue,   // 阻塞队列，存放等待线程
                          ThreadFactory threadFactory,         // 线程工厂
                          RejectedExecutionHandler handler) {……}  // 拒绝策略
```

先核心，再阻塞队列，最后再外借！（只有阻塞队列满了，才会开启最大线程数）
线程优先创建在：corePoolSize + BlockingQueue，放满时，再借用maxiumPoolSize
一般情况下，阻塞队列设置大一点，不太可能用到最大线程数！都会乖乖在阻塞队列里排队的！

提供一个模版，IDEA快捷键：newThreadPool
```
// 默认设置为系统可用的处理器数量  
int corePoolSize = Runtime.getRuntime().availableProcessors();  

// 默认设置为核心线程数的2倍  
int maxPoolSize = corePoolSize * 2;  

// 默认设置为60秒  
long keepAliveTime = 60L;  
TimeUnit unit = TimeUnit.SECONDS;  

// 使用LinkedBlockingQueue并设置容量为1024  
BlockingQueue<Runnable> workQueue = new LinkedBlockingQueue<>(1024); 

// 线程工厂，是默认
ThreadFactory threadFactory = Executors.defaultThreadFactory();  

// 拒绝策略：当线程池已满时,抛出RejectedExecutionException异常  
RejectedExecutionHandler handler = new ThreadPoolExecutor.AbortPolicy();  
  
ThreadPoolExecutor threadPool = new ThreadPoolExecutor(  
        corePoolSize,  
        maxPoolSize,  
        keepAliveTime,  
        unit,  
        workQueue,  
        threadFactory,  
        handler  
);  
  
threadPool.execute(()->{  
    //线程池中的线程需要执行的逻辑  
});
```


常驻线程数量用完，等待的线程会放入阻塞队列，等待空闲了进去。

思考：虽然提供了[[#【❄️经典白雪 】三种默认线程池]]，但是阿里开发手册约定，不允许使用Executors中的方法创建！为什么不用三种默认线程池（single、fixed、cached）
回答：因为这几个都使用的是无界阻塞队列，会导致队列中阻塞的线程越积越多，最终导致OOM！

拒绝策略，当线程达到最大数量，且阻塞队列也满时
1、AbortPolicy       抛异常
2、CallerRunsPolicy     谁叫你来的，你去找谁
3、DiscardPolicy            直接丢弃最新过来的
4、DiscardOldestPolicy   丢弃最早未处理的请求

线程池中核心线程数的配置规则
1、CPU密集型，线程数和CPU数相等；
2、I/O密集型，线程数等于2倍CPU数；

可以将线程数提取到[[SpringCloud#Consul 配置中心]]，当需要修改时，去配置中心修改即可。
###### 自定义线程池
需要通过ThreadPoolExecutor的方式，指定7个参数，自定义创建。

注意！
![[#^c83cbd]]

思考：new一个thread 和 threadPool启动的区别？

new thread需要调用start()方法，启动线程
new thread需要指定线程的名称
start()方法启动后，不需要手动关闭，因为run方法执行完，自动关闭
```
new Thread(()->{  
    //线程需要执行的逻辑  
},"线程名").start();  
```

threadPool需要执行execute()方法，启动整个线程池
threadPool中每个线程的名称默认是：pool-1-thread-N
需要调用shutdown手动关闭线程池
```
threadPool.execute(()->{  
    //线程需要执行的逻辑  
    System.out.println(Thread.currentThread().getName()+"在办理业务");  
});
```


四种拒绝策略
- AbortPlolicy（默认）：抛出异常RejectedExecutionException
- CallerRunsPolicy：调用者模式，谁让你来的，你找谁去
- DiscardOldestPolicy：抛弃队列中等待最久的任务，把当前任务加到队列中。（MD，有一个不讲理的）
- DiscardPolicy：默默丢弃无法处理的任务

###### 常用方法
- void execute(Runnable接口)

-  Future submit(Callable接口)
	开启线程池中的任务，可以执行Runnable接口和Callable接口类型的任务
	返回值是持有计算结果的Future对象。

 - shutdown()
	关闭线程池

示例代码
```
ThreadPoolExecutor executor = new ThreadPoolExecutor(参数1,2，3，4，5，6，7)；

//方式1:使用execute()方法提交一个Runnable任务，无返回值
executor.executor(runnableTask);


//方式2:使用submit()方法提交一个Callable任务，并返回Future对象，获取任务结果
Future<Result> future = executor.submit(callableTask);

executor.shutdowm
```

总结与思考：使用自定义线程池ThreadPoolExecutor提交任务后，怎么获取结果呢？
1、使用[[#Future接口]]或[[#CompletableFuture接口]]去接值
2、使用[[#【最常用】CountDownLatch 计数器]]知道当前任务的执行情况。
	例如发出去四个线程，回来一个，countDownLatch就计数一次，记数到4，就知道完成

## 分支合并框架 Fork/Join
Fork：将一个复杂任务进行拆分，大事化小
Join：把拆分的子任务，合并。

继承RecuriveTask类
在写下面代码的时候，为什么extends RecuriveTask类的时候，编译报错，必须要实现一个方法？不是接口才有这种现象吗！问了ChatGPT，恍然大悟，RecuiveTask是抽象类，也必须@Override其中的方法。
![[Java_02_面向对象#^ccbc10]]

```
class MyTask extends RecursiveTask<Integer> {  
  
    @Override  
    protected Integer compute() {  
        //task01.fork()  
        //task02.fork()        
        // result = task01.join() + task02.join()        
        return null;  
    }  
}

public class ForkJoinDemo {  
    public static void main(String[] args) throws Exception {  
        MyTask myTask = new MyTask();  
  
        // 创建分支合并池对象  
        ForkJoinPool forkJoinPool = new ForkJoinPool();  
        ForkJoinTask<Integer> forkJoinTask = forkJoinPool.submit(myTask);  
          
        //获取结果  
        Integer result = forkJoinTask.get();  
        System.out.println(result);  
    }  
}
```
## LockSupport类
是一个==**线程阻塞的工具类**==

1、不用同步代码块，也不用lock() + unlock()
可以在任意位置调用park，使线程发生阻塞！然后再另一个线程发放unpark许可证perimit

2、LockSupport类，使用一种permit（许可证）的概念来阻塞和唤醒线程的功能。每个线程都有一个许可（permit）
注意，与[[#Semaphore 信号量]]不同，permit不能累加！
permit许可证默认是无，所以一开始调用park方法，当前线程就会被阻塞，直到别的线程给当前线程发放（unpark）permit，park方法才会被唤醒。（这里顺序上，和前面两种有点区别）

**undefined对wait()和notify() 和 await()和signal()的改进。**
![[标准包.类#^06143b]]
![[#^f41349]]

park() 和 unpark() 可以突破上面两种的限制
1、无锁块要求
2、允许先唤醒，后等待
3、成双成对要牢记！
4、unpark方法无论调用多少次，都只算发一次permit；掉一次park就阻塞一次！

思考：为什么park()和unpark()可以突破调用顺序？
回答：生产一张permit，发放出去，我无视调用顺序。只要发出去了许可证，不管什么时候来，你带着这个permit，我消费即可。

### 常用方法
所有方法都是静态方法！
#### static void park()
阻塞线程
没有许可证，就不允许当前线程进行线程调度。
联想到：[[标准包.类#wait() -> @Overried Object类]] 、[[#await()]]

#### static void unpark(thread)
传递一个线程，给哪个线程发放permit许可证。解除被阻塞的线程

联想到：[[标准包.类#notify() -> @Override Object类]]  [[#signal()]]

### 没有返回值的异步调用 runAsync(Runnable)

```
CompletableFuture<Void> completableFuture = ComplatableFuture.runAsync(()->{});  
```
runAsync需要传递一个Runnable接口

### 有返回值的异步调用 supplyAsync(Supplier)
[[Java_03_高级特性#Supplier -> get()]]

获取结果
whenComplete(Consumer).get()
```
CompletableFuture<Void> completableFuture = ComplatableFuture.supplyAsync(()->{return something});  

completableFuture.whenComplete((t,u)->{}).get();
```

## CAS 自旋锁
Compare and Swap（比较并交换）
是一种多线程并发编程中的原子操作。[[并发与并行#乐观锁]]，

CAS本质是乐观锁思想的贯彻：我乐观的认为我拿走的时候是这个值，修改完写回去的时候还是这个值

包含三个操作数：
	内存位置V
	预期原值A：从主内存拷贝到线程中的副本拷贝！
	更新值B
执行CAS操作的时候，将内存位置的值与预期原值进行比较：
- 如果==相匹配==，那么处理器会自动将该位置值更新为更新值
- 如果==不匹配==，处理器不做操作，多个线程同时执行CAS操作==只会有一个成功==。

```
public class CASdemo {  
    public static void main(String[] args) {  
        AtomicInteger atomicInteger = new AtomicInteger(5);  // 内存位置的值  
        // expect:旧的期望值   update:需要更新的值  
        // 对比expect如果等于initialValue，就把它改为2022  
        System.out.println(atomicInteger.compareAndSet(5, 2022) + "\t" + atomicInteger.get());  
    }  
}
```

### 【核心实现类】UnSafe类
`jdk/jre/lib/rt.jar/sun/misc/Unsafe.class`
工作中不要用这个类，不安全的！

CAS不是通过加锁保证线程安全的，而是利用硬件保证原子性。不需要用户态和内核态的切换。
底层是CPU指令（cmpxchg指令：比较并交换的操作）

原子类.compareAndSet()  -->  ==unsafe==.compareAndSwapInt() -->  ==native.==compareAndSwapInt()


### 【实现类】`AtomicReference<V>`
自定义，引用类型的原子类型！

示例代码
```
public class AtomicReferenceDemo {  
    public static void main(String[] args) {  
        AtomicReference<User> atomicReference = new AtomicReference<>();  
        User z3 = new User("z3",22);  
        User l4 = new User("l4",28);  
  
        // 张三这个User实例对象，就具备了原子性  
        atomicReference.set(z3);  
        System.out.println(atomicReference.compareAndSet(z3, l4)+"\t"+atomicReference.get());  
    }  
}
```

### 自旋锁
spin lock
CAS是实现自旋锁的基础。当[[#boolean compareAndSet(expect, update)]]返回true的时候，跳出循环，false的话就无限循环。
![[Java_01_基础#^094fff]]

redis实现分布式锁的时候
[[redis#V4：自旋替代递归]]
[[redis#WATCH key]]是CAS的具体应用。监视的key发现被动过，整个MULTI……EXEC的事务全部失败！

### CAS的缺点
1、ABA问题：A变成B再变成A，CAS会认为没有变化，有些业务场景不容忍这样的情况（引入版本号，戳记流失 --> [[#【实现类】AtomicStampedReference]]）
2、只支持单个变量的操作；
3、循环时间长，CPU开销大

## 原子类

天生具备原子性，不需要加synchronized这样的重锁！
原子类是对CAS 自旋锁机制的落地实现。可以避免传统的加锁机制带来的性能损耗和线程阻塞。

将一些复合操作，包装成一个原子操作，在高并发多线程环境下，确保线程安全！
### 通用方法
#### boolean compareAndSet(expect, update)
比较并交换！就是CAS
#### get
获取当前的值
#### getAndSet(newValue)
获取当前值，并设置新值
#### getAndIncrement()
获取当前值，并自增。原子版的 `i++`
#### getAndDecrrement()
获取当前值，并自减。原子版 i--
#### getAndAdd(delta)
获取当前的值，并加上预期的值
#### incrementAndGet()
自增1，并返回自增后的结果
是原子性的 i++，线程安全的！

### [[Java_01_基础#基元类型]]原子类

#### 【实现类】AtomicInteger
具有原子性操作的int基本数据类型

#### 【实现类】AtomicLong
具有原子性操作的long基本数据类型
推荐使用：[[#【实现类】LongAdder]]

#### 【实现类】AtomicBoolean
具有原子性操作的boolean基本数据类型
![[标准包.类#^eccb16]]
### [[Java_01_基础#数组]]类型原子类
#### 【实现类】AtomicIntegerArray
构造方法，必须指定长度，没有空参构造方法
#### 【实现类】AtomicLongArray
#### 【实现类】AtomicReferenceArray

### [[Java_01_基础#引用类型]]原子类

#### 【实现类】AtomicReference

#### 【实现类】AtomicStampedReference
带有戳记流水的引用类！解决CAS缺点的ABA问题

构造方法必须传两个参数：一个对象，一个初始戳记
![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202408012244274.png)

```
Book javaBook = new Book(1,"javaBook");  
AtomicStampedReference<Book> stampedReference = new AtomicStampedReference(javaBook,1);  
System.out.println(stampedReference.getReference()+"\t"+stampedReference.getStamp());  
Book mysqlBook = new Book(2,"mysqlBook");
```

#### 【实现类】AtomicMarkableReference
将戳记流水自增的1-2-3-4，简化为：动过-没动过！
```
AtomicMarkableReference markableReference = new AtomicMarkableReference(100,false);

boolean marked = markableReference.isMarked();
```
- 构造方法，传递initRef：需要改的值    initMark：标志位的值
- isMarked()    返回当前的标志位的值

### 对象的属性修改原子类
不是修改对象，而是修改对象中的属性值的！              **局部麻醉！**
更加细粒度的修改！

以一种线程安全的方式操作非线程安全对象内的字段。
![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202408020020048.png)

前提条件
1、对象的属性必须使用：public volatile
2、每次使用必须使用静态方法newUpdater()创建一个更新起，并且需要设置需要更新的

```
AtomicIntegerFieldUpdater<BankAccount> fieldUpdater = AtomicIntegerFieldUpdater.newUpdater(BankAccount.class,"money");  
public void transMoney(BankAccount bankAccount){  
    fieldUpdater.getAndIncrement(bankAccount);  
}
```
#### 【实现类】AtomicIntegerFieldUpdater
用volatile修饰的int字段（volatile :可见性、有序性）

#### 【实现类】AtomicLongFieldUpdater
用volatile修饰的long字段（volatile :可见性、有序性）

#### 【实现类】AtomicReferenceFieldUpdater
用volatile修饰的引用类型字段（volatile :可见性、有序性）


### 原子操作增强类
继承自Striped64类，比AtomicLong等原子类，在高并发环境下，性能更好！低并发下，差不多！

LongAdder为什么这么快呢？  [[标准包.类#Striped64类]]
核心思路：（高并发情况下，CAS自旋很恐怖）必须规避掉CAS自旋的消耗！分散热点，拆分成Cell类！
分散热点，将value的值分散在Cell数组中，不同线程命中不同的Cell数组中的槽位，各个线程只对自己槽位中的那个值进行CAS自旋，这样热点就分散了！最后累加结果，只要把各个槽位的数据相加即可。
![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202408021055232.png)

#### 常用方法
##### add(long x)
将当前值value加x
##### increment()
将当前值value加1
##### decrement()
将当前值value减1
##### sum()
返回当前值，特别注意，在没有并发更新value的情况下，sum会返回一个精确值，在存在并发情况下，sum不保证返回精确值
##### reset()
将value重置为0，可用于替代重写new一个LongAdder，但此方法只能在没有并发更新的情况下使用
##### sumThenReset()
获取当前value，并将value重置为0

#### 【实现类】LongAdder
LongAdder 只能计算加法，且只能从零开始计算

推荐使用LongAdder对象，比AtomicLong性能更好（减少乐观锁的自旋重试次数） #Java编程规约 
```
AtomicInteger count = new AtomicInteger();
count.addAndGet(1);
```

适用场景：热门商品点赞计数器！
##### 常用方法
###### increment()
按1递增
###### sum()
获取累加结果
#### 【实现类】LongAccumulator
LongAccumulator 提供了自定义的函数操作

==构造方法必须传两个参数==：
1、LongBinaryOperator  函数式接口  -> @Override applyAsLong(left , right)
2、long identity              初始化值
```
// identity就是x,accumulate传的1就是y  
LongAccumulator longAccumulator = new LongAccumulator((x,y)->{return x+y;}, 0 );  
longAccumulator.accumulate(1);
```
##### 常用方法
###### accumulate(long x)
按定义的函数，累加！
###### get()
获得累加结果

对比：synchronized AtomicLong LongAdder LongAccumulator 的性能，相差10倍
```
----costTime: 4936 毫秒	 clickBySynchronized: 50000000
----costTime: 765 毫秒	 clickByAtomicLong: 50000000
----costTime: 121 毫秒	 clickByLongAdder: 50000000
----costTime: 96 毫秒	 clickByLongAccumulator: 50000000
```

#### 【实现类】DoubleAdder

#### 【实现类】DoubleAccumulator



## `ThreadLocal<T>`类
线程局部变量！是类中的私有静态字段！（private static)

其他线程无法访问到该数据！每个线程对这个值的修改都是独立的！

最重要的注意点
![[#^c83cbd]]

### 基本概念
联想到[[JVM#JMM内存模型]] [[计算机操作系统#内存模型]]
如下图：共享变量的副本，就是ThreadLocal!  
![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202407312130904.png)

#### ThreadLocalMap静态内部类
[[标准包.类#Thread类]] 和 [[标准包.类#ThreadLocal类]]  和 ThreadLocalMap类的关联和区别？
- 每个Thread类的实例对象里都有一个Field：ThreadLocal（体现了每个线程独一份）
- 每个ThreadLocal类，有一个[[Java_02_面向对象#静态内部类]]，叫：ThreadLocalMap
![[Java_02_面向对象#^876f5b]]

线程内部的共享变量副本（ThreadLocal)的内部结构：
![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202408011323746.png)

Thread -> ThreadLocal -> ThreadLocalMap -> Entry对象（弱引用）
当我们操作threadLocal实例赋值的时候，实际上是以当前threadLocal实例对象为key，值为value，往Entry里面存放。

每一个线程类实例，里面都存在一个Entry对象
	Key：ThreadLocal对象
	Value：要存的值

#### ThreadLocal内存泄漏问题
ThreadLocalMap中的Entry继承了WeakReference

[[JVM#强、软、弱、虚引用]]  -> 弱引用：一发生GC，就立刻被回收掉（不管内存够不够）。太适合ThreadLocal了！
[[JVM#内存泄漏]]

Thread -> ThreadLocal --WeakReference包装--> ThreadLocalMap -> Entry对象（弱引用）

为什么Entry要继承WeakReference？
当方法执行完毕，方法中变量对threadLocal对象的引用就断了，实际上这个threadLocal我们已经不需要的，但是在ThreadLocalMap中的key也是指向threadLocal对象的，就容易造成内存泄漏，始终GC不掉！

如果Entry的key被设为null（GC），还有一个坑！一旦Entrt的key=null，就没有办法访问这些key和value，value对值的引用，会造成内存泄漏！
因此，弱引用（WeakReference）不能100%杜绝内存泄漏，因此才需要我们在不使用某个threadLocal对象的时候，手动调用remove()方法，目的就是清理Entry中key=null，value还有值的情况！

set、get、remove方法都会走到这个方法 expungeStaleEntry()   清理废旧的Entry（key=null的那些）！

###  常用方法
#### set(T value)
将一个变量设置到ThreadLocal副本中，键值对存储。
key：ThreadLocal对象
value：传进来的参数。

每个threadLocal对象只能放一次value！如果要放多种数据，可以new多个threadLocal对象实例，往里面放数据。

```
ThreadLocal<Integer> intValue = new ThreadLocal<>();
ThreadLocal<String> strValue = new ThreadLocal<>();

intValue.set(10);
strValue.set("abc");

int value1 = intValue.get();
String value2 = strValue.get();
```

#### T get()
在当前线程中，利用当前线程的threadLocal对象，获取值！

#### remove()
注意：移除这个值，避免内存泄漏。
不要给一个线程携带太多的东西，用完了及时释放掉。

==注意==：必须回收（remove）自定义的ThreadLocal变量，尤其是在线程池场景下，线程经常会被复用，如果不清理自定义的ThreadLocal变量，会影响后续业务逻辑和造成内存泄漏等问题，推荐使用try-finally块回收。 #Java编程规约 ^c83cbd
```
objectThreadLocal.set(userInfo);

try{
	//
}finally{
	objectThreadLocal.remove();
}
```

每个线程都有一个ThreadLocalMap，其中key是弱引用，但是其中Value与值是强引用，如果线程存活时间长，但是ThreadLocal不用了，强引用是GC不了的，就会发生内存泄漏！

补充一个使用场景，便于理解：JDBC中手写连接池，利用线程本地变量，存储连接信息Connection，确保一个线程中多个方法可以获取同一个connection。
![[JDBC#^ba30d5]]

#### ~~initialValue()~~
设置初始值，否则是null，容易报空指针异常

不推荐使用这个方式！
```
ThreadLocal<Integer> saleVolume = new ThreadLocal<Integer>(){  
    @Override  
    protected Integer initialValue() {  
        return 0;  
    }  
};
```

#### static withInitial(Supplier)
推荐此方法进行ThreadLocal初始化！一定要设置初始值，否则是null，容易报空指针异常
```
ThreadLocal<Integer> saleVolume = ThreadLocal.withInitial(()->0);
```

### 使用场景
最佳实践：
1、一定要用withInitial方法，设定一个初始化值，否则return null，容易报空指针异常
2、建议加上static，一般只需要初始化一次就够了，只分配一块内存空间。
3、**==用完一定要remove()==**！

一个线程的执行过程，包含多个类（Controller -> Service层），需要大量数据传递，全部用参数传递不方便，就可以使用线程私有的ThreadLocal传递！
例如：同一个线程中，Controller和Service层，传递User对象！不同线程访问，对应不同的User，同一个ThreadLocal，能拿到各自向其中存放的值。
![CleanShot 2024-07-10 at 11.33.34.png](https://raw.githubusercontent.com/rockyshen/blog-img/master/CleanShot%202024-07-10%20at%2011.33.34.png)


## AQS 抽象队列同步器
java.util.concurrent.locks.AbstractOwnableSynchronizer

源码位置：IDEA/External Libaries/java/util/concurrent/locks/AbstractQueuedSynchronizer.java
是JUC内容中最重要的基石：

Abstract Queued Synchronizer   抽象的队列同步器
继承自：AbstactOwnableSynchronizer类

[[设计模式#Template Method 模版设计模式]]
Queue：一个队列，提供给抢不到锁的兄弟，等待的地方！

是用来实现锁（或其他同步器组件）的公共基础部分的抽象实现。
主要解决：锁分配给“谁”的问题！（锁分配问题）

整体就是一个抽象的FIFO队列来完成：
	1、资源获取线程的排队工作；
	2、并通过一个int类变量标志持有锁的状态。

CLH队列（Craig、Landin、Hagersten三位计算机科学家）
AQS中的队列是CLH队列的升级，双向链表 + 状态位

![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202408021258822.jpg)
### 锁与同步器的关系
进一步理解锁和同步器的关系

锁：是面向程序员，面向使用者的

同步器：是面向锁的实现者，需要明确锁应该具备哪些功能：
	1、同步状态管理；                         现在这把锁，有人在用啊！
	2、同步队列的管理和维护；         你在队列里等一等，到了我叫你
	3、阻塞线程排队和通知；             都在这里老老实实排队，到了我会叫你们
	4、唤醒机制                                   那个线程B，到你了，去拿锁吧

AQS内部封装了一个Node静态内部类，存放

AQS源代码
![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202408021256544.png)

![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202408021323576.jpg)
### state 状态位
AQS类的成员变量，标识状态位，0标识没人占用锁；1标识有人占用锁！

### Node静态内部类
每个被阻塞的线程  就是一个节点。以Node对象的方式，放入CLH队列。

#### waitStatus

| 属性         | XX                                                |
| ---------- | ------------------------------------------------- |
| SHARED     | 共享型节点                                             |
| EXCLUSIVE  | 独占型节点                                             |
| CANCELLED  | 1   不想排队了，取消等待                                    |
| SIGNAL     | -1   waitstatus  -> thread need unparking         |
| CONDITION  | -2   waitstatus  -> thread is waiting on conditon |
| PROPAGATE  | -3   传播，广播                                        |
| waitStatus | 区别于AQS的int status<br>Node的等待状态，初始是0，状态就是上面大写的几种   |
| prev       | 前节点                                               |
| next       | 后节点                                               |

### CLH队列
每个需要等待的Node节点，以双链表的形式加入CLH队列。

********************
2024.8.2 终于趁着小姑娘和宝宝去威海旅行的一周时间，把@阳哥的JUC高级课程完成整体框架性的学习，其中还有原子类LongAdder的源码分析（P92-98）和AQS（P146-155）的源码分析暂时搁置，等后面有需求了，再回过来学习即可（整体框架和理论我已经清楚，应付面试足矣！

源码分析，待补充！

至此，可以说Java语言的完整的版图，基本上到今天算是覆盖全面了！Flag🚩，里程碑记录之！现在就是加油修改完善面试回答！准备模拟面试，加油加油！
