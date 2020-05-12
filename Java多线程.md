## Java多线程

------

### 一，多线程

线程的创建：使用匿名类，可以创建临时线程

```markdown
一：继承Thread类(可以调用有参构造)
	1.声明一个 类1 继承 Thread类
	2.重写Run方法(根据具体功能)
	3.创建一个 类1 对象
	4.使用 类1对象.start() 启动线程。
二：实现Runable接口
	1.声明一个类2实现 Runable接口
	2.重写Run方法(根据具体功能)
	3.创建一个 类2 对象,
	4.创建Thread类 对象(使用带参构造器，参数为 类2对象)
	5.使用 Thread类对象.start() 启动线程。
三：实现Callable接口( 有返回值 )
	1.声明一个类a实现Callable接口
	2.重写call方法
	3.创建类a的对象
	4.创建一个类FutureTask的对象，以类a的对象为参数
	5.创建一个Thread对象，以FutureTask对象为参数
	(6.) Object object = FutureTask对象.get();  获得call方法返回值
四：线程池：(常见Api:ExecutorService 和 Executors) 
	1.创建一个线程池,ExecutorService 真正的线程池接口;其常见子类 ThreadPoolExecutor，用来设置线程池属性
		ExecutorService executorService = Executors.newFixedThreadPool(10);
		//实际返回的是ThreadPoolExecutor实现类对象,ThreadPoolExecutor service = (ThreadPoolExecutor) 				executorService;	把executorService接口强转化为实现类ThreadPoolExecutor，用实现类					ThreadPoolExecutor对象service对线程池各项属性进行管理
	2.在线程池 添加线程对象
	    executorService.submit(thread_call); //适合 Callable 线程，再调用get 方法得到其返回值
        executorService.execute(thread_run); //适合 Runable 线程。
     3. executorService.shutdown();//关闭线程池
        
        
        
​区别：继承Thread类,对Thread中run方法重写 ; 实现Runable接口，对Runable接口中run方法进行重写

​实现Runable接口好处：
	避免单继承的局限性：java类是单继承的，一旦继承Thread类后，不能继承其他类
	多个线程共享一份数据，适合多个线程处理一份数据 
	
​实现Callable接口好处：
	1.call方法有返回值
	2.可以抛出异常
	3.支持泛型返回值
	4.可以借助FutureTask 得到返回值

​线程池好处：为了避免大量线程创建销毁，影响性能
	1.提高响应速度(线程池已经放入线程，随时用随时取)
	2.降低资源消耗(重复利用线程)
	3.方便对线程的管理 使用ExecutorService的子类ThreadPoolExecutor
		 corePoolSize：核心池的大小 
		 maximumPoolSize：最大线程数 
		 keepAliveTime：线程没有任务时最多保持多长时间后会终止 
```

```java
	class MyThread extends Thread{
        @Override
        public void run() {
            for (int i = 0; i < 100; i++) {
                if (i % 2 != 0) {
                    System.out.println(i);
                }
            }
    	}
        注意：Thread.currentThread().getName()获得当前线程名。
	 		每一个线程对象只能Start一次，可以new多个线程对象。
             start作用：①启动线程 ②调用run方法
             对象.run()  可以启动该程序,但 是在main线程中执行的
            
        ​Thread常用方法：
            1.start 启动线程，在启动run方法
            2.run 在当前线程中 启动run方法
            3.Thread.currentTread() 返回当前线程
            4.Thread.currentTread().getName()=this.getName() 返回当前线程名( 线程中this 返回当前线程)
            5.Thread.currentTread().setName()=this.setName() 设置当前线程名
  				或者调用父类Thread 的带参构造器
                    public 类名(String name){
                        super(name);//调用的父类的带参构造
                   }
        	6.this.yield()=Thread.currentTread().yield()=yield() 释放当前cpu线程使用权(下一次cpu 也有可				能继续条用当前线程)
            7.线程名1.join()  对线程名1进行阻塞
            8.线程名1.sleep(xxx 毫秒)  线程阻塞时间;报异常用 try-catch 捕捉异常，不能用throws抛出异常：因为
                重写是抛出异常不能比父类大，Thread 中run方法没抛出异常是最小的抛出异常
            9.线程名1.isAlive()  线程是否存活: Boolean 类型
                
		​线程优先级(  从概率方面  )
                Thread.MAX_PRIORITY  10
                Thread.MIN_PRIORITY  1
                Thread.NORM_PRIORITY 5
                或者 具体的数字
                线程对象.getPriority()  // 获得线程对象优先级
                线程对象.setPriority   // 设置线程对象优先级
```

线程安全

​	线程五种状态

<img src="C:\Users\art\AppData\Roaming\Typora\typora-user-images\image-20200504185101134.png" alt="image-20200504185101134" style="zoom: 67%;" />

### 线程安全

##### 	<span style='color: #228B22;font-size:20px;'>线程同步：降低程序执行效率</span>

