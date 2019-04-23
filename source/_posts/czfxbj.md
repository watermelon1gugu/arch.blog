---
title: 春招复习笔记
date: 2019-04-23 11:33:03
tags: 春招
---

## HTTP

> 超文本传输协议，是一个基于请求与响应，无状态的，应用层的协议，常基于TCP/IP协议传输数据，互联网上应用最为广泛的一种网络协议,所有的WWW文件都必须遵守这个标准。**设计HTTP的初衷是为了提供一种发布和接收HTML页面的方法。**

## HTTP设置缓存

Cache-Control:在响应头中设置，用于通知浏览器该资源需要被缓存。

![](https://upload-images.jianshu.io/upload_images/6383185-0fbb4b92637d7e61..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/878/format/webp)

## HTTPS

HTTPS是在HTTP协议的基础上，增加了保密措施的一种协议。所以其主要作用是保证通信的安全，主要解决：

- 防止第三方冒充服务器
- 防止第三方拦截通信报文，窃取通信中请求报文，响应报文的内容
- 防止第三方拦截通信报文，篡改报文内容

### 加密内容

1. 对称加密算法：加密、解密用同一个秘钥，速度快。
2. 非对称加密算法：加密、解密用不同的秘钥。公私钥对。私钥加密的，所有公钥都可以解密，公钥加密的，只有私钥可以解密。
3. Hash算法：一种单向加密，理论上不可解密

HTTPS将三种都利用上了

### HTTP通信过程

- 客户端向服务端发起请求
  1. 此时请求报文可能被截获，泄露请求信息
  2. 请求可能被截获，并冒充服务器给其响应
- 服务端向客户端发起请求
  1. 请求报文可能被截获，泄露响应信息

综上所述 在HTTP通信工程中，应该将request和response加密

1. 对称加密，需要客户端和服务器用相同的秘钥加密、解密。主要问题是服务端需要管理海量的秘钥。
2. 非对称加密：加密请求报文时，没什么问题。响应时，由于加密是服务器利用私钥加密的，所以所有拥有公钥的客户端都可以解密，这样就可能被其他要公钥的客户端窃取报文。并且非对称算法，速度比较慢。
3. hash：算法不可解密，传过去没什么用

由于非对称算法，所有公钥客户端都可以解密响应报文，其不能够被选择，HASH更不行，只有对称算法可以考虑。需要解决秘钥同步问题。

### 给对称算法的秘钥加密

HTTPS用非对称算法进行加密（实际上，只是加密了生成这个秘钥的一部分原料）。

1. 在服务端保存这样一个秘钥
2. 在所有的客户端保存着服务器的秘钥对应的公钥。
3. 这个（对称算法)秘钥由**客户端**生成，用公钥加密，传给服务器，这样就只有服务器有秘钥可以解密，即使被拦截，也无法解密。

总结：服务器保存一个非对称公私钥，并讲公钥发给客户端，客户端用公钥给客户端生成的对称秘钥加密，并发给服务端，只有服务端有私钥可以解密。这样双方都有了对称秘钥。

那么，如何防止第三方冒充目标服务器，黑客发给你一个他的公钥，之后的加密就没有意义了。

### 证书

服务端为了向客户端证明自己，会给客户端发一个凭证，这里的这个凭证就是证书。这个证书不仅需要证明自己的身份，并且还必须防伪造。

这里一般是服务器向第三方颁发证书的机构申请一个证明自己的证书，利用这个证书证明自己的身份，并且客户端会根据这个颁证的机构的算法去进行防伪验证，看看证书是否伪造，如果不是伪造的，则没有问题。

防伪算法

- 颁发证书时，会根据申请方的信息用hash类的加密算法生成**摘要**信息，客户端也用这个算法生成摘要，与其比较
- 并对这个摘要进行非对称加密，防止对其篡改
- 将加密后的摘要信息、生成摘要的加密算法、和证书的基本信息放在一起，作为证书颁发给申请方，
- 客户端利用颁证机构的公钥，对其加密后的摘要进行解密。

这里的客户端的公钥是怎么来的呢，由于颁证机构并不多，操作系统里对一些信用良好的机构进行了信任处理，将其公钥事先安装好了。

ps:每个服务摘要算法不一样 客户端用预装公钥解密证书信息后得知摘要算法 并hash出客户端摘要进行比对

### 认证流程

- 服务提供者向办证机构申请证书。
- 将证书发布到web服务中
- 客户端向服务端发起申请，并发起一个随机数a。
- 服务端将证书返回客户端证明自己，并发会一个随机数b。
- 客户端根据证书的颁发机构，拿到预装的公钥，对其摘要进行解密
- 客户端根据证书中的摘要算法，进行客户端的摘要计算，并将计算结果和解密后的摘要进行比较
- 如果不一致，说明证书是伪造或被篡改过，立即停止通信
- 如果一致，则生成第三个随机数c，并用a、b、c生成对称加密算法的密钥，并用证书中的公钥（对应服务器的密钥）对c进行加密，然后将c发给服务端
- 服务端用密钥将c解密，并用a、b、c生成对称加密算法的密钥。
- 之后的通信将报文进行签名，并与报文一起进行对称加密（防止他人恶意篡改信息，以试错方式进行密钥的破解，以增加破解难度）

## 浏览器输入url后经历了什么