```}java
同步代码块
    
1.解决 Runable 接口同步问题
class  Thread_run implements Runnable{

    private int tick = 100;// 实现Runable 接口 中一切变量 为共享代码块
    Object object = new Object();
    @Override

    public  void run() {
        while (true){
            //此处object 可替换为 this  / Thread_run.class
            synchronized (object) {
                if (tick > 0) {
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + "卖出票：" + tick);
                    tick--;
                } else {
                    break;
                }
            }
        }
    }
}
2.解决 Thread类的同步问题  
class Thread_ex extends Thread{
    private static Object object = new Object();// 锁 一定用static 修饰，保证为同一把锁
    private static int tick = 100;// 继承Thread  使用static  修饰表示为共享数据
    @Override
    public void run() {
        while (true){
           //此处object （不可替换为 this） 可替换为Thread_ex.class/ 慎用this作为锁(this可能不是同一把锁)
            synchronized (object) {
                if (tick > 0) {
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + "卖出票：" + tick);
                    tick--;
                } else {
                    break;
                }
            }
        }
    }
}
    同步方法：当共享数据 完全处于一个方法时可，以声明为同步方法
    
说明：1.操作共享数据的代码块，即同步代码 -->不能太多，也不能太少
      2.共享数据即，多个线程操作变量
      3.同步监视器，即锁。任何一个类的对象都可以充当锁
     要求：多个线程必须共用同一把锁
   Runable接口同步信号量，可以考虑用 this
        
​同步方法 
        
1.解决 Runable 接口同步问题	
class Tick_1 implements Runnable {
    private int tick_num = 100;
    @Override
    public void run() {
        while (tick_num>0) {
            Comm();
        }
    }
    private synchronized  void Comm() {//同步方法
        if (tick_num > 0) {
            System.out.println(Thread.currentThread().getName() + "卖票" + tick_num);
            tick_num--;
        }
    }
}    
2.解决 Thread类的同步问题  
   class ticket extends Thread{
    private static int tick_num = 100;
    @Override
    public void run() {
        while (tick_num>0){
            Fun();
        }
    }
    private static synchronized void Fun(){//同步方法
        if (tick_num>0){
            System.out.println("线程：" + Thread.currentThread().getName() + ";买票" + tick_num);
            tick_num -- ;
        }
    }
    public ticket(String name){
        super(name);
    }
}

同步方法总结：
    1.  同步方法仍然涉及同步信号量，只是不显示出现
    2.	静态方法的同步信号量为 当前类本身.class(静态中不能调this)
   		非静态方法的同步心号量为  this(慎用)
        
 ​同步锁  jdk5.0出现的 
    class Thread_lock implements Runnable{
    private int tick = 100;
      //  实现Runable 接口  这个锁本来就是 公用的
        // lock 锁，必须共用
    private ReentrantLock reentrantLock = new ReentrantLock();
    @Override
    public void run() {
        while (true) {
            try {
                reentrantLock.lock();
                if (tick > 0) {
                    System.out.println(Thread.currentThread().getName() + "卖票,票号：" + tick);
                    tick--;
                } else {
                    break;
                }
            }finally {
                reentrantLock.unlock();
            }
        }
    }
}

synchronized 与 同步锁 有何异同？
    同：都是为了解决线程安全
    异：synchronized自动上锁解锁 ; 同步锁手动上锁,手动解锁,且jvm调用线程花费时间更少，性能好
```

## 	<span style='color:red;font-size:20px;'>单例模式    -->   改为线程安全</span>

```java
class Fun{

    private static Fun instance = null;
    private Fun(){}
    //线程安全 第一方法 效率低
//    public  static Fun getInstance(){
//        synchronized(Fun.class) {
//            if (instance == null) {
//                instance = new Fun();
//            }
//            return instance;
//        }
//    }
//    //线程安全 第一方法 效率高 ，后来的对象申请时 发现instance 不为null 直接返回
    public static Fun getInstance() {
            if (instance == null) {
                synchronized (Fun.class) 
                {
                    if (instance == null) 
                    {
                        instance = new Fun();
                    }
                }
            }
            return instance;
    }
}

```

​	<span style='color:red;font-size:20px;'>线程通信</span>

```java
public void run() {
        synchronized (this) {
            while (true) {
                notify();//唤醒优先级高的线程
                if (number <= 100) {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + ":" + number);
                    number++;

                    try {
                        wait();//使当前线程，进入堵塞状态
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                } else {
                    break;
                }
            }
        }
    }

wait(),notify(),notifyAll() = this.wait(),this.notify(),this.notifyAll()
    wait() 使当前线程进入阻塞状态， 并释放同步监视器                 	 运行态-->阻塞态
    notify() 唤醒一个被wait的线程，若有多个线程，唤醒优先级就最高的一个    阻塞态-->就绪态
    notifyAll() 唤醒所有被wait的进程
 说明：
    1.wait(),notify(),notifyAll()必须在 同步代码块 或者 同步方法 中使用,否则报异常
    2.wait(),notify(),notifyAll()的调用者 即 this  必须和 同步代码块 或者 同步方法的监视器一样
    3.wait(),notify(),notifyAll()是定义在 Object 中
    
 面试题：wait() 与 sleep()的异同
    相同点：都使当前进程变为阻塞态
    不同店： 1. wait定义在Object类中，sleep定义在Thrad中
    	    2. 调用需求不同,sleep可以在任何需要的情况下调用， wait必须在 同步代码块 或 同步方法 中调用
    		3. 关于是否释放监视器， 如果两者在同步代码块 或 同步方法中调用
    			wait 释放监视器; sleep 不会释放监视器
```