1. DNS域名解析；

   浏览器首先确认域名对应的服务器在哪，将域名解析成服务器IP地址这项工作由DNS服务器来完成。客户端收到你输入的域名地址后，它首先去找本地的hosts文件，检查在该文件中是否有相应的域名、IP对应关系，如果有，则向其IP地址发送请求，如果没有，再去找DNS服务器。一般用户很少去编辑修改hosts文件。

   ![](https://upload-images.jianshu.io/upload_images/5996435-b926543a4358a161.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/805/format/webp)

   从客户端到本地服务器属于**递归查询**，而DNS服务器之间的交互属于**迭代查询**。

   正常情况下，本地DNS服务器的缓存中已有comDNS服务器的地址，因此请求根域名服务器这一步不是必需的。

   ps:获得IP后用ARP  用MAC找到IP

2. 建立TCP连接；

   三次握手 

   - 客户端先发送一个带有SYN标志的数据包到服务端，服务端接收到后

   - 回传一个 SYN/ACK标志的数据包表示收到

   - 客户端再回一个ACK

     ACK表示收到，SYN表示第一次发送，确认sequence numer

   ![](https://upload-images.jianshu.io/upload_images/5996435-1ac4f67e1f2a2115.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/592/format/webp)

3. 发送HTTP请求；

4. 服务器处理请求；

5. 返回响应结果；

6. 关闭TCP连接；

7. 浏览器解析HTML；

8. 浏览器布局渲染；

## Bean的生命周期

bean创建---初始化---销毁的过程

由容器管理bean的生命周期

我们可以自定义初始化和销毁方法，容器在bean执行到当前生命周期的时候来调用我们的自定义初始化销毁方法。



1.指定初始化销毁方法：

```java
@Bean(initMethod = "init",destroyMethod = "destroy")//在此注册，在Car类中写入方法
    public Car car(){
        return new Car();
    }

@Component
public class Car {
    public Car(){
        System.out.println("car constructor");
    }
    public void init(){
        System.out.println("car ... init ...");
    }
    public void destroy(){
        System.out.println("car ... destory ...");
    }
}
```



创建容器 

```java
AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
//容器创建完成时，所有单实例bean都会被创建出来
//容器关闭时，都销毁
```

构造(对象创建)

​	单实例，在容器启动的时候创建对象

​	多实例，在每次获取的时候创建对象

初始化：

​	对象创建完成，并赋值好，调用初始化方法。。。

1. 通过@Bean指定init-method 和 destory-method

2. 通过让Bean实现InitializingBean(定义初始化逻辑)，

   ​	DisposableBean(销毁逻辑)

3. 使用JSR250中的注解

   @PostConstruct，在bean创建完成，并且属性赋值完成，来执行初始化

   @PreDestory，在bean在容器中移除销毁之前来调用

4. BeanPostProcessor：bean的后置处理器

   在bean的初始化前后进行处理工作：

   postProcessorBeforeInitialization 在初始化调用之前工作

   postProcessorAfterInitialization 在的初始化调用之后工作

```java
//实现DisposableBean, InitializingBean
public class Cat implements DisposableBean, InitializingBean {	
    public Cat(){
        System.out.println("Cat constructor");
    }
    public void destroy() throws Exception {
        System.out.println("cat...destroy");
    } 
    public void afterPropertiesSet() throws Exception {
        System.out.println("cat...init");
    }
}
```

```java
@Component //后置处理器
public class MyBeanPostProcesser implements BeanPostProcessor {
    //参数 bean beanName
    public Object postProcessBeforeInitialization(Object o, String s) throws BeansException {
        System.out.println("postProcessBeforeInitialization"+"=>"+s);
        return o;
    }

    public Object postProcessAfterInitialization(Object o, String s) throws BeansException {
        System.out.println("postProcessAfterInitialization"+"=>"+s);
        return o;
    }
}

```



销毁：

​	单实例，容器关闭时销毁。

​	多实例，容器只负责创建，不管理，关闭不销毁

```java
@Scope("prototype")//添加@Scope("prototype") 是多实例，需要再创建 并且销毁时间取决于你 容器不管
```

```java
applicationContext.getBean("car");
```

### beanPostProcesser原理

populateBean(beanName,mbd,instanceWrapper);给bean进行属性赋值

initializeBean{

applyBeanPostProcessorBeforeInitialization(wrappedBean,beanName);//调用前置处理器

invokeInitMethods(beanName,wrappedBean,mbd);//执行自定义初始化方法

applyBeanPostProcessoorsAfterInitialization(wrappedBean,beanName);//调用后置处理器

}

### Spring底层对BeanPostProcessor的使用

ApplicationContextAware 

```java
public class Dog implements ApplicationContextAware {
    private ApplicationContext applicationContext;
     @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;//传入容器
    }
}
```

在创建Dog对象前，利用BeanPostProcessor用反射instanceof 判断是否实现ApplicationContextAware或者其他Aware接口

如果是，调用invokeAwareInterface给bean注入值{

判断实现的是哪个aware ，比如是ApplicationContextAware，就会调用bean的setApplicationContext方法 将IOC容器注入

```java
((ApplicationContextAware)bean).setApplicationContext(this.application);
```

}

**注解功能的实现靠BeanPostProcessor解析注解**

bean赋值 注入其他主键 @Autowired，生命周期注解功能 @Async,xxx 都靠BeanPostProcessor实现

## 数组中第K大元素

### 快排

```C++
int getK(int* nums,int K,int left,int right){
    if(left>=right)return -1;
    int count = left;
    int key = nums[left];
    swap(nums[left],nums[right]);
    for(int i = left;i<right;i++){
        if(nums[i]<key){
            continue;
        }else{
            swap(nums[i],nums[count]);
            count++;
        }
    }
    swap(nums[right],nums[count]);
    if(count+1==K){
        return nums[count];
    }else if(count +1 <K){
        return getK(nums,K,count+1,right);
    } else if (count +1 >K){
        return getK(nums,K,left,count-1);
    }
}
```



### 构建容量为K的小顶堆

## HashMap和HashTable和ConcurrentHashmap

**关注点**:

- Map类取代了旧的抽象类Dictionary 拥有更好的性能
- 没有重复的Key，可以有重复的Value
- Value可以是List、Map、Set类对象
- KV是否允许为null，以实现类约束为准

![hashmap](https://user-images.githubusercontent.com/25349066/53565586-d2d6c580-3b94-11e9-97fa-849f3b68819f.png)

**只有HashMap KV可以为null**  TreeMap的Value可以为null  其他都不可以 TreeMap基于红黑树

TreeMap是有序的集合 性能没有HashMap和ConcurrenHashMap高 和他们都继承于AbstractMap

HashMap是依靠hashCode和equals实现去重，TreeMap依靠Comparable或者Coomparator实现去重

HashMap结构：slot槽 table hash桶

![hash struct](https://user-images.githubusercontent.com/25349066/53566610-a1132e00-3b97-11e9-8b7b-ad91954e92dd.png)

hashmap的问题：数据迁移时丢失，死链(两个线程运行到同一处)

两个线程 AB执行 transfer 方法 虽然 newTable 是局部变量 但是原先
table 中的 Entry 链表是共享的。产生问题的根源是 Entry next 被并发修改。这可能
导致
(I)对象丢失。
(2)两个对象互链。
(3)对象自己互链 

hashtable 是表锁

## CAS

compare and swap 拿预期值（希望内存中现在的值，因为当前操作基于该值）去比较，比较成功后置换  整个操作原子

ABA问题：两次cas操作后 预期值和原先一样 其实已经改变过 解决 Atomic类中维护状态戳

## JAVA并发包JUC

### 1.线程同步类

这些类使线程间的协调更加容易，支持了更加丰富的的线程协调场景，逐步跳台了wait()和notify()进行同步的方法，主要代表为 CountDownLatch、Semaphore信号量、CyclicBarrier.

### 2.并发集合类

ConcurrentHashMap、CopyOnWriteArrayList、ConcurrentSkipListMap、BlockingQueue

### 3.线程管理类

线程池 比如使用Executors静态工厂或者ThreadPoolExecutor

通过ScheduleExecutorService执行定时任务

------

ThreadPoolExecutor：

```java
public ThreadPoolExecutor{
    int corePoolSize,					// 第1个参数 常驻核心线程数 线程执行完毕也不销毁
    int maximumPoolSize,				// 第2个参数 最大线程数 大于corePoolSize部分为动态创建
    long keepAliveTime,					// 第3个参数 线程池中的线程空闲时间 线程任务结束后 超过空闲时间就销毁，如果ThreadPoolExecutor的allowCoreThreadTimeOut设置为true，核心线程超时后也会被回收
    TimeUnit unit,						// 第4个参数 时间单位 比如 TimeUnit.SECONDS
    BlockingQueue<runnable> workQueue,	// 第5个参数 缓存队列。当请求的线程大于maximumPoolSize时，线程加入缓存队列。比如LinkedBlockingQueue
    ThreadFactory threadFactory,		// 第6个参数 线程工厂，用来生产一组相同任务的线程。线程池的命名是通过给这个factory增加组名的前缀来实现的，这样就可以知道是哪个线程工厂生产的
    RejectedExecutionHandler handler){	// 第7个参数 执行拒绝策略的对象。当超过第五个参数workQueue的任务缓存区上限时，就可以通过该策略处理请求，一种简单的限流保护。
        //友好的拒绝策略
        //1.保存到数据库中进行削峰填谷 2.转向某个提示页面 3.打印日志
        if(corePoolSize < 0 ||
          // maximumPoolSize 必须大于等于0 也要大于或等于corePoolSize 
          maximumPoolSize <= 0 ||
          maximumPoolSize < corePoolSize ||
          keepAliveTime < 0)
          throw new IllegalArgumentException();// 第2处
        if(workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
    }
}
```

都是靠executor的execute来运行

```java
  public static void main(String[] args) {
        Executor executor = new Executor() {
            @Override
            public void execute(Runnable command) {
                
            }
        };
    }
```

Executor**s**静态工厂　五个

```java
Executors.newWorkStealingPool();//JDK8引入，创建持有足够线程的线程池支持给定的并行度，并通过使用多个队列减少竞争，此构造方法中把CPU数量设置为默认的并行度 返回FoorkJoinPool
Executors.newCachedThreadPool();//最大池数量为Integer.MAX_VALUE 高度可伸缩的线程池，有OOM（out of memory）风险 KeepAlieTime为60秒 核心为0
Executors.newFixedThreadPool(2);//固定线程 不存在空闲线程 即keepAliveTime = 0
Executors.newSingleThreadExecutor();//单线程串行执行所有任务 保证按任务的提交顺序依次执行
Executors.newScheduledThreadPool(2);//线程最大为Integer.MAX_VALUE 和CachedThreadPool的区别为不回收工作线程 支持定时及周期性任务执行
```

所有类似 LinkedBlockingQueue这样的无界队列 ，如果瞬间请求非常大 有OOM风险。

除了newWorkStealingPool 其他都有资源耗尽风险

- 合理设计各类参数，应根据实际业务场景来设置合理的工作线程数。
- 线程资源必须通过线程池提供，不允许在应用中自行显式创建线程
- 创建线程或线程池请指定有意义的线程名称，方便出错时回溯

## ThreadLocal

### 强引用

```java
Object object = new Object();
```

只要对象有强引用指向，并且 GC Roots 可达，那么 Java 内存回收时，即使濒临内存耗尽，也不会回收该对象。

### 软引用

引用力度弱于“强引用”　，用在非必需对象的场景。在即将OOM前，垃圾回收器会把这些软引用指向的对象加入回收范围。

### 弱引用

相比前两者最弱，也是描述非必需对象，如果只有弱引用指向，则在下一个YGC (Young GC)时被回收。主要指向易消失的对象。

### 虚引用

极弱的引用，定义完成后就无法通过该引用获取对象，唯一目的是希望在这个对象被回收时收到一个系统通知。虚引用必须与引用队列联合使用，如果发现存在虚引用，就会在回收对象内存前，把这个虚引用加入与之关联的引用队列
中。

![](https://user-images.githubusercontent.com/25349066/53676378-5222e100-3cdc-11e9-989d-347865f0ec87.png)

- 每个ThreadLocal只能保存一个变量副本，如果想要上线一个线程能够保存多个副本以上，就需要创建多个ThreadLocal。
- ThreadLocal内部的ThreadLocalMap键为弱引用，会有内存泄漏的风险。
- 适用于无状态，副本变量独立后不影响业务逻辑的高并发场景。如果如果业务逻辑强依赖于副本变量，则不适合用ThreadLocal解决，需要另寻解决方案。

### 4.锁相关类

以lock接口为核心，在实际场景中进行互斥的锁相关类 

比如ReentrantLock 可重入锁。

共享锁 （读锁)

### volatile

每个线程都有独立的缓存  volatile保证内存可见性，让变量不存cache 存主存 对其他线程可见。

i++分为 "读-改-写"三个步骤

volatile没有互斥性 多个线程仍然可以同时改 问题依然存在 因为i++有三步操作 不是原子操作

### atomic

java.util.concurrent.atomic 包下提供了常用的原子变量

1. volatile保证内存可见性

2. CAS算法保证数据的原子性

   CAS算法是**硬件**对于并发操作共享数据的支持

   CAS包含了3个操作数

   内存值V 预估值A 更新值 B

   当且仅当A=V 时 才把B值赋给V 否则什么都不做 

   无锁 无时间片切换

### 同步容器类

synchronized  List的问题

添加操作多时 copyOnWriteList开销大 每次都会复制 并发迭代多哦时可以选择

## 多线程

### new状态 创建线程的三种方法

1. 继承Thread类

2. 实现Runnable类

3. 实现Callable接口

   相比第一种 推荐第二种方法，实现Runnable的方式更加灵活，对外暴露的细节较少，让使用者专注于实现线程的run()方法上。

   第三种Callable接口call()的声明如下

   ```java
   V call() throws Exception;
   ```

   Callable和Runnable的区别 Callable和Future可以直接获取返回值，前两个需要共享内存。call()可以抛出异常，而Runnable只有通过setDefaultUncaughtExceptionHandler()的方式才能在主线程中捕捉到子线程异常。

### Runnable状态

就绪态 调用start()后 运行之前的状态 等待分配时间片 start()不可多次调用 否则会抛出IllegalStateException异常。

### Running状态

运行状态，是run()正在执行时线程的状态 线程可能由某些因素退出running 比如时间、异常、调度、锁等

### Blocked状态

阻塞状态 

- 同步阻塞 锁被其他线程占用
- 调用Thread的某些方法，自动让出CPU执行权，比如sleep()、join()等
- 等待阻塞：执行了wait()

### Dead状态

终止状态 run()执行结束 或异常退出后的状态 此状态不可逆转

### 线程安全

1. 数据单线程内可见：最典型的是线程局部变量，ThreadLocal就是这种方法
2. 只读对象：private final 避免被篡改
3. 线程安全类：比如StringBuffr 采用synchronized关键字修饰相关方法
4. 同步与锁机制：

## hashCode和equals

hashCode和equals配合来判断两个对象是否相等。 

由于不可避免地会存在哈希值冲突的情况，因此当hashCode相同时，还要调用一遍equals，不同时，直接判断为不同，加快了处理效率。

## springboot和spring和spring mvc的区别

**Spring 是一个“引擎”；**

**Spring MVC 是基于Spring的一个 MVC 框架；**

**Spring Boot 是基于Spring4的条件注册的一套快速开发整合包。**

## 数据库

### 范式

1NF : 数据项不可再分

2NF：消除非主属性对码的部分依赖（只需要部分主码便可以确定某属性）

3NF：消除非主属性对码的传递依赖

BCNF：消除主属性对码的部分依赖和传递依赖

4NF:消除非平凡且非函数依赖的多值依赖

![image](https://user-images.githubusercontent.com/25349066/53689779-43016900-3d98-11e9-957f-acf254a27e0f.png)

### ACID

隔离：数据库系统提供一定的隔离机制，保证事务在不受外部并发操作影响的“独立”环境执行。这意味着事务处理过程中的中间状态对外部是不可见的，反之亦然。

持久：事务完成之后，它对于数据的修改是永久性的，即使出现系统故障也能够保持。

原子：事务是一个原子操作单元，其对数据的修改，要么全都执行，要么全都不执行。

一致 ：在事务开始和完成时，数据都必须保持一致状态。这意味着所有相关的数据规则都必须应用于事务的修改，以操持完整性；事务结束时，所有的内部数据结构（如B树索引或双向链表）也都必须是正确的。

## 隔离级别

未提交读 ： 脏读

提交读 ： 不可重复读

可重复读 ：幻读

串行 ： 单线程

幻读：一个事务读同一张表，

## Innodb和myisam

MyISAM关注点 ： **性能** 

InnoDB关注点：   **事务**

| 对比项 | MyISAM                                                 | InnoDB                                                       |
| ------ | ------------------------------------------------------ | ------------------------------------------------------------ |
| 主外键 | 不支持                                                 | 支持                                                         |
| 事务   | 不支持                                                 | 支持                                                         |
| 行表锁 | 表锁，即使操作一条记录也会锁住整个表，不适合高并发操作 | 行锁，操作时只锁某一行，不对其他行有影响， <font color=#ff0000>适合高并发的操作</font>，行锁性能开销大 |
| 缓存   | 只缓存索引，不缓存真实数据                             | 不仅缓存索引，还要缓存真实数据，对内存要求较高，而且内存大小对性能有决定性的影响 |
| 表空间 | 小                                                     | 大                                                           |

都默认安装

MyISAM偏向读

parcona

### 索引

是一种数据结构  是帮助Mysql高效获取数据的数据结构 

“排好序的快速查找数据结构”

比如 mysql 先找m再找y

如果没有索引 就要遍历



在数据之外，<font color = #ff0000>数据库还维护着满足特定查找算法的数据结构</font>，这些数据结构以某种方式引用指向数据，这样就可以在这些数据结构的基础上实现高级查找算法，这种数据结构就是索引。



一般来说 索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储在磁盘上。

<font color=#ff0000>我们平时所说的索引，没有特别指明。都是指b树（多路搜索树）结构组织的索引</font>。其中聚集索引，次要索引，覆盖索引，复合索引，前缀索引，唯一索引默认都是使用B+树索引，统称索引。当然除了B+树这类索引以外，还有hash索引

聚集索引  ： 比如新华字典 一个表只能有一个聚集索引

非聚集索引：新华字典的偏旁索引 实际顺序和结构顺序不一定一致。

mysql在使用like查询的时候只有使用后面的%时，才会使用到索引

### MVCC

并发控制版本技术

与隔离级别紧密联系的另外一个东西是并发调度，通过并发调度实现隔离级别。对于并发调度，不同的数据库厂商有不同的实现机制，但基本原理类似，都是通过加锁来保护数据对象不同时被多个事务修改。

多版本的并发控制(MVCC)相对于传统的基于锁的并发控制主要特点是**读不上锁**，这种特性对于读多写少的场景，大大提高了系统的并发度，因此大部分关系型数据库都实现了MVCC。

InnoDB的MVCC,是**通过在每行记录后面保存两个隐藏的列来实现的,这两个列，分别保存了这个行的创建时间，一个保存的是行的删除时间**。

这里**存储的并不是实际的时间值,而是系统版本号(可以理解为事务的ID)**，每开始一个新的事务，系统版本号就会自动递增，事务开始时刻的系统版本号会作为事务的ID.

**INSERT**

InnoDB为新插入的每一行保存当前系统版本号作为版本号. 

 

**SELECT**

InnoDB会根据以下两个条件检查每行记录: 

a.InnoDB只会查找版本早于当前事务版本的数据行

(也就是,行的系统版本号小于或等于事务的系统版本号)，这样可以确保事务读取的行，要么是在事务开始前已经存在的，要么是事务自身插入或者修改过的. 

 

b.行的删除版本要么未定义,要么大于当前事务版本号

这可以确保事务读取到的行，在事务开始之前未被删除. 

只有a,b同时满足的记录，才能返回作为查询结果.

 

**DELETE**

InnoDB会为删除的每一行保存当前系统的版本号(事务的ID)作为删除标识. 

 

**UPDATE**

InnoDB执行UPDATE，实际上是新插入了一行记录，并保存其创建时间为当前事务的ID，同时保存当前事务ID到要UPDATE的行的删除时间.

### 红黑树

每一颗红黑树都对应一颗4阶B树

1. 根节点都是黑
2. 叶子都是黑
3. 红节点不相连
4. 到叶子的路径上黑节点都一样
5. 节点都是红黑

## 类加载

[![image](https://user-images.githubusercontent.com/25349066/53934786-f93db900-40de-11e9-8e97-68b26a60fae3.png)](https://user-images.githubusercontent.com/25349066/53934786-f93db900-40de-11e9-8e97-68b26a60fae3.png)



## 反射

每个类有该类的静态class对象 通过该class对象实现反射

## 安全



## KMP

![1552101280980](/home/arch/.config/Typora/typora-user-images/1552101280980.png)

在B不对应时 已经确定B之前的都是对应的 同时B之前的AB与开头的AB为最长前后缀 故可以直接将第一个AB移动到前一个AB的位置

## sql题

1000个列，列的值非0即1，求每一行值加起来大于300的数据行，全部select出来

```sql
SET @qry = (SELECT CONCAT('SELECT id, SUM(', GROUP_CONCAT(INFORMATION_SCHEMA.COLUMNS.COLUMN_NAME SEPARATOR '+'), ') as all_sum
 FROM ', INFORMATION_SCHEMA.COLUMNS.TABLE_NAME)
            FROM INFORMATION_SCHEMA.COLUMNS
            WHERE INFORMATION_SCHEMA.COLUMNS.TABLE_NAME = 'table_name');
SELECT id
from (@qry)
where all_sum > 300;
```



mysql内部有information_schema数据库 内部的columns表有所有表的信息 根据表名找到所有的列名 使用separator设定用+号分割

**group_concat( [DISTINCT]  要连接的字段   [Order BY** **排序字段** **ASC/DESC]   [Separator '分隔符'] )**

按照分隔符连接字段

concat 函数用于将多个字符串连接成一个字符串   

作者：公众号-Java名企面试吧

链接：

https://www.nowcoder.com/discuss/140003

来源：牛客网

## BIO、NIO与AIO的概念与区别 

  BIO表示同步阻塞式IO，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。 

> 传统IO面向流　NIO面向缓冲区
>
> NIO三大核心部分 Channel 、Buffer、Selector 管道 缓存 选择器
>
> 数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。
>
> 通过selector监听多个通道的事件 

  NIO表示同步非阻塞IO，服务器实现模式为一个请求一个线程，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。 

  AIO表示异步非阻塞IO，服务器实现模式为一个有效请求一个线程，客户端的I/O请求都是由操作系统先完成IO操作后再通知服务器应用来启动线程进行处理。

NIO非阻塞靠selector

```java
while(selector.selector()>0) //每次selector都是轮询所有的channel
```

一个线程轮询多个连接

## 常见的运行时异常有哪些

- ArithmeticException（算术异常）    

- ClassCastException （类转换异常）    

- IllegalArgumentException （非法参数异常）    

- IndexOutOfBoundsException （下标越界异常）    

- NullPointerException （空指针异常）    

- SecurityException （安全异常）

  

## TCP/UDP

![image](https://user-images.githubusercontent.com/25349066/54065517-f5d03c00-425c-11e9-8f89-1c5a2c5a4a15.png)

![image](https://user-images.githubusercontent.com/25349066/54065666-3630b980-425f-11e9-98f3-409672104b5c.png)

[![image](https://user-images.githubusercontent.com/25349066/54065527-07b1df00-425d-11e9-950c-c7eebb96e5ce.png)](https://user-images.githubusercontent.com/25349066/54065527-07b1df00-425d-11e9-950c-c7eebb96e5ce.png)
[![image](https://user-images.githubusercontent.com/25349066/54065531-14cece00-425d-11e9-8336-6d74d2c18bac.png)](https://user-images.githubusercontent.com/25349066/54065531-14cece00-425d-11e9-8336-6d74d2c18bac.png)

## 排序稳定性

归并和冒泡稳定

## sql找前三

```sql
select * from student a where not exists(select * from student b where a.class = b.class and b.score > a.score group by class having count(*)>3)

```

```sql
select * from student a where (select count(*) from student b where a.class = b.class and b.score >a.scoore group by class)<3
```

## tomcat类加载机制

> **Tomcat的类加载机制是违反了双亲委托原则的，对于一些未加载的非基础类(Object,String等)，各个web应用自己的类加载器(WebAppClassLoader)会优先加载，加载不到时再交给commonClassLoader走双亲委托。** 
>
> **对于JVM来说：**
>
> **因此，按照这个过程可以想到，如果同样在CLASSPATH指定的目录中和自己工作目录中存放相同的class，会优先加载CLASSPATH目录中的文件。**
>
> 2、我们思考一下：Tomcat是个web容器， 那么它要解决什么问题： 
>
> 1. 一个web容器可能需要部署两个应用程序，不同的应用程序可能会依赖同一个第三方类库的不同版本，不能要求同一个类库在同一个服务器只有一份，因此要保证每个应用程序的类库都是独立的，保证相互隔离。 
> 2. 部署在同一个web容器中相同的类库相同的版本可以共享。否则，如果服务器有10个应用程序，那么要有10份相同的类库加载进虚拟机，这是扯淡的。 
> 3. web容器也有自己依赖的类库，不能于应用程序的类库混淆。基于安全考虑，应该让容器的类库和程序的类库隔离开来。 
> 4. web容器要支持jsp的修改，我们知道，jsp 文件最终也是要编译成class文件才能在虚拟机中运行，但程序运行后修改jsp已经是司空见惯的事情，否则要你何用？ 所以，web容器需要支持 jsp 修改后不用重启。
>
> **JAVA类加载双亲委托的目的是要维护一个类的一个版本**
>
> > 双亲委派模型要求除了顶层的启动类加载器之外，其余的类加载器都应当由自己的父类加载器加载。
>
> 而tomcat是容器 可能会同时运行多个应用，每个应用的依赖类版本不同，希望有多个版本
>
> 很显然，tomcat 不是这样实现，tomcat 为了实现隔离性，没有遵守这个约定，每个webappClassLoader加载自己的目录下的class文件，不会传递给父类加载器。

## GC

![Minor GCãMajor GCåFull GCä¹é´çåºå"](http://static.open-open.com/lib/uploadImg/20150424/20150424214649_846.jpg)

### minor GC

从年轻代空间（包括 Eden 和 Survivor 区域）回收内存被称为 Minor GC。这一定义既清晰又易于理解。但是，当发生Minor GC事件的时候，有一些有趣的地方需要注意到：

1. 当 JVM 无法为一个新的对象分配空间时会触发 Minor GC，比如当 Eden 区满了。所以分配率越高，越频繁执行 Minor GC。
2. 内存池被填满的时候，其中的内容全部会被复制，指针会从0开始跟踪空闲内存。Eden 和 Survivor 区进行了标记和复制操作，取代了经典的标记、扫描、压缩、清理操作。所以 Eden 和 Survivor 区不存在内存碎片。写指针总是停留在所使用内存池的顶部。
3. 执行 Minor GC 操作时，不会影响到永久代。从永久代到年轻代的引用被当成 GC roots，从年轻代到永久代的引用在标记阶段被直接忽略掉。
4. 质疑常规的认知，所有的 Minor GC 都会触发“全世界的暂停（stop-the-world）”，停止应用程序的线程。对于大部分应用程序，停顿导致的延迟都是可以忽略不计的。其中的真相就 是，大部分 Eden 区中的对象都能被认为是垃圾，永远也不会被复制到 Survivor 区或者老年代空间。如果正好相反，Eden 区大部分新生对象不符合 GC 条件，Minor GC 执行时暂停的时间将会长很多。

所以 Minor GC 的情况就相当清楚了——每次 Minor GC 会清理年轻代的内存。

## Major GC VS Full GC

大家应该注意到，目前，这些术语无论是在 JVM 规范还是在垃圾收集研究论文中都没有正式的定义。但是我们一看就知道这些在我们已经知道的基础之上做出的定义是正确的，Minor GC 清理年轻带内存应该被设计得简单：

- Major GC 是清理老年代。
- Full GC 是清理整个堆空间—包括年轻代和老年代。

很不幸，实际上它还有点复杂且令人困惑。首先，许多 Major GC 是由 Minor GC 触发的，所以很多情况下将这两种 GC 分离是不太可能的。另一方面，许多现代垃圾收集机制会清理部分永久代空间，所以使用“cleaning”一词只是部分正确。

这使得我们不用去关心到底是叫 Major GC 还是 Full GC，大家应该关注当前的 GC 是否停止了所有应用程序的线程，还是能够并发的处理而不用停掉应用程序的线程。

## 代理模式

### 静态代理

```java
public class CountProxy implements Count {  
    private CountImpl countImpl;  //组合一个业务实现类对象来进行真正的业务方法的调用
    /** 
     * 覆盖默认构造器 
     *  
     * @param countImpl 
     */  
    public CountProxy(CountImpl countImpl) {  
        this.countImpl = countImpl;  
    }  
  
    @Override  
    public void queryCount() {  
        System.out.println("查询账户的预处理——————");  
        // 调用真正的查询账户方法
        countImpl.queryCount();  
        System.out.println("查询账户之后————————");  
    }  
  
    @Override  
    public void updateCount() {  
        System.out.println("修改账户之前的预处理——————");  
        // 调用真正的修改账户操作
        countImpl.updateCount();  
        System.out.println("修改账户之后——————————");  
    }  
}  
```

### JDK动态代理

实现 调用管理接口InvocationHandler  创建动态代理类

```java
public class BookFacadeProxy implements InvocationHandler {  
    private Object target;//这其实业务实现类对象，用来调用具体的业务方法 
    /** 
     * 绑定业务对象并返回一个代理类  
     */  
    public Object bind(Object target) {  
        this.target = target;  //接收业务实现类对象参数
       //通过反射机制，创建一个代理类对象实例并返回。用户进行方法调用时使用
       //创建代理对象时，需要传递该业务类的类加载器（用来获取业务实现类的元数据，在包装方法是调用真正的业务方法）、接口、handler实现类
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),  
                target.getClass().getInterfaces(), this); }  
    /** 
     * 包装调用方法：进行预处理、调用后处理 
     */  
    public Object invoke(Object proxy, Method method, Object[] args)  
            throws Throwable {  
        Object result=null;  

        System.out.println("预处理操作——————");  
        //调用真正的业务方法  
        result=method.invoke(target, args);  

        System.out.println("调用后处理——————");  
        return result;  
    }  
 
}  
```

### CGLIB

```java
public class BookFacadeCglib implements MethodInterceptor {  
    private Object target;//业务类对象，供代理方法中进行真正的业务方法调用
  
    //相当于JDK动态代理中的绑定
    public Object getInstance(Object target) {  
        this.target = target;  //给业务对象赋值
        Enhancer enhancer = new Enhancer(); //创建加强器，用来创建动态代理类
        enhancer.setSuperclass(this.target.getClass());  //为加强器指定要代理的业务类（即：为下面生成的代理类指定父类）
        //设置回调：对于代理类上所有方法的调用，都会调用CallBack，而Callback则需要实现intercept()方法进行拦
        enhancer.setCallback(this); 
       // 创建动态代理类对象并返回  
       return enhancer.create(); 
    }
    // 实现回调方法 
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable { 
        System.out.println("预处理——————");
        proxy.invokeSuper(obj, args); //调用业务类（父类中）的方法
        System.out.println("调用后操作——————");
        return null; 
    } 
```

比较

静态代理是通过在代码中显式定义一个业务实现类一个代理，在代理类中对同名的业务方法进行包装，用户通过代理类调用被包装过的业务方法；

JDK动态代理是通过接口中的方法名，在动态生成的代理类中调用业务实现类的同名方法；

CGlib动态代理是通过继承业务类，生成的动态代理类是业务类的子类，通过重写业务方法进行代理；

## 对象进入老年代的几种方式

一般情况是四种，但是尤其以第一种来源最多

1.新生代对象每经历依次minor gc，年龄会加一，当达到年龄阀值会直接进入老年代。阀值大小一般为15

2.为了能更好地适应不同程度的内存状况，虚拟机并不是永远地要求对象的年龄必须达到了MaxTenuringThreshold才能晋升到老年代，如果在Survivor空间中相同年龄的所有对象大小的总和大于Survivor空间的一半，年龄大于或等于年龄的对象就可以直接进入老年代，无须等到MaxTenuringThreshold中要求的年龄。

3.大对象直接进入老年代

4.新生代复制算法需要一个survivor区进行轮换备份，如果出现大量对象在minor gc后仍然存活的情况时，就需要老年代进行分配担保，让survivor无法容纳的对象直接进入老年代

## redis持久化

redis提供两种方式进行持久化，一种是RDB持久化（原理是将Reids在内存中的数据库记录定时dump到磁盘上的RDB持久化），另外一种是AOF持久化（原理是将Reids的操作日志以追加的方式写入文件）

RDB：定时dump

AOF：日志追加

RDB优势：适合大规模的数据恢复，对数据完整性和一致性要求不高

------



RDB劣势：在一定间隔时间做一次备份，所以如果redis意外down掉的话，就会丢失最后一次快照后的所有修改。

fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑

------



AOF优势：每修改同步：appendfsync always   同步持久化 每次发生数据变更会被立即记录到磁盘  性能较差但数据完整性比较好

每秒同步：appendfsync everysec    异步操作，每秒记录   如果一秒内宕机，有数据丢失

不同步：appendfsync no   从不同步

------



AOF劣势: 相同数据集的数据而言aof文件要远大于rdb文件，恢复速度慢于rdb 

aof运行效率要慢于rdb,每秒同步策略效率较好，不同步效率和rdb相同

## Redis 复制

![image](https://user-images.githubusercontent.com/25349066/54109950-0c23f680-441b-11e9-8038-d2f11376b850.png)

slave of 或者写入配置文件 若没写 每次都需要slave of

SLAVEOF no one 反客为主 变成主服务器

### 哨兵模式

反客为主的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库

![image](https://user-images.githubusercontent.com/25349066/54110066-5b6a2700-441b-11e9-85e2-7124b8922345.png)

## 同步和异步的区别

同步交互：指发送一个请求,需要等待返回,然后才能够发送下一个请求，有个等待过程；

异步交互：指发送一个请求,不需要等待返回,随时可以再发送下一个请求，即不需要等待。

 区别：一个需要等待，一个不需要等待，在部分情况下，我们的项目开发中都会优先选择不需要等待的异步交互方式。

## 中断

**中断**、**故障**、**陷阱**、**终止**

中断向量表 2KB 256个 前32个为intel保留

## springboot和spring的区别

springboot没有和如何Web MVC 持久化绑定 

约定大于配置 简化大量xml配置 开箱即用

SSM：面向xml编程

springboot：面向注解编程

springboot提供默认配置

## 分布式一致性协议

- 二阶段提交协议(2pc)
- 三阶段提交协议(3pc)
- paxios
- zab

### 2pc

1. 提交事务请求

   > 1. 协调者向参与者发送事务，并询问是否可以执行事务提交操作，等待参与者响应；
   > 2. 参与者执行事务，将操作写入本地事务日志，向协调者发送反馈；

2. 执行事务请求

   > ### 参与者反馈，全部ACK
   >
   > 1. 协调者向参与者发送Commit；
   > 2. 参与者执行事务提交，释放事务资源，反馈ACK，；
   > 3. 协调者收到反馈完成事务
   >
   > ### 参与者反馈，存在NO；协调者等待超时
   >
   > 1. 协调者向参与者发送RollBack；
   > 2. 参与者利用undo，进行事务回滚；
   > 3. 参与者事务回滚之后，向协调者发送ACK 反馈；
   > 4. 协调者接收到ACK，完成事务中断；

   提交事务请求 -> 执行事务提交；缺点：同步阻塞（参与者之间阻塞）、单点问题，脑裂（导致数据不一致问题）；主要用于关系型数据库中，解决了分布式事务的原子性问题；

   二阶段提交有什么缺点？ 答：效率不够高，因为在资源锁定的时候，订单系统不能接受其他请求，业界采用三阶段提交。

### 3pc

cancommit -> precommit -> docommit；优缺点：降低了参与者同步阻塞范围，但是又引入了数据不一致性问题（若出现网络分区）、单点问题依然存在；

1. CanCommit

   > 1. 协调者向参与者发送cancommit 请求（包含事务内容），等待参与者反馈；
   > 2. 参与者接收到cancommit 请求之后，反馈ACK或者NO；

2. PreCommit

   > ### 参与者反馈，全部ACK
   >
   > 1. 协调者向参与者发送precommit 请求，等待反馈；
   > 2. 参与者执行事务，将操作写入本地事务日志，向协调者发送反馈；
   >
   > ### 参与者反馈，存在NO；协调者等待超时
   >
   > 1. 协调者向参与者发送abort 请求；
   > 2. 参与者无论是接收到abort 请求还是等待超时都会中断事务；

3. DoCommit

   > ### 参与者反馈，全部ACK
   >
   > 1. 协调者向参与者发送docommit 请求，等待反馈；
   > 2. 参与者执行事务提交，释放事务资源，反馈ACK，；
   > 3. 协调者接收反馈，完成事务提交；
   >
   > ### 参与者反馈，存在NO；协调者等待超时
   >
   > 1. 协调者向参与者发送abort 请求；
   > 2. 参与者进行事务回滚；
   > 3. 参与者反馈ACK；
   > 4. 协调者接收到ACK，完成事务中断；

### paxios

### zab

ZAB 协议是为分布式协调服务（Zookeeper）专门设计的一种支持**故障恢复**的**原子广播**协议。

## zookeeper

Zoookeeper是一个开源的分布式的，为分布式应用提供协调服务的Apache项目。

> Zookeeper从设计模式角度来理解，是一个基于**观察者模式**设计的分布式服务管理框架。负责存储和管理大家都关心的数据，然后接受观察者的注册，一旦数据状态发生变化，Zookeeper就将负责通知已经在Zookeeper上注册的那些观察者。类似于Master/Slave 

![image](https://user-images.githubusercontent.com/25349066/54125902-37214100-4441-11e9-9abe-b9aea85efe20.png)

Zookeeper = 文件系统 + 通知机制（观察节点并通知客户端)

集群中只有半数**以上**节点存活 ，Zookeeper就能正常服务

![image](https://user-images.githubusercontent.com/25349066/54126545-c418ca00-4442-11e9-9901-2392c37c9515.png)

全局数据一致 实时性 一段时间范围client能读到最新数据  部署Zookeeper一定是奇数台！

Zookeeper文件系统类似于linux 树形结构 每个节点1M 用于存放配置文件 而不是数据

提供的服务包括：统一命名服务（为服务命名 类似于域名 代理）、统一配置管理（将配置信息写入Znode）、统一集群管理(将节点信息写入Zookeeper上的一个Znode，监听这个Znode可获取它的实时状态变化 例如Hbase中Master状态监控和选举)、服务器节点动态上下线、软负载均衡等。

## linux查找最近使用文件

find / -name targetfilename

### 统计单个文件

统计单个文件字符串出现次数，语法：grep 字符串 文件名|wc -l ，grep输出，wc -l按行统计，每行重复只统计一个

如：统计task-hbase-transform.log中NullPointerException出现的次数

grep NullPointerException task-hbase-transform.log|wc -l

grep -o 12 test | wc -c 

![1552312061317](/home/arch/.config/Typora/typora-user-images/1552312061317.png)

```bash
#!bin/sh
for file in /logs/task-hbase-transform/* #日志文件路径
do
    if test -f $file #如果是文件，统计异常数量，并输出到ex.log
    then
         e=`grep Exception "$file"|wc -l` #按行统计并输出
         echo "Exception--"$file"--"$e >>ex.log #把统计内容输出到ex.log中
        #echo $file 是文件   >> c.log
    else
        echo $file 是目录
    fi
done
```

```bash
#!/bin/bash

a=0
for file in $(find / -mmin -30)
do	
	if [ -f $file ];then
	b=$(grep -o "h" $file |wc -c)
	a=$[$a+$b]
	fi
done
echo a
```

## Http头

通用头域包含Cache-Control、 Connection、Date、Pragma、Transfer-Encoding、Upgrade、Via

## Cookie 有什么字段

**name**　　字段为一个cookie的名称。

**value**　　字段为一个cookie的值。

**domain**　　字段为可以访问此cookie的域名。

非顶级域名，如二级域名或者三级域名，设置的cookie的domain只能为顶级域名或者二级域名或者三级域名本身，不能设置其他二级域名的cookie，否则cookie无法生成。

顶级域名只能设置domain为顶级域名，不能设置为二级域名或者三级域名，否则cookie无法生成。

二级域名能读取设置了domain为顶级域名或者自身的cookie，不能读取其他二级域名domain的cookie。所以要想cookie在多个二级域名中共享，需要设置domain为顶级域名，这样就可以在所有二级域名里面或者到这个cookie的值了。
顶级域名只能获取到domain设置为顶级域名的cookie，其他domain设置为二级域名的无法获取。

**path**　　字段为可以访问此cookie的页面路径。 比如domain是abc.com,path是/test，那么只有/test路径下的页面可以读取此cookie。

**expires/Max-Age** 　　字段为此cookie超时时间。若设置其值为一个时间，那么当到达此时间后，此cookie失效。不设置的话默认值是Session，意思是cookie会和session一起失效。当浏览器关闭(不是浏览器标签页，而是整个浏览器) 后，此cookie失效。

**Size**　　字段 此cookie大小。

**http**　　字段  cookie的httponly属性。若此属性为true，则只有在http请求头中会带有此cookie的信息，而不能通过document.cookie来访问此cookie。

**secure**　　 字段 设置是否只能通过https来传递此条cookie

## Cookie和session怎么联系起来

用户首次与Web服务器建立连接的时候，服务器会给用户分发一个 SessionID作为标识。SessionID是一个由24个字符组成的随机字符串。用户每次提交页面，浏览器都会把这个SessionID包含在 HTTP头中提交给Web服务器，这样Web服务器就能区分当前请求页面的是哪一个客户端。这个SessionID就是保存在客户端的，属于客户端Session。其实客户端Session默认是以cookie的形式来存储的。

## TCP属性用来控制TCP连接

## IPV6和IPV4区别

128位地址

没有checksum 不能分段

简化而高效的头部



## chmod和chown

chmod修改文件权限

chown修改用户组

## spring循环引用



## fail-fast

多个线程同时操作list 某个线程执行迭代器过程中，数据项变化 抛出异常

## Spring循环依赖

1. 单例模式 Autowaired直接写在成员名上

   spring会在实例化时创建成员对象，但发现成员对象同样在创建中时，抛出循环依赖异常

2. 单例模式 Autowaired 写在set方法上，不会报错，正常运行 spring可以先实例化完，成员为null时注入其他类中

3. 多例模式写在set方法上，报错，spring容器无法缓存prototype实例，**对于“prototype”作用域Bean，Spring容器无法完成依赖注入，因为“prototype”作用域的Bean，Spring容器不进行缓存，因此无法提前暴露一个创建中的Bean。** 

## **Raft算法**



## hashmap 大小为什么一直是2的n次幂

假设扩容前的table大小为2的N次方，有上述put方法解析可知，元素的table索引为其hash值的后N位确定

那么扩容后的table大小即为2的N+1次方，则其中元素的table索引为其hash值的后N+1位确定，比原来多了一位

因此，table中的元素只有两种情况：

1. 元素hash值第N+1位为0：不需要进行位置调整
2. 元素hash值第N+1位为1：调整至原索引的两倍位置

## http 1.0 1.1 2.0

1.1:

- 缓存处理 1.0中使用If-Modified-Since 1.1引入了更多的缓存
- **带宽优化及网络连接的使用** HTTP1.0中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持**断点续传**功能，HTTP1.1则在请求头引入了range头域，它允许只请求资源的某个部分，即返回码是206（Partial Content），这样就方便了开发者自由的选择以便于充分利用带宽和连接。
- **错误通知的管理**，在HTTP1.1中新增了24个错误状态响应码，如409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除。
- **Host头处理**，在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。
- **长连接**，HTTP 1.1支持长连接（PersistentConnection）和请求的流水线（Pipelining）处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟，在HTTP1.1中默认开启Connection： keep-alive，一定程度上弥补了HTTP1.0每次请求都要创建连接的缺点。

1.2:

- **新的二进制格式**（Binary Format），HTTP1.x的解析是基于文本。基于文本协议的格式解析存在天然缺陷，文本的表现形式有多样性，要做到健壮性考虑的场景必然很多，二进制则不同，只认0和1的组合。基于这种考虑HTTP2.0的协议解析决定采用二进制格式，实现方便且健壮。
- **多路复用**（MultiPlexing），即连接共享，即每一个request都是是用作连接共享机制的。一个request对应一个id，这样一个连接上可以有多个request，每个连接的request可以随机的混杂在一起，接收方可以根据request的 id将request再归属到各自不同的服务端请求里面。
- **header压缩**，如上文中所言，对前面提到过HTTP1.x的header带有大量信息，而且每次都要重复发送，HTTP2.0使用encoder来减少需要传输的header大小，通讯双方各自cache一份header fields表，既避免了重复header的传输，又减小了需要传输的大小。
- **服务端推送**（server push），同SPDY一样，HTTP2.0也具有server push功能。

## 重量级锁 轻量级锁 偏向锁

### 重量级锁

内置锁在Java中被抽象为监视器锁（monitor）。在JDK 1.6之前，监视器锁可以认为直接对应底层操作系统中的互斥量（mutex）。这种同步方式的成本非常高，包括系统调用引起的内核态与用户态切换、线程阻塞造成的线程切换等。因此，后来称这种锁为“重量级锁”。

### 自旋锁

首先，内核态与用户态的切换上不容易优化。但**通过自旋锁，可以减少线程阻塞造成的线程切换**（包括挂起线程和恢复线程）。

如果锁的粒度小，那么**锁的持有时间比较短**（尽管具体的持有时间无法得知，但可以认为，通常有一部分锁能满足上述性质）。那么，对于竞争这些锁的而言，因为锁阻塞造成线程切换的时间与锁持有的时间相当，减少线程阻塞造成的线程切换，能得到较大的性能提升。具体如下：

- 当前线程竞争锁失败时，打算阻塞自己
- 不直接阻塞自己，而是自旋（空等待，比如一个空的有限for循环）一会
- 在自旋的同时重新竞争锁
- 如果自旋结束前获得了锁，那么锁获取成功；否则，自旋结束后阻塞自己

*如果在自旋的时间内，锁就被旧owner释放了，那么当前线程就不需要阻塞自己*（也不需要在未来锁释放时恢复），减少了一次线程切换。

“锁的持有时间比较短”这一条件可以放宽。实际上，只要锁竞争的时间比较短（比如线程1快释放锁的时候，线程2才会来竞争锁），就能够提高自旋获得锁的概率。这通常发生在**锁持有时间长，但竞争不激烈**的场景中。

#### 缺点

- 单核处理器上，不存在实际的并行，当前线程不阻塞自己的话，旧owner就不能执行，锁永远不会释放，此时不管自旋多久都是浪费；进而，如果线程多而处理器少，自旋也会造成不少无谓的浪费。
- 自旋锁要占用CPU，如果是计算密集型任务，这一优化通常得不偿失，减少锁的使用是更好的选择。
- 如果锁竞争的时间比较长，那么自旋通常不能获得锁，白白浪费了自旋占用的CPU时间。这通常发生在*锁持有时间长，且竞争激烈*的场景中，此时应主动禁用自旋锁。

### 自适应自旋锁

自适应意味着自旋的时间不再固定了，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定：

- 如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也很有可能再次成功，进而它将允许自旋等待持续相对更长的时间，比如100个循环。
- 相反的，如果对于某个锁，自旋很少成功获得过，那在以后要获取这个锁时将可能减少自旋时间甚至省略自旋过程，以避免浪费处理器资源。

**自适应自旋解决的是“锁竞争时间不确定”的问题**。JVM很难感知到确切的锁竞争时间，而交给用户分析就违反了JVM的设计初衷。*自适应自旋假定不同线程持有同一个锁对象的时间基本相当，竞争程度趋于稳定，因此，可以根据上一次自旋的时间与结果调整下一次自旋的时间*。

#### 缺点

然而，自适应自旋也没能彻底解决该问题，*如果默认的自旋次数设置不合理（过高或过低），那么自适应的过程将很难收敛到合适的值*。

### 轻量级锁

自旋锁的目标是降低线程切换的成本。如果锁竞争激烈，我们不得不依赖于重量级锁，让竞争失败的线程阻塞；如果完全没有实际的锁竞争，那么申请重量级锁都是浪费的。**轻量级锁的目标是，减少无实际竞争情况下，使用重量级锁产生的性能消耗**，包括系统调用引起的内核态与用户态切换、线程阻塞造成的线程切换等。

顾名思义，轻量级锁是相对于重量级锁而言的。使用轻量级锁时，不需要申请互斥量，仅仅*将Mark Word中的部分字节CAS更新指向线程栈中的Lock Record，如果更新成功，则轻量级锁获取成功*，记录锁状态为轻量级锁；*否则，说明已经有线程获得了轻量级锁，目前发生了锁竞争（不适合继续使用轻量级锁），接下来膨胀为重量级锁*。

> Mark Word是对象头的一部分；每个线程都拥有自己的线程栈（虚拟机栈），记录线程和函数调用的基本信息。二者属于JVM的基础内容，此处不做介绍。

当然，由于轻量级锁天然瞄准不存在锁竞争的场景，如果存在锁竞争但不激烈，仍然可以用自旋锁优化，*自旋失败后再膨胀为重量级锁*。

**PS：竞争对象时，CAS改变对象头部信息 获取释放均改变**

### 缺点

同自旋锁相似：

- 如果*锁竞争激烈*，那么轻量级将很快膨胀为重量级锁，那么维持轻量级锁的过程就成了浪费。

### 偏向锁

在没有实际竞争的情况下，还能够针对部分场景继续优化。如果不仅仅没有实际竞争，自始至终，使用锁的线程都只有一个，那么，维护轻量级锁都是浪费的。**偏向锁的目标是，减少无竞争且只有一个线程使用锁的情况下，使用轻量级锁产生的性能消耗**。轻量级锁每次申请、释放锁都至少需要一次CAS，但偏向锁只有初始化时需要一次CAS。

“偏向”的意思是，*偏向锁假定将来只有第一个申请锁的线程会使用锁*（不会有任何线程再来申请锁），因此，*只需要在Mark Word中CAS记录owner（本质上也是更新，但初始值为空），如果记录成功，则偏向锁获取成功*，记录锁状态为偏向锁，*以后当前线程等于owner就可以零成本的直接获得锁；否则，说明有其他线程竞争，膨胀为轻量级锁*。

偏向锁无法使用自旋锁优化，因为一旦有其他线程申请锁，就破坏了偏向锁的假定。

### 缺点

同样的，如果明显存在其他线程申请锁，那么偏向锁将很快膨胀为轻量级锁。

### 小结

> 偏向锁、轻量级锁、重量级锁分配和膨胀的详细过程见后。会涉及一些Mark Word与CAS的知识。

偏向锁、轻量级锁、重量级锁适用于不同的并发场景：

- 偏向锁：无实际竞争，且将来只有第一个申请锁的线程会使用锁。
- 轻量级锁：无实际竞争，多个线程交替使用锁；允许短时间的锁竞争。
- 重量级锁：有实际竞争，且锁竞争时间长。

另外，如果锁竞争时间短，可以使用自旋锁进一步优化轻量级锁、重量级锁的性能，减少线程切换。

如果锁竞争程度逐渐提高（缓慢），那么从偏向锁逐步膨胀到重量锁，能够提高系统的整体性能。

膨胀就是 偏向->轻量->重量的过程

![img](https://upload-images.jianshu.io/upload_images/4491294-345a15342fad119a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

### 长连接、短连接

短连接每次发包都会重新连接（TCP）

## 垃圾回收器



目的：高效收集，减少停顿（stop the world）

CMS： 并发标记清除   使用标记清除 产生大量碎片  

> 通过初始标记、并发标记、重新标记、并发清除四个步骤完成垃圾回收工作 1、3需要停顿  2,4可以同步进行  适用于CPU多的服务器

G1: G1垃圾收集器也是以关注延迟为目标、服务器端应用的垃圾收集器  和CMS相比，G1具备压缩功能，能避免碎片问题，G1的暂停时间更加可控。

> G1将Java堆空间分割成了若干相同大小的区域，即region，，包括 Eden Survivor Old Humongous 四种类型。
>
> Humongous是特殊的Old类型，专门放置大型文件  。这样的划分意味着不需要一个连续的空间管理对象，G1将 
>
> 空间分为多个区域，优先收集垃圾最多的区域  G1的YGC时 S0和S1不会交换

新生代：mark-copy

老年代:  标记清除和标记压缩

## 聚集索引和非聚集索引

聚集索引  ： 比如新华字典 一个表只能有一个聚集索引

非聚集索引：新华字典的偏旁索引 实际顺序和结构顺序不一定一致。

## 给你两个文件（字符串形式的）如何找出他们之间的不同地方？

LCS 最长公共子串 两个字符串构建对比矩阵

## 操作系统分页分段

打个比方，比如说你去[听课](https://www.baidu.com/s?wd=%E5%90%AC%E8%AF%BE&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)，带了一个纸质笔记本做笔记。笔记本有100张纸，课程有语文、数学、英语[三门](https://www.baidu.com/s?wd=%E4%B8%89%E9%97%A8&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)，对于这个笔记本的使用，为了便于以后复习方便，你可以有两种选择。

第一种是，你从本子的第一张纸开始用，并且事先在本子上做划分：第2张到第30张纸记语文笔记，第31到60张纸记数学笔记，第61到100张纸记英语笔记，最后在第一张纸做个列表，记录着三门笔记各自的范围。这就是分段管理，第一张纸叫段表。

第二种是，你从第二张纸开始做笔记，各种课的笔记是连在一起的：第2张纸是数学，第3张是语文，第4张英语……最后呢，你在第一张纸做了一个目录，记录着语文笔记在第3、7、14、15张纸……，数学笔记在第2、6、8、9、11……，英语笔记在第4、5、12……。这就是分页管理，第一张纸叫页表。你要复习哪一门课，就到页表里查寻相关的纸的编号，然后翻到那一页去复习

## Spring事务

ACID

Spring事务管理接口：
PlatformTransactionManager： （平台）事务管理器
TransactionDefinition： 事务定义信息(事务隔离级别、传播行为、超时、只读、回滚规则)
TransactionStatus： 事务运行状态
所谓事务管理，其实就是“按照给定的事务规则来执行提交或者回滚操作”。

PlatformTransactionManager接口介绍
Spring并不直接管理事务，而是提供了多种事务管理器 ，他们将事务管理的职责委托给Hibernate或者JTA等持久化机制所提供的相关平台框架的事务来实现。 Spring事务管理器的接口是： org.springframework.transaction.PlatformTransactionManager ，通过这个接口，Spring为各个平台如JDBC、Hibernate等都提供了对应的事务管理器，但是具体的实现就是各个平台自己的事情了。

事务管理器接口 PlatformTransactionManager 通过 getTransaction(TransactionDefinition definition) 方法来得到一个事务，这个方法里面的参数是 TransactionDefinition类 ，这个类就定义了一些基本的事务属性。

![äºå¡å±æ§](https://user-gold-cdn.xitu.io/2018/5/20/1637b43a47916b2d?w=350&h=313&f=png&s=30164)

TransactionDefinition接口中的方法如下：
TransactionDefinition接口中定义了5个方法以及一些表示事务属性的常量比如隔离级别、传播行为等等的常量。

我下面只是列出了TransactionDefinition接口中的方法而没有给出接口中定义的常量，该接口中的常量信息会在后面依次介绍到。

## java能不能自己写一个类叫java.lang.System/String

不能自己写以"java."开头的类，其要么不能加载进内存，要么即使你用自定义的类加载器去强行加载，也会收到一个SecurityException。

**1.Bootstrap Loader**（启动类加载器）：加载System.getProperty("sun.boot.class.path")所指定的路径或jar。通过System.out.println(System.getProperty("sun.boot.class.path"));打印，发现主要是“D:\Program Files\Java**\jdk1.6.0_10\jre\lib**”中的jar包。

**2.Extended Loader**（标准扩展类加载器ExtClassLoader）：加载System.getProperty("java.ext.dirs")所指定的路径或jar。在使用Java运行程序时，也可以指定其搜索路径，例如：java -Djava.ext.dirs=d:\projects\testproj\classes HelloWorld。

通过打印System.out.println(System.getProperty("java.ext.dirs"));，可以发现主要加载目录为：

“D:\Program Files\Java\**jdk1.6.0_10\jre\lib\ext**;C:\WINDOWS\**Sun\Java\lib\ext**”

**3、AppClass Loader**（系统类加载器AppClassLoader）：加载System.getProperty("java.class.path")所指定的路径或jar。在使用Java运行程序时，也可以加上-cp来覆盖原有的Classpath设置，例如： java -cp ./lavasoft/classes HelloWorld

## ARP

**ARP（Address Resolution Protocol）即地址解析协议， 用于实现从 IP 地址到 MAC 地址的映射，即询问目标IP对应的MAC地址**。

应用接受用户提交的数据，触发TCP建立连接，TCP的第一个SYN报文通过connect函数到达IP层，IP层通过查询路由表：

　　如果目的IP和自己在同一个网段：

　　当IP层的ARP高速缓存表中存在目的IP对应的MAC地址时，则调用网络接口send函数（参数为IP Packet和目的MAC））将数据提交给网络接口，网络接口完成Ethernet Header + IP + CRC的封装，并发送出去；

　　当IP层的ARP高速缓存表中不存在目的IP对应的MAC地址时，则IP层将TCP的SYN缓存下来，发送ARP广播请求目的IP的MAC，收到ARP应答之后，将应答之中的<IP地址，对应的MAC>对缓存在本地ARP高速缓存表中，然后完成TCP SYN的IP封装，调用网络接口send函数（参数为IP Packet和目的MAC））将数据提交给网络接口，网络接口完成Ethernet Header + IP + CRC的封装，并发送出去；。

　　如果目的IP地址和自己不在同一个网段，就需要将包发送给默认网关，这需要知道默认网关的MAC地址：

　　当IP层的ARP高速缓存表中存在默认网关对应的MAC地址时，则调用网络接口send函数（参数为IP Packet和默认网关的MAC）将数据提交给网络接口，网络接口完成Ethernet Header + IP + CRC

　　当IP层的ARP高速缓存表中不存在默认网关对应的MAC地址时，则IP层将TCP的SYN缓存下来，发送ARP广播请求默认网关的MAC，收到ARP应答之后，将应答之中的<默认网关地址，对应的MAC>对缓存在本地ARP高速缓存表中，然后完成TCP SYN的IP封装，调用网络接口send函数（参数为IP Packet和默认网关的MAC）将数据提交给网络接口，网络接口完成Ethernet Header + IP + CRC的封装，并发送出去。

### ARP的位置

　　OSI模型有七层，TCP在第4层传输层，IP在第3层网络层，而ARP在第2层数据链路层。高层对低层是有强依赖的，所以TCP的建立前要进行ARP的请求和应答。

　　ARP高速缓存表在网络层使用。如果每次建立TCP连接都发送ARP请求，会降低效率，因此在主机、交换机、路由器上都会有ARP缓存表。建立TCP连接时先查询ARP缓存表，如果有效，直接读取ARP表项的内容进行第二层数据包的发送；只有表失效时才进行ARP请求和应答进行MAC地址的获取，以建立TCP连接。

## interrupt()

**interrupt()不能中断在运行中的线程，它只能改变中断状态而已。**

线程在运行中调用interrupt()并不会有任何影响

当一个线程被中断后，在进入wait，sleep，join方法时会抛出异常。是的，这一点也没有错，但是这有什么意义呢?如果你知道那个线程的状态已经处于中断状态，为什么还要让它进入这三个方法呢?当然有时是必须这么做的，但大多数时候没有这么做的理由，所以我上面主要介绍了在已经调用这三个方法的线程上调用interrupt()方法让它从这几个方法的”暂停”状态中恢复过来。这个恢复过来就可以包含两个目的： 
一. [可以使线程继续执行]，那就是在catch语句中执行醒来后的逻辑，或由catch语句 
转回正常的逻辑。总之它是从wait，sleep，join的暂停状态活过来了。 
二. [可以直接停止线程的运行]，当然在catch中什么也不处理，或return，那么就完成了当前线程的使命，可以使在上面”暂停”的状态中立即真正的”停止”。

**操作线程方法时 一定要 Thread.currentThread()** 

```java
public class InterruptionInJava implements Runnable{
    private volatile static boolean on = false;
    public static void main(String[] args) throws InterruptedException {
        Thread testThread = new Thread(new InterruptionInJava(),"InterruptionInJava");
        //start thread
        testThread.start();
        Thread.sleep(1000);
        testThread.interrupt();
        InterruptionInJava.on = true;

        System.out.println("main end");

    }

    @Override
    public void run() {
        while(!on){
            try {
                Thread.sleep(10000000);
            } catch (InterruptedException e) {
                System.out.println("caught exception: "+e);
            }
        }
    }
}
```

![img](https://img-blog.csdn.net/20180806101913339?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgyMzc3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![img](https://img-blog.csdn.net/20180806102145995?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgyMzc3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## Redis 主从复制原理

### 全量复制

Redis全量复制一般发生在Slave初始化阶段，这时Slave需要将Master上的所有数据都复制一份。具体步骤如下： 

- 从服务器连接主服务器，发送SYNC命令； 
- 主服务器接收到SYNC命名后，开始执行BGSAVE命令生成RDB文件并使用缓冲区记录此后执行的所有写命令； 
- 主服务器BGSAVE执行完后，向所有从服务器发送快照文件，并在发送期间继续记录被执行的写命令； 
- 从服务器收到快照文件后丢弃所有旧数据，载入收到的快照； 
- 主服务器快照发送完毕后开始向从服务器发送缓冲区中的写命令； 
- 从服务器完成对快照的载入，开始接收命令请求，并执行来自主服务器缓冲区的写命令；

### 增量复制

Redis增量复制是指Slave初始化后开始正常工作时主服务器发生的写操作同步到从服务器的过程。 
增量复制的过程主要是主服务器每执行一个写命令就会向从服务器发送相同的写命令，从服务器接收并执行收到的写命令。

**注意点**
如果多个Slave断线了，需要重启的时候，因为只要Slave启动，就会发送sync请求和主机全量同步，当多个同时出现的时候，可能会导致Master IO剧增宕机。

```
1）Redis使用异步复制。但从Redis 2.8开始，从服务器会周期性的应答从复制流中处理的数据量。
2）一个主服务器可以有多个从服务器。
3）从服务器也可以接受其他从服务器的连接。除了多个从服务器连接到一个主服务器之外，多个从服务器也可以连接到一个从服务器上，形成一个
   图状结构。
4）Redis主从复制不阻塞主服务器端。也就是说当若干个从服务器在进行初始同步时，主服务器仍然可以处理请求。
5）主从复制也不阻塞从服务器端。当从服务器进行初始同步时，它使用旧版本的数据来应对查询请求，假设你在redis.conf配置文件是这么配置的。
   否则的话，你可以配置当复制流关闭时让从服务器给客户端返回一个错误。但是，当初始同步完成后，需要删除旧的数据集和加载新的数据集，在
   这个短暂的时间内，从服务器会阻塞连接进来的请求。
6）主从复制可以用来增强扩展性，使用多个从服务器来处理只读的请求（比如，繁重的排序操作可以放到从服务器去做），也可以简单的用来做数据冗余。
7）使用主从复制可以为主服务器免除把数据写入磁盘的消耗：在主服务器的redis.conf文件中配置“避免保存”（注释掉所有“保存“命令），然后连接一个配
   置为“进行保存”的从服务器即可。但是这个配置要确保主服务器不会自动重启（要获得更多信息请阅读下一段）
```

### 部分重新同步

```
从Redis 2.8开始，如果遭遇连接断开，重新连接之后可以从中断处继续进行复制，而不必重新同步。
```

 

```
它的工作原理是这样：
主服务器端为复制流维护一个内存缓冲区（``in``-memory backlog）。主从服务器都维护一个复制偏移量（replication offset）和master run ``id` `，
当连接断开时，从服务器会重新连接上主服务器，然后请求继续复制，假如主从服务器的两个master run ``id``相同，并且指定的偏移量在内存缓冲
区中还有效，复制就会从上次中断的点开始继续。如果其中一个条件不满足，就会进行完全重新同步（在2.8版本之前就是直接进行完全重新同步）。
因为主运行``id``不保存在磁盘中，如果从服务器重启了的话就只能进行完全同步了。
```

 

```
部分重新同步这个新特性内部使用PSYNC命令，旧的实现中使用SYNC命令。Redis2.8版本可以检测出它所连接的服务器是否支持PSYNC命令，不支持的
话使用SYNC命令。
```

## 红黑树 和 b+树的用途有什么区别？

1. 红黑树多用在内部排序，即全放在内存中的，STL的map和set的内部实现就是红黑树。
2. B+树多用于外存上时，B+也被成为一个磁盘友好的数据结构。

## 公平锁

公平锁的实现机理在于每次有线程来抢占锁的时候，都会检查一遍有没有等待队列，如果有， 当前线程会执行如下步骤：

if (!hasQueuedPredecessors() &&
    compareAndSetState(0, acquires)) {
    setExclusiveOwnerThread(current);
    return true;
}

## volatile防止重排序

![image](https://user-images.githubusercontent.com/25349066/54355388-f0775500-4693-11e9-8e77-f474651eeb0e.png)

## 消息队列

消息队列常见的使用场景吧，其实场景有很多，但是比较核心的有 3 个：**解耦**、**异步**、**削峰**。

通过一个 MQ，Pub/Sub 发布订阅消息这么一个模型，A 系统就跟其它系统彻底解耦了。

## Redis原子操作

## 并发和并行

并发是同步 多线程来回切换

并行是真正的同时

## 多线程顺序执行

信号量实现同步

```java
public class MultipleThreadRotationUsingSemaphore {
    public static void main(String[] args) {
        PrintABCUsingSemaphore printABC = new PrintABCUsingSemaphore();
        new Thread(() -> printABC.printA()).start();
        new Thread(() -> printABC.printB()).start();
        new Thread(() -> printABC.printC()).start();
    }
}
class PrintABCUsingSemaphore {
    private Semaphore semaphoreA = new Semaphore(1);
    private Semaphore semaphoreB = new Semaphore(0);
    private Semaphore semaphoreC = new Semaphore(0);
    //private int attempts = 0;
    public void printA() {
        print("A", semaphoreA, semaphoreB);
    }
    public void printB() {
        print("B", semaphoreB, semaphoreC);
    }
    public void printC() {
        print("C", semaphoreC, semaphoreA);
    }
    private void print(String name, Semaphore currentSemaphore, Semaphore nextSemaphore) {
        for (int i = 0; i < 10; ) {
            try {
                currentSemaphore.acquire();
                //System.out.println(Thread.currentThread().getName()+" try to print "+name+", attempts : "+(++attempts));
                System.out.println(Thread.currentThread().getName() +" print "+ name);
                i++;
                nextSemaphore.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## 序列化和反序列化

实现序列化接口

两个相同的类 如果序列号编码不对 仍然不可以序列化

## CountDownLatch和CyclicBarrier原理

这个时候，我们应该对于countDownLatch.await()方法是怎么“阻塞”当前线程的，已经非常明白了。其实说白了，就是当你调用了countDownLatch.await()方法后，你当前线程就会进入了一个死循环当中，在这个死循环里面，会不断的进行判断，通过调用tryAcquireShared方法，不断判断我们上面说的那个计数器，看看它的值是否为0了（为0的时候，其实就是我们调用了足够多次数的countDownLatch.countDown（）方法的时候），如果是为0的话，tryAcquireShared就会返回1，代码也会进入到图中的红框部分，然后跳出了循环，也就不再“阻塞”当前线程了。需要注意的是，说是在不停的循环，其实也并非在不停的执行for循环里面的内容，因为在后面调用parkAndCheckInterrupt（）方法时，在这个方法里面是会调用 LockSupport.park(this);，来禁用当前线程的。

CyclicBarrier的关键在于利用ReentrantLock和Condition，每个调用await()方法的线程都被加入到condition队列中进行等待，所有参与线程都调用了await()之后，执行设置的后续动作barrierCommand，再唤醒condition队列中的所有等待线程，重置count，并"更新换代"。



CountDownLatch的计数器只能使用一次，而CyclicBarrier的计数器可以使用reset()方法重置，可以使用多次，所以CyclicBarrier能够处理更为复杂的场景；

CyclicBarrier还提供了一些其他有用的方法，比如getNumberWaiting()方法可以获得CyclicBarrier阻塞的线程数量，isBroken()方法用来了解阻塞的线程是否被中断；

CountDownLatch允许一个或多个线程等待一组事件的产生，而CyclicBarrier用于等待其他线程运行到栅栏位置。

## 访问权限

![img](https://images2015.cnblogs.com/blog/690292/201609/690292-20160923095944481-1758567758.png)

## 当前读&快照读

### 快照读

读取的是记录数据的可见版本（可能是过期的数据)

在事务第一个select形成一张快照

更新版本号 删除版本号  为了实现可重复读

### 当前读

执行update、insert、delete时默认为当前读

会读取最新的记录，锁住相关内容

## 间隙锁

当我们用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据记录的**索引项**加锁；对于键值在条件范围内但并不存在的记录，叫做“间隙（GAP)”，InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁（Next-Key锁）。

举例来说，假如user表中只有101条记录，其empid的值分别是 1,2,...,100,101，下面的SQL：

select * from  user where user_id > 100 for update;
是一个范围条件的检索，InnoDB不仅会对符合条件的user_id值为101的记录加锁，也会对user_id大于101（这些记录并不存在）的“间隙”加锁。

InnoDB使用间隙锁的目的，一方面是为了防止[幻读](https://www.baidu.com/s?wd=%E5%B9%BB%E8%AF%BB&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)，以满足相关隔离级别的要求，对于上面的例子，要是不使用间隙锁，如果其他事务插入了user_id大于100的任何记录，那么本事务如果再次执行上述语句，就会发生幻读；另外一方面，是为了满足其恢复和复制的需要

## RPC

远程过程调用

阿里dubbo

靠分布式网络来调用服务

## Servlet多线程

Java Servlet在服务器中是单例存在的

Servlet容器为了提高响应速度，默认只允许同一个serlvet一个实例存在，对于多个请求使用多线程的方式来处理，避免了实例化servlet时的开销，当有请求到来时，servlet容器会从线程池中取一个空闲的线程，来处理请求，处理完毕后即放回线程池。
 这样对于serlvet的实例变量在使用时需要注意下，实例变量并不是线程安全的，局部变量才是。下面看一个例子。

```java
public class TestServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
       private String message;
    /**
     * @see HttpServlet#HttpServlet()
     */
    public TestServlet() {
        super();
        // TODO Auto-generated constructor stub
    }

    /**
     * @see HttpServlet#doGet(HttpServletRequest request, HttpServletResponse response)
     */
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // TODO Auto-generated method stub
        message = request.getParameter("message");
        String oldMessage = message;
        Thread cur = Thread.currentThread();
        System.err.println("当前threadId="+cur.getId()+" message="+message);
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        System.err.println("oldMesssage="+oldMessage+" message = "+message);
    }

    /**
     * @see HttpServlet#doPost(HttpServletRequest request, HttpServletResponse response)
     */
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // TODO Auto-generated method stub
    }

}
```

https://www.jianshu.com/p/6d013d656b42

## SpringMVC响应请求的原因

Spring MVC框架仍然需要Servlet，但这个Servlet是由Spring框架提供，无需应用开发人员重复实现。

1. 浏览器发出请求，携带着描述用户请求内容的信息。

2. 请求的第一站是Spring的DispatcherServlet（Spring MVC的请求都会通过一个前端控制器Servlet，前端控制器是常用的Web应用程序模式，而DispatcherServlet就是这个前端控制器），它的作用是把请求发送给视图控制器。

3. 在应用程序中通常都有多个控制器，DispatcherServlet需要知道将请求发往哪个控制器，具体的方法就是查询处理器映射（handler mapping），这个处理器映射是通过@RequestMapping方法来建立的：在控制器中通过@RequestMapping（value=”访问路径”，method=GET,POST)规定相应的URL由什么方法来响应，然后

   

4. DispatcherServlet就将请求发给选中的控制器。
   控制器在完成处理以后，会返回一个字符串，这个字符串就是需要响应的页面。处理的过程可能会产生一些信息（model），将这些model添加到request的属性里面，然后发往用户浏览器。

   

5. 控制器返回的视图名并不能定位视图文件，还需要设置视图解析器（ViewResolver），才能完成定位。在视图解析器中设置视图的路径、前缀、后缀等参数，组合起来就是类似“/WEB-INF/views/*.jsp”的字符串，再结合视图控制器传过来的视图名，就明确了视图文件的物理路径，此时才得到真正的视图。

   

6. 视图文件jsp页面解析request传回的信息，例如其中的model，再格式化为html显示，这样，整个MVC流程就完成了。