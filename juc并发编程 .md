# 一。进程和线程

## **进程：**

进程可以看作是程序运行的一个实例，进程用来加载指令到CPU，将数据加载到内存，管理IO的。

大部分的程序可以开启多个进程的实例，例如：记事本，画图本

但是有一部分只能开启一个进程实例，例如：QQ音乐，360管家

进程就是用来**分配资源**的，可以看作 **一个正在运行的程序**

## **线程：**

一个线程是一个指令流，每一条指令流以一定的顺序交给CPU执行

一个进程可以包含多个线程



# 二。并行和并发

## **并发：**

某段时间内，宏观上所有的任务同时执行，但微观上是交替执行的，比如CPU循环交替的执行每个任务，但是循环交替的太快了，所以感知不到在交替运行。

比如：一个人先吃饭，再写作业，在吃饭，耗时0.1s，最终完成这两件事，因为循环的做的速度太快了，就好像同时在做这两件事，但实际上是交替在做

单核CPU执行多个任务的时候都是并发执行的。

## **并行：**

真正的同一时刻执行多个任务，比如：一个人左手吃饭，同时右手写作业，这两件事就是实打实的同时在做。

多核CPU能够实现并行。

## 并发和并行：

但是大多数情况是并发和并行都有，比如产生资源的竞争，雇主做饭+休息，保姆做饭+照顾孩子，因为只有一口锅，所以做饭这件事情得交替执行，这是并发，但是休息和照顾孩子可以各做各的，这是并行。



# 三。同步和异步

## 同步：

需要等待结果返回才能继续进行，就是同步。

## 异步：

不需要等待结果返回就继续执行，就是异步

多线程可以让方法执行变成异步的， 比如读取磁盘文件要花费5s，如果没有线程调度机制，那这5s内调用者只能等着，什么也做不了。

## 提升效率：

多核CPU的优势，可以开启多个线程提高运行效率。

比如：有4个计算，如果是单核的那就只能串行执行，总时间就为4*每个计算运行的时间

但是**多核CPU**下，可以开启4个线程**并行**计算，总时间就为最耗时的那个计算时间

单核CPU即使使用多线程，和单线程模式差别不大，因为是单核的，一次仍然只是一个线程在计算

但是多核CPU的情况下，速度就有差别了。

## 结论：

单核CPU下，多线程并不能实际提升效率，知识为了能够在不同的任务之间切换，不同的线程轮流使用CPU，不至于一个线程一直占用CPU。



# 四。Java线程

Java程序只要一启动都会有一个线程，也就是主线程。

主线程之外可以额外创建新的线程。

## 创建线程的方法：

### 1。new Thread

通过 new Thread 创建线程的实列，并配合匿名内部类实现 Runnable 接口当中的 run 方法。

Runnable是一个接口，这个接口当中定义了一个抽象方法 run，没有具体实现，他代表线程将来需要具体执行什么样的任务。

实现的时候可以使用匿名内部类来简化这个接口的实现。

创建线程之后，需要调用 **start** 方法来开启这个线程。

因为创建完线程对象之后，这个线程还未启动。

调用start方法之后，JVM 才会创建一个新的操作系统线程去执行任务。

```java
@Slf4j
public class ThreadTestDemo1 {
    public static void main(String[] args) {
        // 创建一个新的线程对象
        Thread t1 = new Thread() {
            @Override
            public void run() {
                log.debug("副线程 is running...");
            }
        };

        // 为副线程命名
        t1.setName("t1");
        // 启动线程，此时jvm才会创建一个新的操作系统线程去执行任务
        t1.start();

        // 主线程执行任务
        log.debug("主线程 is running...");
    }
}
```

执行结果：

主线程 和  副线程是并行的执行任务的

![](juc并发编程.assets/屏幕截图 2026-03-16 182550.png)



### 2。Runnable 和 Thread 分开创建

可以先创建Runable的对象，指定执行的任务。

然后再创建线程对象。

这种方式更灵活。

```java
@Slf4j
public class ThreadTestDemo2 {
    public static void main(String[] args) {
        // 创建任务对象
        Runnable r = new Runnable() {
            @Override
            public void run() {
                log.debug("副线程 is running...");
            }
        };

        // 创建线程对象，构造函数接接受任务对象 和 线程名名称
        Thread t = new Thread(r,"t1");

        // 开启线程
        t.start();

        // 主线程
        log.debug("主线程 is running...");
    }
}
```

执行结果一样：

主线程 和 副线程都是并行执行的。

![](juc并发编程.assets/屏幕截图 2026-03-16 183240.png)



### 3。简化

可以用lambda表达式简化匿名内部类的书写。

```java
@Slf4j
public class ThreadTestDemo2 {
    public static void main(String[] args) {
        // 创建任务对象
        Runnable r = () -> {log.debug("副线程 is running...");};

        // 创建线程对象，构造函数接接受任务对象 和 线程名名称
        Thread t = new Thread(r,"t1");

        // 开启线程
        t.start();

        // 主线程
        log.debug("主线程 is running...");
    }
}
```



### 4。原理

方法1：

创建了Thread类的一个匿名子类，并且重写了run方法，线程任务是直接写在匿名子类中的。

方法2：

先实现Runnable接口，重写run方法。

再将Runnable对象传递给Thread的构造函数。

线程任务和Thread对象是分离的。



执行流程是这样的：

Thread本身重写了Runnable接口中的run方法：

```java
@Override
public void run() {
    if (target != null) {
        target.run();
    }
}
```

方法1：

```
因为是直接通过匿名子类实现了接口，重写了run方法，所以没走构造函数
t1.start() → 调用Thread子类的run() → 直接执行重写的run()方法
```

方法2：

```
调用构造函数，判断target也就是传入的Runnable对象是否为空，不为空就调用实现的run方法
t.start() → 调用Thread.run() → 检查target是否为null → 调用target.run()
```

方法2更推荐使用



### 3。FutureTask 和  Thread  创建线程

FutureTask封装了Callable，Callable与Runnable不同的是，Callable定义的线程方法可以有返回值

```java
public interface Callable<V> {
    V call() throws Exception;
}
```

 所以FutureTask同样可以定义有返回值的线程任务

```java
@Slf4j
public class ThreadTestDemo3 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 创建任务对象
        FutureTask<Integer> task = new FutureTask<>(() -> {
            log.debug("副线程 is running...");
            Thread.sleep(2000);
            return 200;
        });

        // 创建线程对象
        Thread t = new Thread(task,"t1");
        // 开启副线程
        t.start();

        // 主线程等副线程执行完毕拿到返回值
        log.debug("{}",task.get());
    }
}

```

它支持主线程阻塞获取结果，阻塞直到任务完成返回结果

![](juc并发编程.assets/屏幕截图 2026-03-16 212917.png)



# 五。线程运行原理

## 栈与栈帧

JVM由堆，栈，方法区组成，栈内存就是用来给线程用的。

每个线程启动之后，JVM就会为其分配一块栈内存。

每个栈由多个栈帧组成，对应着每次方法调用时所占用的内存。

每个线程只能有一个活动栈帧，对应着当前正在执行的任务的那个方法。

```java
public class ThreadTestDemo4 {
    public static void main(String[] args) {
        method1( 10);
    }

    private static void method1(int x) {
        int y = x + 1;
        Object m = method2();
        System.out.println(m);
    }

    private static Object method2() {
        Object n = new Object();
        return n;
    }
}
```

比如这段程序，主线程先调用main方法，就会由一个栈分配给主线程，调用m1，m2时又会分配。

以DeBug模式运行。

![](juc并发编程.assets/屏幕截图 2026-03-16 220646.png)



## 线程运行原理

```java
public class ThreadTestDemo4 {
    public static void main(String[] args) {
        method1( 10);
    }

    private static void method1(int x) {
        int y = x + 1;
        Object m = method2();
        System.out.println(m);
    }

    private static Object method2() {
        Object n = new Object();
        return n;
    }
}
```

这段代码执行的原理：

**类加载**：

jvm的内存模型当中有一块区域：**方法区**，用来存放**类信息**，编译之后的**字节码**，**静态常量**等。

程序一启动，会把类的信息，字节码等加载到方法区。

<img src="juc并发编程.assets/屏幕截图 2026-03-17 210654.png" style="zoom: 67%;" />

**JVM**中的另一块内存区域：**栈**

每个线程一启动，就在**栈**中分配到一块**虚拟机栈**，这是**线程私有**的，每个线程都有自己的栈。

这块栈的**生命周期和线程一致**：线程创建 -> 栈创建；线程结束 -> 栈销毁。

而栈里面又会存储一个个**栈帧**，线程中只要有方法调用就产生一个栈帧，只要线程每次调用了一个方法，就会在栈里创建一个栈帧。

每个线程只有一个**活动栈帧**，对应着正在调用的方法

线程
 └── JVM栈（线程私有）
       ├── 栈帧（方法A）
       ├── 栈帧（方法B）
       └── 栈帧（方法C）

**栈帧**中包含：

1.**局部变量表**：存放 方法参数 ，方法内定义的的局部变量，对象引用（对象本身在堆内存中，所以存放的是引用），基本数据类型的值。

2.**返回地址**：方法执行完成之后，返回到调用方法的位置。

3.**锁记录**：

4.**操作数栈**：用于方法执行过程中进行计算，例如：

```java
int c = a + b;

执行过程：

a 入栈
b 入栈
相加
结果出栈
```

图解：

每个线程的栈内存是私有的

![](juc并发编程.assets/v2-47e25488fbf2221a258292bd264406d6_1440w.jpg)

栈帧：

![v2-5911e96a8958e23e0707a818a1ccaa6d_1440w](juc并发编程.assets/v2-5911e96a8958e23e0707a818a1ccaa6d_1440w.jpg)

上面代码的栈内存：

<img src="juc并发编程.assets/屏幕截图 2026-03-17 213213.png" style="zoom:67%;" />



## 线程上下文切换 Thread Context Switch

任务调度器把时间片分给线程之后，线程去使用CPU计算，假如时间片内线程没有执行完任务或者线程被阻塞了，就要把CPU让出来交给另一个线程使用，那交换的过程中操作系统得记录原先的线程执行到哪一步了，之后又轮到这个线程的时候才能接着执行任务。

线程上下文就是线程执行过程中需要保存和恢复的一组信息，它保证了线程被中断之后恢复运行。

发生Thread Context Switch，操作系统会保存当前线程的状态信息，并且恢复另一个线程的状态信息。

Java中的程序计数器会记录下一条程序指令的地址。

比如：main线程被挂起，执行t1线程

<img src="juc并发编程.assets/屏幕截图 2026-03-17 221647.png" style="zoom:67%;" />



# 六。线程方法

## 1。run 和 start

new了一个线程之后，首先进入初始状态，然后调用 **start** 方法来到就绪状态，这里不会立即执行线程任务，而是等待系统资源分配，分配到时间片后，才去执行。

如果直接调用 **run** 方法，实际上**并没有启动一个新的线程去执行任务**，还是**主线程执行**。

下面直接调用**run**方法

```java
@Slf4j
public class ThreadTestDemo5 {
    public static void main(String[] args) {
        // 创建新的线程
        Thread t = new Thread("t1"){
            @Override
            public void run() {
                log.debug("线程 is running...");
            }
        };

        // 直接调用run方法
        t.run();

        log.debug("线程 is runing...");
    }
}

```

执行结果：都是主线程执行任务

<img src="juc并发编程.assets/屏幕截图 2026-03-18 142606.png" style="zoom:67%;" />

**run**是执行的任务，直接调用会被当作是main线程下的普通方法执行。

**start**会去创建一个新的线程，并且去执行**run**方法



## 2。getState( ) 线程状态 

1.**初始 NEW** ：new ，创建了一个线程对象，还没有调用start

2.**运行 RUNNABE **：就绪和运行中都是运行态（Java中）

线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取CPU的使用权，此时处于就绪状态（ready）。就绪状态的线程在获得CPU时间片后变为运行中状态（running）。

3.**阻塞 BLOCKED**：线程阻塞于锁。

4.**等待** **WAITING**：线程需要等待其他线程做出一些动作。

5.**超时等待** **TIMED_WAITING**：可以在指定的时间自行返回，于等待不同。

6**.终止** **TERMINATED**：线程执行完毕。

getState（）能获取线程当前状态。

```java
@Slf4j
public class ThreadTestDemo5 {
    public static void main(String[] args) {
        // 创建新的线程
        Thread t = new Thread("t1"){
            @Override
            public void run() {
                log.debug("线程 is running...");
            }
        };

        log.debug(t.getState().toString());

        t.start();

        log.debug(t.getState().toString());
    }
}
```

<img src="juc并发编程.assets/屏幕截图 2026-03-18 150740.png" style="zoom:67%;" />



start 方法不能被多次调用，如果多次调用会抛出异常：
start 方法内部首先会判断线程状态，如果不是 “NEW” 状态，就会抛异常。

因为第一次调用start方法，线程状态变为运行态。

再次调用判断不为“NEW”，所以抛异常。

<img src="juc并发编程.assets/屏幕截图 2026-03-18 151113.png" style="zoom:67%;" />

这个0代表就代表 “NEW”

<img src="juc并发编程.assets/屏幕截图 2026-03-18 151208.png" style="zoom:67%;" />

多次调用抛异常。



## 3。sleep

sleep方法能让线程状态从 **RUNNABE **变为 **TIMED_WAITING**

```JAVA
@Slf4j
public class ThreadTestDemo5 {
    public static void main(String[] args) throws InterruptedException {
        // 创建新的线程
        Thread t = new Thread("t1"){
            @Override
            public void run() {
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        };

        t.start();

        // 副线程处于 Runnable 状态
        log.debug("副线程状态：{}",t.getState());

        Thread.sleep(500);

        // 副线程处于 TIMED_WAITING 状态
        log.debug("副线程状态：{}",t.getState());
    }
}
```

这里让主线程休眠一会是因为，直接调用 t.getState(）的话，副线程还没进入等待状态。

![](juc并发编程.assets/屏幕截图 2026-03-18 215559.png)



## 4。interrupt 打断 sleep

其他线程使用interrupt方法可以打断正在睡眠的线程，这时候 sleep 方法会抛出InterruptedException异常。

```java
@Slf4j
public class ThreadTestDemo6 {
    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread("t1") {
            @Override
            public void run() {
                log.debug("副线程进入睡眠");
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    log.debug("副线程被中断");
                    throw new RuntimeException(e);
                }
            }
        };

        t.start();

        Thread.sleep(1000);
        log.debug("主线程中断副线程睡眠");

        // 主线程中断副线程
        t.interrupt();
    }
}
```

![](juc并发编程.assets/屏幕截图 2026-03-18 221649.png)

但是睡眠结束之后的线程可能未必立即继续执行，因为CPU可能正在执行其他线程，得等其他线程执行完，这个线程再继续执行。



## 5。TimeUnit 的 sleep

TimeUnit 的sleep方法也可以使线程睡眠，但是可读性更高。

```java
@Slf4j
public class ThreadTestDemo7 {
    public static void main(String[] args) throws InterruptedException {
        log.debug("睡眠之前");
        TimeUnit.SECONDS.sleep(1);
        log.debug("睡眠之后");
    }
}
```

![](juc并发编程.assets/屏幕截图 2026-03-18 222354.png)



## 6。yield 和 线程优先级

**yield：**

调用 yield 可以使线程从 Running 变为 Runnable就绪态 ，让出CPU，任务调度器去执行其他线程。

但是他依赖于操作系统的任务调度器。假如一个线程让出了CPU，但是又没有其他线程被任务调度器调用，那这个线程又会变为Running。

**线程优先级：**

Thread中可以为线程设置优先级，线程优先级越高，越先提示任务调度器调用这个线程，但这只是提示，不能百分百保证线程一定被调用。

CPU比较忙，优先级越高的线程会获得更多的时间片；但是CPU闲的时候，优先级没啥用。

```java
    /**
     * The minimum priority that a thread can have.
     */
    public static final int MIN_PRIORITY = 1;

    /**
     * The default priority that is assigned to a thread.
     */
    public static final int NORM_PRIORITY = 5;

    /**
     * The maximum priority that a thread can have.
     */
    public static final int MAX_PRIORITY = 10;
```



yield：线程2让出时间片，那他执行的次数肯定不如线程1多，所以数字不如线程1大。

```java
public class ThreadTestDemo8 {
    public static void main(String[] args) {
        Runnable task1 = new Runnable() {
            @Override
            public void run() {
                int count = 0;
                while (true) {
                    System.out.println("线程1：" + count++);
                }
            }
        };

        Runnable task2 = new Runnable() {
            @Override
            public void run() {
                int count = 0;
                while (true) {
                    // 让出时间片
                    Thread.yield();
                    System.out.println("线程2：" + count++);
                }
            }
        };

        Thread t1 = new Thread(task1,"t1");
        Thread t2 = new Thread(task2,"t2");

        t1.start();
        t2.start();
    }
}
```

![](juc并发编程.assets/屏幕截图 2026-03-18 224649.png)



优先级：

线程2的优先级高于线程1，那他分到的时间片就多，执行的次数就多，数就大。

```java
public class ThreadTestDemo8 {
    public static void main(String[] args) {
        Runnable task1 = new Runnable() {
            @Override
            public void run() {
                int count = 0;
                while (true) {
                    System.out.println("线程1：" + count++);
                }
            }
        };

        Runnable task2 = new Runnable() {
            @Override
            public void run() {
                int count = 0;
                while (true) {
                    System.out.println("            线程2：" + count++);
                }
            }
        };

        Thread t1 = new Thread(task1,"t1");
        Thread t2 = new Thread(task2,"t2");

        t1.setPriority(Thread.MIN_PRIORITY);
        t2.setPriority(Thread.MAX_PRIORITY);

        t1.start();
        t2.start();
    }
}

```

![](juc并发编程.assets/屏幕截图 2026-03-18 225042.png)

yield和优先级并不能真正的控制线程的执行，只是给任务调度器提示而已。



## 7。防止CPU占用100%

如果没有使用CPU计算，while（true）会使CPU陷入空转浪费CPU，可以使用sleep或者

yield让出CPU给其他程序。

```java
public class ThreadTestDemo9 {
    public static void main(String[] args) throws InterruptedException {
        while (true) {
            Thread.sleep(50);
        }
    }
}
```



## 8。join

join可以让一个线程同步等待另一个线程。

```java
public class ThreadTestDemo10 {
    static int r = 0;

    public static void main(String[] args) {
        test1();
    }

    private static void test1() {
        log.debug("主线程调用test1()");

        Thread t1 = new Thread("t1") {
            @Override
            public void run() {
                log.debug("副线程开始执行");
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    log.debug("副线程被中断");
                }
                log.debug("副线程执行结束");
                r = 10;
            }
        };

        t1.start();
        log.debug("r 的结果为：{}",r);
        log.debug("主线程结束");
    }
}
```

按理来说，应该打印 r=10，但是r=0

![](juc并发编程.assets/屏幕截图 2026-03-19 172008.png)

因为主线程和副线程是并行执行的，t1线程需要在1s之后将r设置为10。

而主线程一开始就打印，所以只能打印r=0。



使用**join**：

join可以让线程等其他待线程结束，A线程想要B线程结束之后在执行，可以在A线程中通过B线程调用join方法，实现同步。

<img src="juc并发编程.assets/屏幕截图 2026-03-19 205048.png" style="zoom: 67%;" />

```java
@Slf4j
public class ThreadTestDemo10 {
    static int r = 0;

    public static void main(String[] args) throws InterruptedException {
        test1();
    }

    private static void test1() throws InterruptedException {
        log.debug("主线程调用test1()");

        Thread t1 = new Thread("t1") {
            @Override
            public void run() {
                log.debug("副线程开始执行");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    log.debug("副线程被中断");
                }
                log.debug("副线程执行结束");
                r = 10;
            }
        };

        t1.start();

        // 等待副线程执行结束
        t1.join();
        log.debug("r 的结果为：{}",r);
        log.debug("主线程结束");
    }
}
```

![](juc并发编程.assets/屏幕截图 2026-03-19 182244.png)



### 有时效地join

join方法可以传入一个时间参数，表示最多等待线程多长时间，超过这个时间就不等了。

比如：副线程要sleep 2s，但是主线程只等它1.5s，结果就是 r=0.

```java
![屏幕截图 2026-03-19 211143](../../Pictures/Screenshots/屏幕截图 2026-03-19 211143.png)@Slf4j
public class ThreadTestDemo11 {
    static int r = 0;
    public static void main(String[] args) throws InterruptedException {
        test2();
    }

    private static void test2() throws InterruptedException {
        Thread t1 = new Thread("t1") {
            @Override
            public void run() {
                try {
                    // 休眠2s
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                r = 10;
            }
        };

        t1.start();
        log.debug("join 开始");
        // 只等1.5s
        t1.join(1500);
        log.debug("join 结束，r = {}",r);
    }
}
```

![](juc并发编程.assets/屏幕截图 2026-03-19 211143.png)



## 9。interrupt详解

interrupt可以打断sleep，wait，join

### 打断阻塞

线程如果被打断，打断之后会有一个打断标记，是个boolean值，如果被打断，就是真；否则就是假。

但是sleep，wait，join被打断之后，会抛出InterruptedException异常，此时会把这个打断标记清空，也就是置为假。

调用isIntuerrupted（）方法可以查看打断标记，并且不会重置打断标记。

```java
@Slf4j
public class ThreadTestDemo12 {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        });

        t1.start();
        log.debug("开始打断...");
        Thread.sleep(1000);
        t1.interrupt();
        log.debug("打断标记：{}",t1.isInterrupted());
    }
}

```

![](juc并发编程.assets/屏幕截图 2026-03-19 213504.png)



### 打断正常线程

如果调用interrupt打断一个正常运行的线程，`interrupt()` 方法**不会强制停止线程**，只是设置一个**中断标志位**，提醒线程应该被打断。

```java
@Slf4j
public class ThreadTestDemo13 {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            while (true) {

            }
        }, "t1");

        t1.start();
        Thread.sleep(1000);

        log.debug("begin interrupt t1");
        t1.interrupt();
    }
}
```

可以看到程序并没有停止。

但是可以配合中断标记优雅的中断线程。

```java
@Slf4j
public class ThreadTestDemo13 {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            while (true) {
                boolean interrupted = Thread.currentThread().isInterrupted();
                if (interrupted) {
                    log.debug("t1被提醒要中断");
                    break;
                }
            }
        }, "t1");

        t1.start();
        Thread.sleep(1000);

        log.debug("begin interrupt t1");
        t1.interrupt();
    }
}
```

![](juc并发编程.assets/屏幕截图 2026-03-19 220641.png)



## 10。interrupted

interrupted测试当前线程是否被中断（检查中断标志），返回一个boolean并清除中断状态，第二次再调用时中断状态已经被清除，将返回一个false。

// interrupted和interrupt都可以打断线程，但是interrupted打断之后会重置打断标记，也就是设为false；但是interrupt不会重置打断标记。//

```java
@Slf4j
public class Demo2 {
    public static void main(String[] args) throws InterruptedException {

        Thread t = new Thread(() -> {
            Thread currentThread = Thread.currentThread();
            
            // 第一次检查
            boolean interrupted1 = Thread.interrupted();
            log.debug("第一次检查：{}", interrupted1);  // false
            
            // 主线程中断
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            
            // 第二次检查（被中断后）
            boolean interrupted2 = Thread.interrupted();
            log.debug("第二次检查：{}", interrupted2);  // true
            
            // 第三次检查（标志位已被清除）
            boolean interrupted3 = Thread.interrupted();
            log.debug("第三次检查：{}", interrupted3);  // false
        }, "t1");

        t.start();
        Thread.sleep(500);
        t.interrupt();  // 中断线程
    }
}
```

![](juc并发编程.assets/屏幕截图 2026-03-20 180653.png)

## 11。两阶段终止模式 （一）

stop方法可以用来停止线程，但是这个方法太暴力了，如果这个线程锁住了某些资源，直接杀死它会使得其他线程获取不到锁拿不到资源，陷入死锁的问题。

所以需要在线程终止之前让他完成必要的清理和资源释放操作，以避免出现资源泄漏和数据损坏等问题。

两阶段终止模式是一种优雅的终止线程的方式，它能够给被终止的线程一个料理后事的机会，而不是一剑封喉。

**阶段一**：线程T1向线程T2发送终止指令，比如 interrupt 设置打断标记，这一步只是提示我要终止你了，不会真的去终止线程。

**阶段二**：线程T2去响应终止指令，完成一些后事料理工作，终止线程。

![](juc并发编程.assets/屏幕截图 2026-03-19 230605.png)

完整的流程图：

比如一个监控系统，

![](juc并发编程.assets/fe77b0e2af925c882a73e027bb3d4594.png)

这里有个细节：

interrupt终止的场景会出现在两个地方：

（1）**runnable**：线程处于正常运行的状态 runnable，这时候调用interrupt会将打断标记设置为true，代表要终止线程。

（2）**sleep**：如果此时线程T2处于sleep状态，调用interrupt会抛出InterruptedException异常，并且重置打断标记为false。这个时候就需要再次调用interrupt将打断标记设置为true，否则下次循环判断打断标记为false的时候就又会执行监控了，跳过了终止线程的环节。

```java
@Slf4j
public class ThreadTestDemo14 {
    public static void main(String[] args) throws InterruptedException {
        TwoPhaseTermination twoPhaseTermination = new TwoPhaseTermination();
        twoPhaseTermination.start();

        Thread.sleep(3500);

        //停止监控
        twoPhaseTermination.stop();
    }
}

@Slf4j
class TwoPhaseTermination {
    private Thread monitor;

    // 启动监控线程
    public void start() {
        monitor = new Thread(() -> {
            // 进行监控
            while (true) {
                // 获取当前线程
                Thread currentThread = Thread.currentThread();
                // 如果当前线程被打断，就退出监控
                if (currentThread.isInterrupted()) {
                    log.debug("monitor线程被中断，要去料理后事");
                    break;
                }
                try {
                    Thread.sleep(1000);
                    log.debug("执行监控记录");
                } catch (InterruptedException e) {
                    // 如果在sleep过程中被打断，重新设置打断标记
                    currentThread.interrupt();
                    e.printStackTrace();
                }
            }
        });
        //启动线程
        monitor.start();
    }
    
    // 停止监控线程
    public void stop() {
        monitor.interrupt();
    }
}
```



## 12。案列

泡茶：分为洗水壶，烧开水，洗茶壶，洗茶杯，拿茶叶等步骤。最耗时的操作是烧开水。

最快的执行流程肯定是：先烧开水，烧水的同时去做洗茶壶，洗茶杯这些操作。

而不是先洗茶杯，系茶壶，最后烧开水这种串行操作。

![](juc并发编程.assets/屏幕截图 2026-03-21 162539.png)

```java
@Slf4j
public class ThreadTestDemo16 {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            log.debug("洗水壶");
            sleep(1);
            log.debug("烧开水");
            sleep(5);
        },"老王");

        Thread t2 = new Thread(() -> {
            log.debug("洗茶壶");
            sleep(1);
            log.debug("洗茶杯");
            sleep(1);
            log.debug("拿茶叶");
            sleep(1);

            // 等待烧开水
            try {
                t1.join();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }

            log.debug("泡茶");
        },"小王");

        t1.start();
        t2.start();
    }

    public static void sleep(int seconds)  {
        try {
            Thread.sleep(seconds * 1000L);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}

```

```cmd
2026-03-21 16:35:48.140 [小王] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo16 - 洗茶壶
2026-03-21 16:35:48.140 [老王] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo16 - 洗水壶
2026-03-21 16:35:49.144 [老王] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo16 - 烧开水
2026-03-21 16:35:49.144 [小王] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo16 - 洗茶杯
2026-03-21 16:35:50.155 [小王] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo16 - 拿茶叶
2026-03-21 16:35:54.146 [小王] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo16 - 泡茶
```



# 七。主线程和守护线程

Java程序默认情况下，只有所有线程都结束运行，程序才结束。只要有一个线程没结束运行，程序都不会停止运行。

守护线程：当其他的非守护线程执行完，即使守护线程中的代码没执行完，也会被强制结束。

线程默认为非守护线程，可通过 t.setDaemon(true);设置为守护线程。





# 共享模型之管程





# 一。线程安全问题

假设有两个线程，一个共享变量 n=0，

线程一的任务：对n进行+1操作，

线程二的任务：对n进行-1操作，

每次他们进行操作时，需要到内存中读取n，之后进行运算操作，最终写回到内存，这是每个线程执行的流程。

按理来说，线程一执行一次+1操作，n变为1；线程二执行一次-1操作，n又变回0，n最终的值应该始终为0。

但是就在线程一执行完+1操作，还未将n写回到内存中时，线程一发生了阻塞，可能是时间片用完了给了线程二，也可能是IO阻塞，总之没把n写回内存，并且此时线程一中的n=1。

此时轮到线程二读n，因为线程一还未写回，所以此时共享变量n还是为0，线程二执行-1操作，n=-1，之后写回内存。

就在这时，线程一恢复运行，继续执行未完成的操作，把n=1写回内存当中，结果把线程二的结果覆盖了。

最后，n=1而不是0。

发生了线程安全问题，但是实际上两个线程都没做错。

```java
@Slf4j
public class ThreadTestDemo17 {
    static int count = 0;

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                count++;
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                count--;
            }
        });

        t1.start();
        t2.start();

        t1.join();
        t2.join();

        log.debug("count = {}",count);
    }
}
```

```cmd
2026-03-21 17:37:52.768 [main] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo17 - count = 1986
```



## 临界区

如果一段代码块内有对**共享资源**的读写操作，那这段代码就是**临界区**。

```java
public void increment() {
	count++;
}
public void decrement() {
	count--;
}
```

## 竞态条件

多个线程在临界区内执行，由于代码的**执行序列不同**而导致结果无法预测，这就是竞态条件。



# 二。synchronized

synchronized 指一种阻塞式解决方案，也称**【对象锁】**。

它采用互斥的方式，保证了同一时刻只能有一个线程持有**【对象锁】**，其他线程要是想获取这个对象锁就会被阻塞，保证了线程执行**临界区的代码**的时候不会有其他线程也来执行。

## 1。加在代码块上

语法：

```java
synchronized (对象) { //线程一（获取锁），线程二（阻塞）
	临界区；
}
```

```java
@Slf4j
public class ThreadTestDemo17 {
    static int count = 0;
    static final Object lock = new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                synchronized (lock) {
                    count++;
                }
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                synchronized (lock) {
                    count--;
                }
            }
        });

        t1.start();
        t2.start();

        t1.join();
        t2.join();

        log.debug("count = {}", count);
    }
}
```

```cmd
2026-03-21 17:41:49.741 [main] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo17 - count = 0
```



### 注意

- 如果多个线程的临界区内访问的共享资源是相同的，就要获取同一个对象的对象锁

  例如 t1 synchronized(obj1) ，t2 synchronized(obj1)；而不是t1 synchronized(obj1)，t2 synchronized(obj2)

  第二种：获取的都不是同一个对象的锁，就压根锁不住，两个线程就压根不是互斥进行的。

可以改造为面向对象的方式

```java
@Slf4j
public class ThreadTestDemo17 {
    public static void main(String[] args) throws InterruptedException {
        Room room = new Room();
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                room.increment();
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                room.decrement();
            }
        });

        t1.start();
        t2.start();

        t1.join();
        t2.join();

        log.debug("count = {}", room.getCount());
    }
}

class Room {
    private int count = 0;

    public void increment() {
        synchronized (this) {
            count++;
        }
    }

    public void decrement() {
        synchronized (this) {
            count--;
        }
    }

    public int getCount() {
        return count;
    }
}
```



## 2。加在普通方法上

```java
Class test{
    public synchronized void add() {
    	count++;
	}
}

# 等价于
Class test{
    public void add() {
    	synchronized (this) {   // 真实等价写法
        	count++;
         }
	}
}
```

这种写法看上去好像是把方所锁住了一样，但其实并不是，synchronized锁的永远是对象。

加在方法上锁住的是this，也就是本身这个对象。

通过哪个对象调用这个方法，锁住的就是这个对象。

如果线程想要执行 add 这个方法，就必须得先拿到 this 这个对象的锁。

这里有个坑，

这种操作，锁住的是this，是调用个方法的实例的对象，

那假如现在，有两个实例对象

A B 两个线程各自执行各自对象的add操作，那么还会锁住吗，还会进行互斥操作吗，

```java
Counter c1 = new Counter();
Counter c2 = new Counter();

Thread A -> c1.add();
Thread B -> c2.add();
```

一定会锁住，但是两个线程不会互斥执行，

因为两个线程拿到的是两个不同对象实列的锁，所以根本不会互斥执行，

synchronized修饰普通方法的锁，是对象级别的，只要锁对象相同，所有使用这把锁的代码都会互斥



## 3。加在静态方法上

synchronized  修饰静态方法

锁的就不是实例对象this了，而是整个类的Class对象

```java
Class test{
    public synchronized static void add() {
    	count++;
	}
}

# 等价于
Class test{
    public static void add() {
    	synchronized (Test.class) {   // 真实等价写法
        	count++;
         }
	}
}
```

就是说，即使创建了同一个类的两个不同的对象，并且通过这两个不同的对象调用这个方法，依然会被锁住。

```java
class Counter {
    private static int count = 0;

    public static synchronized void add() {
        count++;
    }

    public static int getCount() {
        return count;
    }
}

Counter c1 = new Counter();
Counter c2 = new Counter();

Thread A -> c1.add();
Thread B -> c2.add();
```

虽然c1 c2 是不同的实列对象，

但是锁的都是Counter.Class,

所以线程A 和 线程B会互斥。



### 类对象和对象实例

| 特性         | 类对象                      | 对象实例                   |
| ------------ | --------------------------- | -------------------------- |
| **数量**     | 每个类只有1个               | 可以创建无数个             |
| **创建方式** | JVM自动创建                 | 使用 `new` 关键字          |
| **存储位置** | 方法区                      | 堆内存                     |
| **包含内容** | 类的元数据（方法、字段等）  | 实际的数据值               |
| **生命周期** | 类加载时创建，JVM退出时销毁 | 创建时生成，垃圾回收时销毁 |
| **获取方式** | `Room.class`                | `new Room()`               |

JVM 内存布局：

```
JVM 内存布局：

┌──────────────────────────────────┐
│      方法区（Method Area）       │
│                                │
│  ┌──────────────────────────┐  │
│  │  Room.class（类对象）    │  │
│  │  - 类名：Room          │  │
│  │  - 静态字段：count    │  │
│  │  - 方法：increment()  │  │
│  │         decrement()     │  │
│  │         getCount()     │  │
│  │  - 锁对象            │  │
│  └──────────────────────────┘  │
│           ↑                     │
│           │ 唯一               │
└───────────┼─────────────────────┘
            │
            │ 引用
            ↓
┌──────────────────────────────────┐
│       堆内存（Heap）            │
│                                │
│  ┌─────────────┐  ┌──────────┐│
│  │ instance1   │  │instance2 ││
│  │ (实例1)     │  │(实例2)   ││
│  │ - 实例字段  │  │- 实例字段 ││
│  │ - 锁对象    │  │- 锁对象   ││
│  └─────────────┘  └──────────┘│
│                                │
│  ┌─────────────┐               │
│  │ instance3   │               │
│  │ (实例3)     │               │
│  │ - 实例字段  │               │
│  │ - 锁对象    │               │
│  └─────────────┘               │
└──────────────────────────────────┘
```



### 创建对象时，引用是什么东西？

```
步骤1：new Room()
  ↓
在堆内存中创建 Room 对象
  ↓
分配内存空间，初始化字段
  ↓
返回对象的内存地址（例如：0x12345678）

步骤2：Room room
  ↓
在栈内存中创建引用变量 room
  ↓
步骤3：=
  ↓
将对象的内存地址赋值给引用变量
  ↓
room = 0x12345678（存储对象的地址）

栈内存（Stack）                    堆内存（Heap）
┌─────────────────┐              ┌──────────────────────┐
│                 │              │                      │
│  room           │─────────────→│  Room 对象          │
│  0x12345678     │              │  - count = 0        │
│                 │              │  - 方法列表          │
│  引用变量        │              │  - 其他字段          │
│  (存储地址)      │              │                      │
└─────────────────┘              └──────────────────────┘
      ↑
   遥控器（引用）
   存储电视机的地址
```



# 三。变量的线程安全



## 1。成员变量和静态变量

- 如果未被多个线程共享，则安全。
- 如果被多个线程共享了，分两种情况。
  - 只读，则线程安全。
  - 读写都有，则可能不安全。

## 2。局部变量

局部变量作用范围是方法内部，只能在方法内使用，方法外是不知道该变量的。

- 局部变量是安全的。

- 但是局部变量引用的对象未必安全

  - 如果该对象没有逃脱方法的作用范围，则线程安全。

  - 如果逃脱了，则可能不安全。

    

（1）每个线程一启动，就在**栈**中分配到一块**虚拟机栈**，这是**线程私有**的，每个线程都有自己的栈。

当线程调用方法时，会创建栈帧，多个方法创建多个栈帧，每个方法内的局部变量保存在各自的栈帧中，这是封闭的，线程之间无法共享这些变量，所以不存在并发安全问题。

<img src="juc并发编程.assets/屏幕截图 2026-03-21 220254.png" style="zoom: 67%;" />

（2）局部变量引用的对象**未逃逸**（这个对象只在当前方法内使用，没有被外部访问到），线程安全。

```java
public class ThreadTestDemo18 {
    public static void main(String[] args) {
        ThreadSafe threadSafe = new ThreadSafe();
        for (int i = 0; i < 2; i++) {
            new Thread(() -> {
                threadSafe.method1();
            }, "Thread" + (i + 1)).start();
        }
    }
}

class ThreadSafe {
    public void method1() {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 200; i++) {
            method2(list);
            method3(list);
        }
    }

    private void method2(List<String> list) {
        list.add("1");
    }

    private void method3(List<String> list) {
        list.remove(0);
    }
}
```

- list是局部变量，每个线程调用method1都会创建一个list对象，各自操作各自的list，不存在共享。

- 而method2的参数是从method1中传过来的，与method1中引用的是同一个对象。

- method3的参数与method2也一样。

  <img src="juc并发编程.assets/屏幕截图 2026-03-21 233827.png" style="zoom:50%;" />



（3）引用的对象逃逸了，线程不安全。

逃逸的情况：

- 对象在方法内被返回，因为被返回了，所以可能被多个线程访问。

  ```java
  public StringBuilder method() {
      StringBuilder sb = new StringBuilder();
      return sb;
  }
  ```

  

- 对象被赋值给了成员变量，导致多个线程可以共享访问。

  ```java
  StringBuilder sb;
  
  public void method() {
      sb = new StringBuilder();
  }
  ```

  

- 被传入进了其他线程，导致多个线程共享访问。

  ```java
  public void method() {
      StringBuilder sb = new StringBuilder();
  
      new Thread(() -> {
          sb.append("hello");
      }).start();
  }
  ```

  

例如：

```java
public class ThreadTestDemo18 {
    public static void main(String[] args) {
        ThreadUnSafe threadUnSafe = new ThreadUnSafe();
        for (int i = 0; i < 2; i++) {
            new Thread(() -> {
                threadUnSafe.method1();
            }, "Thread" + (i + 1)).start();
        }
    }
}

class ThreadSafe {
    public void method1() {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 200; i++) {
            method2(list);
            method3(list);
        }
    }

    public void method2(List<String> list) {
        list.add("1");
    }

    public void method3(List<String> list) {
        list.remove(0);
    }
}

class ThreadUnSafe extends ThreadSafe {
    @Override
    public void method3(List<String> list) {
        new Thread(() -> {
            list.remove(0);
        }).start();
    }
}
```

子类重写了父类的method3，并且开启了另一个线程来访问list，就导致多个线程共享访问对象，

可以使用private，final关键字来修饰方法，因为private修饰方法对子类不可见，即使子类写了一个同名的方法，也只是一个新的方法，不是重写了父类中的方法。

而final关键字修饰方法可以直接避免子类重写。

但是，引用对象即使逃逸了，也不一定有线程安全问题：

String对象他具有不可变性，即使多个线程同时读取一个String对象，也不存在线程安全问题。



# 四。常见的线程安全类

Interger等包装类，HashTable，String等都是线程安全类，内部提供的方法是线程安全的，具有原子性。

HashTable：

get和put加了synchronized关键字，都是线程安全的。
<img src="juc并发编程.assets/屏幕截图 2026-03-23 153336.png" style="zoom: 67%;" />

<img src="juc并发编程.assets/屏幕截图 2026-03-23 153402.png" alt="屏幕截图 2026-03-23 153402" style="zoom: 67%;" />



## 1。方法组合使用线程不一定安全

虽然线程安全类中的方法是原子性的，但是多个方法组合使用就不是原子性了。

例如：

```java
HashTable hashTabe = new HashTable();
// 线程1，线程2
if (hashTable.get("key") == null) {
	hashTable.put("value");
}
```

线程1先判断key==null，结果发生了上下文切换，线程2判断key也为null，并且写入了value2；

切换为线程1，写入了value1，结果value1把value2覆盖了，发生了线程不安全问题。

<img src="juc并发编程.assets/屏幕截图 2026-03-23 154906.png" style="zoom:67%;" />

## 2。String

String具有不可变性，创建String的对象就不能在修改。

String类被final修饰，防止了子类继承并重写String类内部的方法。

存储字符内容的字节数组被private，final关键字修饰，也防止了在类外去访问和修改这个字节数组，从而保证了不可变性。

但是String中的 subString，replace都可以改变字符串内容。

实际上，subString并没有修改String对象，而是创建一个新的String，并且把原String对象中的字节数组赋值给这个新的String对象中的字节数组。

<img src="juc并发编程.assets/屏幕截图 2026-03-23 160259.png" style="zoom:67%;" />

<img src="juc并发编程.assets/屏幕截图 2026-03-23 160310.png" alt="屏幕截图 2026-03-23 160310" style="zoom:67%;" />



# 五。Monitor

## 1。Java对象头

对象在内存中由三部分组成：

**对象头** ，实例数据，对其填充字段。

**对象头**包括两部分：

**Klass Word** 类型指针：指向**方法区中的类的元数据**的指针，通过它能找对象从属于哪个**Class类**。

**Mark Word** 标记字段：记录哈希、GC 信息、锁状态等。

32位虚拟机下的结构

```ruby
|--------------------------------------------------------|--------------------|
|                  Mark Word (32 bits)                   |       State        |
|--------------------------------------------------------|--------------------|
| identity_hashcode:25 | age:4 | biased_lock:0 | lock:01 |       Normal       | 无锁
|--------------------------------------------------------|--------------------|
|  thread:23 | epoch:2 | age:4 | biased_lock:1 | lock:01 |       Biased       | 偏向锁
|--------------------------------------------------------|--------------------|
|               ptr_to_lock_record:30          | lock:00 | Lightweight Locked | 轻量级锁
|--------------------------------------------------------|--------------------|
|               ptr_to_heavyweight_monitor:30  | lock:10 | Heavyweight Locked | 重量级锁
|--------------------------------------------------------|--------------------|
|                                              | lock:11 |    Marked for GC   |
|--------------------------------------------------------|--------------------|
```

![](juc并发编程.assets/20200606113736103.png)

## 2。Monitor（监视器锁）

### synchronized底层实现

**synchronized** 底层实现依靠 **Monitor**（监视器锁）重量级锁实现的，靠**对象头+Monitor**实现加锁和解锁。

每个Java对象都可以关联一个**Monitor**（由JVM提供），由JVM层面实现的。

**Monitor**包含三部分：

- **Owner**：持有锁的线程
- **EntryList**：阻塞队列 / 入口列表：存放试图获取锁，但是获取失败的线程集合。
- **WaitSet**：等待队列：包含所有调用 `wait()` 方法后被挂起的线程的集合，线程调用wait（）时，会释放掉锁，进入wait set，暂时挂起自己。

<img src="juc并发编程.assets/屏幕截图 2026-03-23 221117.png" style="zoom:67%;" />

比如：

线程t2想要执行临界区内的代码，会尝试让**Obj对象和一个Monitor相关联**，会在Obj对象头中的**MarkWord**中记录**Monitor**的**指针地址**。

此时Monitor中的**Owner**设置为t2，代表t2拥有了这把锁。

<img src="juc并发编程.assets/屏幕截图 2026-03-23 215000.png" style="zoom: 50%;" />

T2上锁的过程中，其他线程也来执行临界区内的代码，就会进入EntryList Blocked。

<img src="juc并发编程.assets/屏幕截图 2026-03-23 215235.png" style="zoom:50%;" />

T2执行完临界区内的代码，然后唤醒EnrtryList中等待的线程竞争锁，竞争时是非公平的，不一定先进来的先获得锁。

<img src="juc并发编程.assets/屏幕截图 2026-03-23 215358.png" style="zoom:50%;" />





## 故事

- 老王 - 操作系统
- 小南 - 线程
- 小女 - 线程
- 房间 - 对象
- 房间门上 - 防盗锁 - Monitor
- 房间门上 - 小南书包 - 轻量级锁
- 房间门上 - 刻上小南大名 - 偏向锁
- 批量重刻名 - 一个类的偏向锁撤销到达 20 阈值
- 不能刻名字 - 批量撤销该类对象的偏向锁，设置该类不可偏向

小南要使用房间保证计算不被其它人干扰（原子性），最初，他用的是防盗锁，当上下文切换时，锁住门。这样，即使他离开了，别人也进不了门，他的工作就是安全的。

但是，很多情况下没人跟他来竞争房间的使用权。小女是要用房间，但使用的时间上是错开的，小南白天用，小女晚上用。每次上锁太麻烦了，有没有更简单的办法呢？

小南和小女商量了一下，约定不锁门了，而是谁用房间，谁把自己的书包挂在门口，但他们的书包样式都一样，因此每次进门前得翻翻书包，看课本是谁的，如果是自己的，那么就可以进门，这样省的上锁解锁了。万一书包不是自己的，那么就在门外等，并通知对方下次用锁门的方式。

后来，小女回老家了，很长一段时间都不会用这个房间。小南每次还是挂书包，翻书包，虽然比锁门省事

于是，小南干脆在门上刻上了自己的名字：【小南专属房间，其它人勿用】，下次来用房间时，只要名字还在，那么说明没人打扰，还是可以安全地使用房间。如果这期间有其它人要用这个房间，那么由使用者将小南刻的名字擦掉，升级为挂书包的方式。

同学们都放假回老家了，小南就膨胀了，在 20 个房间刻上了自己的名字，想进哪个进哪个。后来他自己放假回老家了，这时小女回来了（她也要用这些房间），结果就是得一个个地擦掉小南刻的名字，升级为挂书包的方式。老王觉得这成本有点高，提出了一种批量重刻名的方法，他让小女不用挂书包了，可以直接在门上刻上自己的名字

后来，刻名的现象越来越频繁，老王受不了了：算了，这些房间都不能刻名了，只能挂书包



# 六。synchronized优化原理

## 1。轻量级锁

使用**Lock Record**锁记录充当锁。

- 使用场景：虽然某个**对象被多个线程**访问上锁，但多个线程的**访问时间是错开**的（同一时刻不存在竞争关系），可使用**轻量级锁优化**。

```java
    static final Object obj = new Object();
    public static void method1() {
        synchronized (obj) {
            // 同步块1
            method2();
        }
    }
    
    public static void method2() {
        synchronized (obj) {
            // 同步块2
        }
    }
```

### 加锁

- 线程T1执行method1，在栈里创建一个栈帧，栈帧里创建一个**锁记录（Lock Record）对象**，Lock Record包含两部分：

  **Lock Record**

  - **Object reference** **对象指针**：记录锁住的对象的地址。
  - **Displaced Mark Word**：记录对象的 **Mark Word**

![](juc并发编程.assets/屏幕截图 2026-03-24 161028.png)

- 尝试获取锁，让**锁记录中的Object reference**指向锁的对象，并尝试cas替换Object的 Mark Word，将Marl Word的值存入到锁记录中。

![](juc并发编程.assets/屏幕截图 2026-03-24 163215.png)

- 如果**CAS成功**，对象头中存储**锁记录的地址**和状态00，代表获得了轻量级锁，该线程给对象加了锁。

![](juc并发编程.assets/屏幕截图 2026-03-24 163514.png)

- 如果**CAS失败**，分两种情况：

  - **其他线程争抢对象锁**，发现对象头中的状态为**01**，代表已经上锁了，进入**锁膨胀**。

  - 线程自己又执行了synchronized（**锁重入**），比如：method1调用method2，会在栈帧中**再创建一个锁记录**。Object reference还会指向锁对象，但是cas交换会失败，因为线程自己已经上过锁了。此时锁记录不会存储对象的Mark Word，而是空null。

    这条锁记录可作为**重入的计数**。

![](juc并发编程.assets/屏幕截图 2026-03-24 164944.png)

### 解锁

- 退出synchronized代码块（解锁），如果锁记录取值为null，代表有重入，重置锁记录，重入数-1。

- 不为null，会CAS将 Mark Word的值回复给对象头
  - 成功，解锁成功
  - 失败，说明轻量级锁进入了膨胀或已经升级为重量级锁，进入重量级锁的解锁流程。

![](juc并发编程.assets/屏幕截图 2026-03-24 165854.png)



## 2。锁膨胀

尝试加**轻量级锁**，但是**CAS**操作失败，一种情况代是表有线程已经为对象加上了轻量级锁，此时产生竞争，发生**锁膨胀**，**轻量级锁**变为**重量级锁**。

- T1想给对象加轻量级锁，使用CAS操作交换对象的Mark Word，但是T0已经给对象加了轻量级锁。

![](juc并发编程.assets/屏幕截图 2026-03-24 181541.png)

- T1加轻量级锁肯定失败，进入了锁膨胀。
  - 因为T1拿不到锁，不能让他干耗着，而轻量级没有阻塞队列的概念，此时会为Object对象申请一个**Monitor重量级锁**，让**Object 的Mark Word 指向重量级锁的地址，状态变为10**，Monitor的Owner设置为T0.
  - 然后T1会进入 **Monitor的 EntryList BLOCKED**。

![](juc并发编程.assets/屏幕截图 2026-03-24 182307.png)

- T0想要解锁，使用CAS将Mark Word的值恢复给对象头，但是失败，因为Mark Word此时已经指向了Monitor。解锁流程就进入了**重量级锁的解锁流程**，通过Monitor 地址找到Monitor，将Owner设置为空，唤醒EntryList中BLOCKED的线程。



## 3。自旋优化

重量级锁竞争时，可以使用自旋进行优化。

- **自旋**就是：竞争的时候，发现已经线程抢到对象锁了，这个线程暂时先不进入Monitor的EntryList受到Blocked，而是进入几次循环，如果循环的过程中发现持锁线程已经释放锁了，它就可以成为新的Owner了，避免了阻塞，这是**自旋成功**。

- **自旋失败**：循环过程中，其他线程没释放锁，导致线程还是抢不到锁，最后还是会进入EntrtLlst中。

单核CPU自旋没有用，因为自旋会使用到CPU，单核情况下，一个线程如果已经持有锁，根本没有CPU分给其他线程去自旋。



## 4。偏向锁

**偏向锁**可以优化**轻量级锁**每次重入造成多次CAS操作的问题。

- 轻量级锁在没有竞争的时候（自己一个线程），多次执行synchronized（重入），创建多次锁记录，仍然需要执行CAS操作，尝试对象的Mark Word存入锁记录中，但是都是null。

<img src="juc并发编程.assets/屏幕截图 2026-03-24 210135.png" style="zoom:67%;" />

第一次使用CAS的时候，将**线程ID**设置到对象的对象头中的**Mark Word**中，也就是Mark Word不在存储轻量级锁的锁记录**Lock Record的地址**，也不存储重量级锁**Monitor的地址**，只存储**持锁线程ID**，将对象锁状态设为 biased_lock:1 ， lock:01

之后要是再抢锁，发现线程ID是自己，就不用重新CAS。只要不发生竞争，这个对象就归这个线程所有。

<img src="juc并发编程.assets/屏幕截图 2026-03-24 210938.png" style="zoom:67%;" />

一个对象创建：

处于无锁但“可偏向”的状态。只有当第一次有线程获取锁时，JVM 才会将线程 ID 记录到对象头中，使其进入偏向锁状态。

 在 JDK 8 中：

- 偏向锁默认开启
- 对象头初始就标记为“可偏向”
- JVM 默认有一个**延迟（约 4 秒）**
- 之后才开启偏向锁



### 偏向锁的撤销

- 是指偏向锁退回到不可偏向的未锁定状态或者升级为轻量级锁。

### 撤销-调用对象的hashCode（）

如果对象此时处于偏向锁状态，调用对象的hashCode（）方法会出现很诡异的一件事：

偏向锁会被撤销，对象的锁状态从101 变为 001。

原因：

- 偏向锁会在对象头中存储线程 ID，而调用 hashCode() 后需要在对象头中存储 hashCode，由于 Mark Word 空间有限，无法同时存储两者，因此 JVM 会撤销偏向锁，并将锁升级为轻量级锁或其他状态。



### 撤销-其他线程使用对象

- 当有其他线程使用偏向锁对象时候，偏向锁会升级为轻量级锁。



### 撤销-调用wait/notify

- 偏向锁升级为重量级锁。



### 批量重偏向

重偏向：

某个对象虽然被多个线程访问，但是是**交替访问**的，**某个时间内没有竞争**，这时偏向线程T1的对象还有机会重新偏向于T2，就是说**锁的偏向权可以转让给另一个线程**，重偏向可以重置对象的Thread ID。

T2访问对象，由于此时对对象偏向于T1，会把偏向锁撤销，升级为轻量级锁，但是撤销比较耗费性能。当对一个特定类的对象的**偏向锁撤销次数达到一个阈值时，默认为20次**，基于**类级别**统计撤销次数，**不是单个对象**，JVM 会触发**批量重偏向**，让该类对象可以偏向新线程



### 批量撤销

当JVM发现某个类的锁不仅存在竞争，而且竞争非常频繁，偏向锁反而成了累赘（因为每次都要做撤销操作，得不偿失）时，JVM会决定**彻底放弃对这个类的对象使用偏向锁**。

**什么时候触发批量撤销？**
 当对一个特定类的对象的偏向锁撤销次数达到一个**阈值**时（例如，默认40次），JVM就会认为这个类的锁是“高度竞争的”，不适合再用偏向锁。

**发生了什么？**
 JVM会将该类的标志置为**不可偏向**。之后，所有为该类**新创建**的对象，其对象头都会直接标记为不可偏向的状态（01）。从此，这些新对象再也不会进入偏向锁模式，任何一个线程来加锁，都会直接走轻量级锁的流程。

**注意：**  批量撤销只影响**之后新创建**的对象。之前已经创建出来的、正处于偏向状态的对象，仍然会在下次发生竞争时进行单独的撤销。



## 

# 七。wait / notify

## 1。原理

![](juc并发编程.assets/屏幕截图 2026-03-25 171319.png)

- 持有锁的线程发现条件不满足，调用wait方法，会退出持有锁，进入Monitor的WaitSet中，变为Waitting状态。
- EntryList 和 WaitSet都是用来存放处于阻塞状态的线程，但是EntryList中的线程还未获取到锁，WaitSet中的线程是已经获取到了锁但是放弃了。
- 处于BlOCKED的线程会在Owner线程释放锁时被唤醒，去争抢锁。
- WAITTING线程会在Owner线程调用notify 或notifyALL时被唤醒，但是唤醒之后不会累积获得锁，而是重新进入EntryList中争抢锁。



## 2。API

- obj.wait（）让线程进入WaitSet
- obj.wait（long timeout）带时效的wait，等待时间内如果没有被其他线程唤醒，自己自动唤醒，进入EntryList去获取锁
- obj.notify（） 在WaitSet中挑一个线程唤醒
- obj.notifyAll（）唤醒WaitSet中的全部线程

这些方法都属于Object对象的方法，使用前必须得先获取到对象的锁。

```java
// 未获得对象锁的情况下使用，运行会报错
public class ThreadTestDemo19 {
    static final Object lock = new Object();
    public static void main(String[] args) throws InterruptedException {
        lock.wait();
    }
}

# 报错
Exception in thread "main" java.lang.IllegalMonitorStateException: current thread is not owner
	at java.base/java.lang.Object.wait0(Native Method)
	at java.base/java.lang.Object.wait(Object.java:366)
	at java.base/java.lang.Object.wait(Object.java:339)
	at JUC.JUCTest.Demo.D1.ThreadTestDemo19.main(ThreadTestDemo19.java:6)
```

```java
public class ThreadTestDemo19 {
    static final Object lock = new Object();
    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            synchronized (lock) {
                log.debug("t1 获取到了锁");
                try {
                    log.debug("t1 进入 WaitSet");
                    lock.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                log.debug("t1 从 WaitSet 中唤醒");
            }
        },"t1").start();

        new Thread(() -> {
            synchronized (lock) {
                log.debug("t2 获取到了锁");
                try {
                    log.debug("t2 进入 WaitSet");
                    lock.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                log.debug("t2 从 WaitSet 中唤醒");
            }
        },"t2").start();

        Thread.sleep(2000);
        log.debug("主线程唤醒一个线程");
        synchronized (lock) {
            // 唤醒一个线程
            lock.notify();
            // 唤醒所有线程
            lock.notifyAll();
        }
    }
}
```

![](juc并发编程.assets/屏幕截图 2026-03-25 173956.png)

![](juc并发编程.assets/屏幕截图 2026-03-25 174101.png)



## 3。使用

sleep（int n）和 wait（int n）的区别：

- sleep是Thread的静态方法，wait是Object的方法
- sleep不用强制配合synchronized使用，wait需要和synchronized一起使用，因为需要获得锁才能用
- sleep期间，线程不会释放掉对象锁，wait期间，线程会释放掉对象锁
- 状态都是TIMED_WAITTING

```java
@Slf4j
public class ThreadTestDemo20 {
    static final Object lock = new Object();
    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            synchronized (lock) {
                try {
                    log.debug("线程T1获取到锁");
                    Thread.sleep(20000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }).start();
        Thread.sleep(1000);
        synchronized (lock) {
            log.debug("主线程尝试获取到锁");
        }
    }
}
```

![](juc并发编程.assets/屏幕截图 2026-03-25 181748.png)

可以看到，sleep期间线程并没有释放掉锁。

```java
@Slf4j
public class ThreadTestDemo20 {
    static final Object lock = new Object();
    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            synchronized (lock) {
                try {
                    log.debug("线程T1获取到锁");
                    lock.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }).start();
        Thread.sleep(1000);
        synchronized (lock) {
            log.debug("主线程尝试获取到锁");
        }
    }
}
```

调用wait方法，线程会释放掉锁。

![](juc并发编程.assets/屏幕截图 2026-03-25 182107.png)



### 使用方式1

使用sleep进行阻塞

```java
@Slf4j
public class ThreadTestDemo21 {
    static final Object lock = new Object();
    static boolean hasCigarette = false;
    static boolean hasTakeOut = false;

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            synchronized (lock) {
                log.debug("有没有烟：{}", hasCigarette);
                if (!hasCigarette) {
                    log.debug("没有烟，歇一会");
                    try {
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("有没有烟：{}", hasCigarette);
                if (hasCigarette) {
                    log.debug("有了，烟民开始干活");
                }
            }
        }, "烟民").start();

        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                synchronized (lock) {
                    log.debug("其他人开始干活");
                }
            }, "其他人").start();
        }

        Thread.sleep(1000);
        new Thread(() -> {
            synchronized (lock) {
                hasCigarette = true;
                log.debug("烟到了");
            }
        }, "送烟的").start();
    }
}

```

- 烟民必须得等到2秒之后才醒过来，就算是烟提前送到了，也无法立刻醒来。
- 其他线程一直被阻塞，得等烟民醒来之后，拿到了烟，释放了锁之后，才干活，效率低。
- 送烟人想要获得锁送烟，但是烟民sleep期间不释放锁，导致烟送不进来



### 使用方式2

可以使用**wait和notify**，让阻塞期间其他线程拿到锁去执行。

烟民线程调用wait，释放锁，进入WaitSet中。

其他人线程拿到锁，去执行代码。

送烟线程送完烟之后唤醒烟民线程。

```java
@Slf4j
public class ThreadTestDemo21 {
    static final Object lock = new Object();
    static boolean hasCigarette = false;
    static boolean hasTakeOut = false;

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            synchronized (lock) {
                log.debug("有没有烟：{}", hasCigarette);
                if (!hasCigarette) {
                    log.debug("没有烟，歇一会");
                    try {
                        // 释放锁，进入WaitSet
                        lock.wait();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("有没有烟：{}", hasCigarette);
                if (hasCigarette) {
                    log.debug("有了，烟民开始干活");
                }
            }
        }, "烟民").start();

        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                synchronized (lock) {
                    log.debug("其他人开始干活");
                }
            }, "其他人").start();
        }

        Thread.sleep(1000);
        new Thread(() -> {
            synchronized (lock) {
                hasCigarette = true;
                log.debug("烟到了");
                // 唤醒烟民线程
                lock.notify();
            }
        }, "送烟的").start();
    }
}
```

```cmd
执行结果
2026-03-25 21:32:19.838 [烟民] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有没有烟：false
2026-03-25 21:32:19.840 [烟民] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 没有烟，歇一会
2026-03-25 21:32:19.840 [其他人] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 其他人开始干活
2026-03-25 21:32:19.840 [其他人] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 其他人开始干活
2026-03-25 21:32:19.840 [其他人] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 其他人开始干活
2026-03-25 21:32:19.841 [其他人] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 其他人开始干活
2026-03-25 21:32:19.841 [其他人] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 其他人开始干活
2026-03-25 21:32:20.840 [送烟的] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 烟到了
2026-03-25 21:32:20.840 [烟民] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有没有烟：true
2026-03-25 21:32:20.840 [烟民] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有了，烟民开始干活
```

但是这也有问题：

假设有两个线程都进入了WaitSet，烟民等烟，饥饿者等饭。

现在有个外卖员，送饭的，调用了notify，他肯定想唤醒饥饿者线程，但是notify只能唤醒一个线程，并且是随机的，它不能确保一定会唤醒饥饿者，也有可能会唤醒烟民线程，但人家烟民要的是烟，不是饭，结果就是你送错了。

```java
@Slf4j
public class ThreadTestDemo21 {
    static final Object lock = new Object();
    static boolean hasCigarette = false;
    static boolean hasTakeOut = false;

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            synchronized (lock) {
                log.debug("有没有烟：{}", hasCigarette);
                if (!hasCigarette) {
                    log.debug("没有烟，歇一会");
                    try {
                        // 释放锁，进入WaitSet
                        lock.wait();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("有没有烟：{}", hasCigarette);
                if (hasCigarette) {
                    log.debug("有了，烟民开始干活");
                }
            }
        }, "烟民").start();

        new Thread(() -> {
            synchronized (lock) {
                log.debug("有没有饭：{}", hasTakeOut);
                if (!hasTakeOut) {
                    log.debug("没有饭，等一会");
                    try {
                        // 释放锁，进入WaitSet
                        lock.wait();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("有没有饭：{}", hasTakeOut);
                if (hasTakeOut) {
                    log.debug("有了，饥饿者开始吃饭");
                }
            }
        }, "饥饿者").start();

        Thread.sleep(1000);
        new Thread(() -> {
            synchronized (lock) {
                hasTakeOut = true;
                log.debug("饭到了");
                // 唤醒烟民线程
                lock.notify();
            }
        }, "外卖员").start();
    }
}
```

执行结果：

```cmd
2026-03-25 21:44:13.917 [烟民] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有没有烟：false
2026-03-25 21:44:13.917 [烟民] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 没有烟，歇一会
2026-03-25 21:44:13.919 [饥饿者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有没有饭：false
2026-03-25 21:44:13.919 [饥饿者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 没有饭，等一会
2026-03-25 21:44:14.932 [外卖员] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 饭到了
2026-03-25 21:44:14.932 [烟民] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有没有烟：false
```



### 使用方式3

可以使用**notifyAll**，把全部线程都唤醒，解决上面的问题。

```java
@Slf4j
public class ThreadTestDemo21 {
    static final Object lock = new Object();
    static boolean hasCigarette = false;
    static boolean hasTakeOut = false;

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            synchronized (lock) {
                log.debug("有没有烟：{}", hasCigarette);
                if (!hasCigarette) {
                    log.debug("没有烟，歇一会");
                    try {
                        // 释放锁，进入WaitSet
                        lock.wait();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("有没有烟：{}", hasCigarette);
                if (hasCigarette) {
                    log.debug("有了，烟民开始干活");
                }
            }
        }, "烟民").start();

        new Thread(() -> {
            synchronized (lock) {
                log.debug("有没有饭：{}", hasTakeOut);
                if (!hasTakeOut) {
                    log.debug("没有饭，等一会");
                    try {
                        // 释放锁，进入WaitSet
                        lock.wait();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("有没有饭：{}", hasTakeOut);
                if (hasTakeOut) {
                    log.debug("有了，饥饿者开始吃饭");
                }
            }
        }, "饥饿者").start();

        Thread.sleep(1000);
        new Thread(() -> {
            synchronized (lock) {
                hasTakeOut = true;
                log.debug("饭到了");
                // 唤醒烟民线程
                lock.notifyAll();
            }
        }, "外卖员").start();
    }
}
```

```cmd
2026-03-25 21:54:54.798 [烟民] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有没有烟：false
2026-03-25 21:54:54.799 [烟民] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 没有烟，歇一会
2026-03-25 21:54:54.800 [饥饿者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有没有饭：false
2026-03-25 21:54:54.800 [饥饿者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 没有饭，等一会
2026-03-25 21:54:55.810 [外卖员] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 饭到了
2026-03-25 21:54:55.810 [烟民] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有没有烟：false
2026-03-25 21:54:55.810 [饥饿者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有没有饭：true
2026-03-25 21:54:55.810 [饥饿者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有了，饥饿者开始吃饭
```

但是问题是，烟民还是没有得到烟，但是还是被唤醒了，没干活，并且之后烟民线程就不在WaitSet中了，而是进入了Entry List中，即使后续送烟者送到了烟尝试唤醒烟民也没用了，因为烟民压根就不在WaitSet中。

```java
@Slf4j
public class ThreadTestDemo21 {
    static final Object lock = new Object();
    static boolean hasCigarette = false;
    static boolean hasTakeOut = false;

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            synchronized (lock) {
                log.debug("有没有烟：{}", hasCigarette);
                if (!hasCigarette) {
                    log.debug("没有烟，歇一会");
                    try {
                        // 释放锁，进入WaitSet
                        lock.wait();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("有没有烟：{}", hasCigarette);
                if (hasCigarette) {
                    log.debug("有了，烟民开始干活");
                }
            }
        }, "烟民").start();

        new Thread(() -> {
            synchronized (lock) {
                log.debug("有没有饭：{}", hasTakeOut);
                if (!hasTakeOut) {
                    log.debug("没有饭，等一会");
                    try {
                        // 释放锁，进入WaitSet
                        lock.wait();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("有没有饭：{}", hasTakeOut);
                if (hasTakeOut) {
                    log.debug("有了，饥饿者开始吃饭");
                }
            }
        }, "饥饿者").start();

        Thread.sleep(1000);
        new Thread(() -> {
            synchronized (lock) {
                hasTakeOut = true;
                log.debug("饭到了");
                // 唤醒烟民线程
                lock.notifyAll();
            }
        }, "外卖员").start();

        Thread.sleep(1000);
        new Thread(() -> {
            synchronized (lock) {
                hasCigarette = true;
                log.debug("烟到了");
                // 唤醒饥饿者线程
                lock.notifyAll();
            }
        },"送烟者").start();
    }
}
```

```cmd
2026-03-25 22:12:36.584 [烟民] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有没有烟：false
2026-03-25 22:12:36.585 [烟民] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 没有烟，歇一会
2026-03-25 22:12:36.586 [饥饿者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有没有饭：false
2026-03-25 22:12:36.586 [饥饿者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 没有饭，等一会
2026-03-25 22:12:37.591 [外卖员] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 饭到了
2026-03-25 22:12:37.591 [烟民] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有没有烟：false
2026-03-25 22:12:37.592 [饥饿者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有没有饭：true
2026-03-25 22:12:37.592 [饥饿者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有了，饥饿者开始吃饭
2026-03-25 22:12:38.590 [送烟者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 烟到了
```

烟民压根就拿不到烟。

这也叫：虚假唤醒



### 受用方式4

**把if替换为while**，这样即使唤醒了，进入while，再次调用wait，进入WaitSet

```java
@Slf4j
public class ThreadTestDemo21 {
    static final Object lock = new Object();
    static boolean hasCigarette = false;
    static boolean hasTakeOut = false;

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            synchronized (lock) {
                log.debug("有没有烟：{}", hasCigarette);
                while (!hasCigarette) {
                    log.debug("没有烟，歇一会");
                    try {
                        // 释放锁，进入WaitSet
                        lock.wait();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("有没有烟：{}", hasCigarette);
                if (hasCigarette) {
                    log.debug("有了，烟民开始干活");
                }
            }
        }, "烟民").start();

        new Thread(() -> {
            synchronized (lock) {
                log.debug("有没有饭：{}", hasTakeOut);
                while (!hasTakeOut) {
                    log.debug("没有饭，等一会");
                    try {
                        // 释放锁，进入WaitSet
                        lock.wait();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("有没有饭：{}", hasTakeOut);
                if (hasTakeOut) {
                    log.debug("有了，饥饿者开始吃饭");
                }
            }
        }, "饥饿者").start();

        Thread.sleep(1000);
        new Thread(() -> {
            synchronized (lock) {
                hasTakeOut = true;
                log.debug("饭到了");
                // 唤醒烟民线程
                lock.notifyAll();
            }
        }, "外卖员").start();
    }
}

```

```cmd
![屏幕截图 2026-03-25 225241](../../Pictures/Screenshots/屏幕截图 2026-03-25 225241.png)![屏幕截图 2026-03-25 225241](../../Pictures/Screenshots/屏幕截图 2026-03-25 225241.png)2026-03-25 22:27:40.051 [烟民] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有没有烟：false
2026-03-25 22:27:40.055 [烟民] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 没有烟，歇一会
2026-03-25 22:27:40.055 [饥饿者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有没有饭：false
2026-03-25 22:27:40.055 [饥饿者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 没有饭，等一会
2026-03-25 22:27:41.061 [外卖员] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 饭到了
2026-03-25 22:27:41.061 [烟民] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 没有烟，歇一会
2026-03-25 22:27:41.062 [饥饿者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有没有饭：true
2026-03-25 22:27:41.062 [饥饿者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有了，饥饿者开始吃饭
2026-03-25 22:27:42.074 [送烟者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 烟到了
2026-03-25 22:27:42.074 [烟民] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有没有烟：true
2026-03-25 22:27:42.074 [烟民] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo21 - 有了，烟民开始干活
```



# 八。同步模式-保护性暂停

保护性暂停（Guarded Suspension）用于**一个线程等待另一个线程的执行结果**。

- 有一个结果需要从一个线程传递到另一个线程，让它们关联同一个GuardedObject。
- 两个线程共同关联一个保护对象
- join，Future的实现采用了保护暂停模式。

![](juc并发编程.assets/屏幕截图 2026-03-25 225241.png)

## 设计思路

获取数据：线程抢到对象锁，判断数据是否为空，如果为空代表发送线程还未发送或者未发送完数据，此时调用waiti让线程进入WaitSet中等会，等下次被唤醒，判断数据是否为空，不为空，就拿走数据。

发送数据：线程抢到对象锁，为数据赋值，并调用notifyAll唤醒获取数据的线程。

```java
package JUC.JUCTest.Demo.D1;

import lombok.extern.slf4j.Slf4j;

import java.util.ArrayList;
import java.util.List;

@Slf4j
public class GuardedTest {
    public static void main(String[] args) {
        GuardedLoader guardedLoader = new GuardedLoader();
        // 传输数据线程
        new Thread(() -> {
            // 构造数据集合
            List<Integer> numbersList = new ArrayList<>();
            for (int i = 0; i < 10; i++) {
                numbersList.add(i);
            }
            // 向 guardedLoader 发送数据
            log.debug("开始发送数据，发送数据为：{}",numbersList);
            // 先睡会再发数据，让接收者等会数据
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            guardedLoader.setGuarded(numbersList);
        },"发送者").start();

        // 接收数据线程
        new Thread(() -> {
            // 开始接受数据
            log.debug("开始接收数据");
            List<Integer> guarded = (List<Integer>) guardedLoader.getGuarded();
            log.debug("接收数据：{}",guarded);
        },"接收者").start();
    }

}

@Slf4j
class GuardedLoader {
    private Object guarded;

    // 获取数据
    public Object getGuarded() {
        synchronized (this) {
            // 校验数据是否为空
            while (guarded == null) {
                try {
                    // 数据为空，进入循环等待
                    log.debug("数据为空，进入循环等待");
                    this.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }
        return guarded;
    }

    // 发送数据
    public void setGuarded(Object object) {
        synchronized (this) {
            this.guarded = object;
            this.notifyAll();
        }
    }
}
```

```
2026-03-28 16:27:38.289 [接收者]  - 开始接收数据
2026-03-28 16:27:38.289 [发送者]  - 开始发送数据，发送数据为：[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
2026-03-28 16:27:38.292 [接收者]  - 数据为空，进入循环等待
2026-03-28 16:27:41.293 [接收者]  - 接收数据：[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```



## 增加超时

上面的wait是一直等待，如果其他线程忘记唤醒它，他就会一直等。

可以给他设置超时时间。

```java
    // 获取数据
    public Object getGuarded(long timeout) {
        synchronized (this) {
            long begin = System.currentTimeMillis();
            long passedTime = 0;
            // 校验数据是否为空
            while (guarded == null) {
                long waitTime = timeout - passedTime;
                if (waitTime <= 0) {
                    break;
                }
                try {
                    // 数据为空，进入循环等待
                    log.debug("数据为空，进入循环等待");
                    this.wait(waitTime);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                passedTime = System.currentTimeMillis() - begin;
            }
        }
        return guarded;
    }
```



## join原理

```java
public final synchronized void join(long millis)
throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (millis == 0) {
        while (isAlive()) {//只要线程还没结束，主线程就会一直阻塞
            wait(0);//这里的wait调用的本地方法。
        }
    } else {//等待一段指定的时间
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
```

join底层通过调用wait方法实现线程阻塞，并通过保护性暂停实现超时暂停。

如果超时时间为负数，会抛出异常。

为0，判断线程是否存活，调用wait（0），表示一直阻塞。

否则代表设置了超时时间，采用保护性暂停模式。



## 解耦

如果多个类之间都要使用ObjectGuarded对象，作为参数来传递非常不方便，需要一个中间类来接偶。

Futures就好比一个信箱，一个信箱对应一个房间号，左边线程就相当于居民，右边线程就相当于邮递员。

Futures可以维护一个集合用来存放ObjectGuarded对象，并给每一个对象设置一个id作为唯一标识。

![](juc并发编程.assets/屏幕截图 2026-03-28 173141.png)

```java
@Slf4j
public class GuardedTest {
    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 3; i++) {
            new Person().start();
        }
        Thread.sleep(1000);
        for (Integer id : MailBox.getGuardedObjectsIds()) {
            new PostMan(id,"内容" + id).start();
        }
    }
}

/**
 * 居民
 */
@Slf4j
class Person extends Thread {
    @Override
    public void run() {
        GuardedObject guardedObject = MailBox.createGuardedOBject();
        log.debug("开始收信，信的编号：{}", guardedObject.getId());
        Object mail = guardedObject.getData(5000);
        log.debug("收到信 id：{}，信的内容为：{}", guardedObject.getId(), mail);
    }
}

/**
 * 邮递员
 */
@Slf4j
class PostMan extends Thread {
    // GuarderObject 对象标识
    private int id;
    // 发送的数据
    private String mail;

    public PostMan(int id, String mail) {
        this.id = id;
        this.mail = mail;
    }

    @Override
    public void run() {
        // 获取 GuardedObject 对象
        GuardedObject guardedObject = MailBox.getGuardedObject(id);
        log.debug("发送数据，数据编号为：{}，数据内容为：{}", id, mail);
        // 发送数据
        guardedObject.setData(mail);
    }
}

/**
 * 邮箱类
 *
 */
class MailBox {
    // 用于存放 GuardedObject 的集合
    private static Map<Integer, GuardedObject> mailBox = new Hashtable<>();
    // 用于表示 GuardedObject 的唯一标识
    private static int id = 1;

    // 产生 GuardedObject 的唯一标识
    private synchronized static int generateId() {
        return id++;
    }

    // 创建 GuardedObject 的对象，并给每个对象分配 GuardedObject 的唯一表示
    public static GuardedObject createGuardedOBject() {
        GuardedObject guardedObject = new GuardedObject(generateId());
        mailBox.put(guardedObject.getId(), guardedObject);
        return guardedObject;
    }

    // 获取 GuardedObject 的对象的标识
    public static Set<Integer> getGuardedObjectsIds() {
        return mailBox.keySet();
    }

    // 根据 id 获取 GuardedObject 的对象，并且将其从集合中移除，防止重复使用或者重复获取
    public static GuardedObject getGuardedObject(int id) {
        return mailBox.remove(id);
    }
}

@Slf4j
class GuardedObject {
    private Object guarded;
    // 标识 GuardedObject
    private int id;

    public GuardedObject(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }

    // 获取数据
    public Object getData(long timeout) {
        synchronized (this) {
            long begin = System.currentTimeMillis();
            long passedTime = 0;
            // 校验数据是否为空
            while (guarded == null) {
                long waitTime = timeout - passedTime;
                if (waitTime <= 0) {
                    break;
                }
                try {
                    // 数据为空，进入循环等待
                    log.debug("数据为空，进入循环等待");
                    this.wait(waitTime);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                passedTime = System.currentTimeMillis() - begin;
            }
        }
        return guarded;
    }

    // 发送数据
    public void setData(Object object) {
        synchronized (this) {
            this.guarded = object;
            this.notifyAll();
        }
    }
}

```

```cmd
2026-03-29 17:09:19.265 [Thread-1] DEBUG JUC.JUCTest.Demo.D1.Person - 开始收信，信的编号：3
2026-03-29 17:09:19.265 [Thread-2] DEBUG JUC.JUCTest.Demo.D1.Person - 开始收信，信的编号：2
2026-03-29 17:09:19.268 [Thread-1] DEBUG JUC.JUCTest.Demo.D1.GuardedObject - 数据为空，进入循环等待
2026-03-29 17:09:19.265 [Thread-0] DEBUG JUC.JUCTest.Demo.D1.Person - 开始收信，信的编号：1
2026-03-29 17:09:19.268 [Thread-2] DEBUG JUC.JUCTest.Demo.D1.GuardedObject - 数据为空，进入循环等待
2026-03-29 17:09:19.268 [Thread-0] DEBUG JUC.JUCTest.Demo.D1.GuardedObject - 数据为空，进入循环等待
2026-03-29 17:09:20.265 [Thread-5] DEBUG JUC.JUCTest.Demo.D1.PostMan - 发送数据，数据编号为：1，数据内容为：内容1
2026-03-29 17:09:20.265 [Thread-4] DEBUG JUC.JUCTest.Demo.D1.PostMan - 发送数据，数据编号为：2，数据内容为：内容2
2026-03-29 17:09:20.265 [Thread-3] DEBUG JUC.JUCTest.Demo.D1.PostMan - 发送数据，数据编号为：3，数据内容为：内容3
2026-03-29 17:09:20.265 [Thread-0] DEBUG JUC.JUCTest.Demo.D1.Person - 收到信 id：1，信的内容为：内容1
2026-03-29 17:09:20.265 [Thread-1] DEBUG JUC.JUCTest.Demo.D1.Person - 收到信 id：3，信的内容为：内容3
2026-03-29 17:09:20.265 [Thread-2] DEBUG JUC.JUCTest.Demo.D1.Person - 收到信 id：2，信的内容为：内容2
```



# 九。异步模式-生产者/消费者

- 保护性暂停模式中，产生数据和消费数据是一一对应的。但是生产者和消费者不需要一一对应。
- 消息队列可以平衡生产者和消费者的线程资源。
- 生产者只生产数据，不关心数据被谁或如何消费。
- 消息对列容量有限，满了消息就进不来了，空了也没数据可以消费。

![](juc并发编程.assets/屏幕截图 2026-03-29 174905.png)

## 实现

```java
public class ThreadTestDemo22 {
    public static void main(String[] args) {
        int capacity = 2;
        MessageQueue queue = new MessageQueue(capacity);
        // 创建三个生产者线程
        for (int i = 0; i < 3; i++) {
            int id = i;
            new Thread(() -> {
                // 生产消息
                queue.put(new Message(id, "消息" + id));
            }, "生产者" + i).start();
        }

        // 创建一个消费者线程
        new Thread(() -> {
            // 消费者每隔一秒消费一个消息
            while (true) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                // 消费消息
                queue.take();
            }
        }).start();
    }
}

/**
 * 消息队列类
 */
@Slf4j
class MessageQueue {
    // 用于存放消息的集合
    private final List<Message> list = new LinkedList<>();
    // 队列的容量
    private int capacity;

    public MessageQueue(int capacity) {
        this.capacity = capacity;
    }

    // 获取消息
    public Message take() {
        // 校验消息对列此时是否为空，如果为空，证明生产者还没有生产消息到消息队列中，消费者需要循环等待消息出现
        synchronized (list) {
            while (list.isEmpty()) {
                // 消费者进入等待状态
                try {
                    log.debug("对列为空，消费者进入等待...");
                    list.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            // 从消息队列的头部获取消息，并且将消息从队列中移除
            Message message = list.removeFirst();
            log.debug("已消费消息：{}", message);
            // 通知生产者消息队列不满了
            list.notifyAll();
            return message;
        }
    }

    // 生产消息
    public void put(Message message) {
        // 校验消息对列此时是否已经满了，如果已经满了，则不能再向消息队列中添加消息
        synchronized (list) {
            while (list.size() == capacity) {
                // 生产者进入等待状态
                try {
                    log.debug("对列已满，生产者进入等待...");
                    list.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            // 向消息队列的尾部添加消息
            list.addLast(message);
            log.debug("已生产消息：{}", message);
            // 通知消费者有新的消息出现
            list.notifyAll();
        }
    }
}

/**
 * 消息类
 */
final class Message {
    // 消息 id
    private int id;
    // 消息内容
    private Object message;

    public Message(int id, Object message) {
        this.id = id;
        this.message = message;
    }

    public int getId() {
        return id;
    }

    public Object getMessage() {
        return message;
    }

    @Override
    public String toString() {
        return "Message{" +
                "id=" + id +
                ", message=" + message +
                '}';
    }
}
```

```cmd
2026-03-29 18:49:34.689 [生产者2] DEBUG JUC.JUCTest.Demo.D1.MessageQueue - 已生产消息：Message{id=2, message=消息2}
2026-03-29 18:49:34.697 [生产者1] DEBUG JUC.JUCTest.Demo.D1.MessageQueue - 已生产消息：Message{id=1, message=消息1}
2026-03-29 18:49:34.697 [生产者0] DEBUG JUC.JUCTest.Demo.D1.MessageQueue - 对列已满，生产者进入等待...
2026-03-29 18:49:35.690 [Thread-0] DEBUG JUC.JUCTest.Demo.D1.MessageQueue - 已消费消息：Message{id=2, message=消息2}
2026-03-29 18:49:35.690 [生产者0] DEBUG JUC.JUCTest.Demo.D1.MessageQueue - 已生产消息：Message{id=0, message=消息0}
2026-03-29 18:49:36.690 [Thread-0] DEBUG JUC.JUCTest.Demo.D1.MessageQueue - 已消费消息：Message{id=1, message=消息1}
2026-03-29 18:49:37.695 [Thread-0] DEBUG JUC.JUCTest.Demo.D1.MessageQueue - 已消费消息：Message{id=0, message=消息0}
2026-03-29 18:49:38.701 [Thread-0] DEBUG JUC.JUCTest.Demo.D1.MessageQueue - 对列为空，消费者进入等待...
```



# 十。Park & UnPark

## 使用

park 和 unpark 都是 LockSupport 中的静态方法：

park：暂停当前线程

unpark：恢复线程运行

```java
// 暂停线程运行
LockSupport.park()

// 恢复线程运行
LockSupport.unpark(暂停的线程)
```

```java
@Slf4j
public class ThreadTestDemo23 {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            log.debug("t1 开始运行");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug("park...");
            LockSupport.park();
            log.debug("被唤醒，继续运行");
        }, "t1");
        t1.start();

        Thread.sleep(2000);
        log.debug("unpark t1");
        LockSupport.unpark(t1);
    }
}

```

```cmd
2026-03-29 22:00:50.008 [t1] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo23 - t1 开始运行
2026-03-29 22:00:51.014 [t1] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo23 - park...
2026-03-29 22:00:52.024 [main] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo23 - unpark t1
2026-03-29 22:00:52.025 [t1] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo23 - 被唤醒，继续运行
```

## 与wait & notify的区别

- wait / notify必须配合**对象锁**使用，但是park / unpark不需要。
- park / unpark 是以**线程**为单位进行**阻塞**和**唤醒**，unpark可以精确的唤醒指定线程。但是notify / notifyAll只能随机唤醒某一个或者全部唤醒，不能精确唤醒指定线程。
- unpark有个牛逼的地方，无论是在park**之前调用**还是park**之后调用**，都能唤醒线程。

<img src="juc并发编程.assets/屏幕截图 2026-03-29 221018.png" style="zoom:67%;" />

```cmd
2026-03-29 22:10:04.039 [t1] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo23 - t1 开始运行
2026-03-29 22:10:06.039 [main] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo23 - unpark t1
2026-03-29 22:10:07.049 [t1] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo23 - park...
2026-03-29 22:10:07.050 [t1] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo23 - 被唤醒，继续运行
```

先unpark，但是最后还是还能唤醒线程。



## 原理

每个线程都有一个自己的Parker对象，由三部分组成：counter，cond，mutex

- 线程好比人，Parker好比她的背包，counter是包中的食物（0：耗尽，1：充足），cond是帐篷
- 调用park就是让线程歇会
  - 如果包里食物充足，就吃了，继续上路
  - 如果食物耗尽了，就进帐篷，休息
- 调用unpark，就是补充食物
  - 如果线程在帐篷里，就让他吃了，然后上路
  - 如果线程还在运行，就补充食物，下次再调用park的时候，直接吃了，不用停下来休息了
    - 但是空间有限，最多只能有一份食物，无论调用几次unpark，只能补充一次食物

<img src="juc并发编程.assets/屏幕截图 2026-03-29 222901.png" style="zoom:67%;" />

1.当前线程调用park

2.检查 _counter ，为0，会获得 _mutex 互斥锁

3.线程就进入了 _cond 条件变量阻塞

4.设置 _counter = 0



### 先调用park再调用unpark

先调用park，_counter = 0，Thread-0进入阻塞。

<img src="juc并发编程.assets/屏幕截图 2026-03-29 223546.png" style="zoom:67%;" />

1.调用unpark，将_counter设置为1

2.唤醒_cond条件变量中的Thread-0

3.Thread-0恢复运行

4.将_counter设置为0



### 先调用unpark再调用park

<img src="juc并发编程.assets/屏幕截图 2026-03-29 224141.png" style="zoom:67%;" />

1.调用unpark，将_counter设置为1

2.调用park，检查 _counter = 1

3.Thread-0无需阻塞，继续运行

4.将_counter设置为0

**总结：**

调用park之后会被不会受到阻塞，取决于_counter的值：

如果为0，就进入阻塞，等其他线程调用unpark。这是先park后unpark。

如果为1，无需阻塞，继续运行，并且见_counter设置为0，这是先unpark后park



# 十一。线程状态转换

<img src="juc并发编程.assets/屏幕截图 2026-03-29 225400.png" style="zoom:67%;" />

1. 初始(NEW)：新出创建了一个线程对象，但是还没有通过start（）启动
2. 运行(RUNNABLE)：Java线程中将就绪（ready）和运行中（running）两种状态笼统的称为“运行”。
线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取CPU的使用权，此时处于就绪状态（ready）。就绪状态的线程在获得CPU时间片后变为运行中状态（running）。
3. 阻塞(BLOCKED)：表示线程阻塞于锁（就是没抢到锁，抢锁呢）。
4. 等待(WAITING)：进入该状态的线程需要等待其他线程做出一些特定动作（拿到锁了，但是又放弃了，等着其他线程通知）。
5. 超时等待(TIMED_WAITING)：该状态不同于WAITING，有时效，到了时间自动被唤醒。
6. 终止(TERMINATED)：表示该线程已经执行完毕。



有线程T：

- ## 情况一：NEW --> RUNNABLE

调用 t.start( ) ，线程状态由 new --> runnable

------



- ## 情况二：RUNNABLE <--> WATTING

线程获取到对象锁之后：

- 调用 obj.wait（），t线程进入对象的 WaitSet，状态从 RUNNABLE--> WATTING
- 调用 obj.notify（） ， obj,notifyAll（） ，t.interrupt()，线程被唤醒，进入EntryList，重新争抢锁
  - 抢锁成功，状态从 WATTING--> RUNNABLE
  - 抢锁失败，状态从 WATTING--> BLOCKED（没获得锁）

------



- ## 情况三：RUNNABLE <--> WATTING

- **当前线程**调用 t.join（），**当前线程**会从RUNNABLE --> WATTING
  - 与wait不同的是，**wait是让目标线程进入等待**。而join是让**调用者进入t线程对象的监视器上等待**。
  - t 线程**结束运行**，或调用了**当前线程**的interrupt（），当前线程会从 WATTING --> RUNNABLE

------



- ## 情况四：RUNNABLE <--> WATTING

- 当前线程调用 LockSupport.park（），会让当前线程从 RUNNABLE --> WATTING
- 调用LockSupport.unpark（），或者调用了线程的 interrupt（），会让线程从 WATTING --> RUNNABLE

------



- ## 情况 五： RUNNABLE <--> TIMED_WAITING

t 线程用 `synchronized(obj)` 获取了对象锁后

- 调用 `obj.wait(long n)` 方法时，t 线程从 `RUNNABLE` --> `TIMED_WAITING`
- t 线程等待时间超过了 n 毫秒，或调用 obj.notify(), obj.notifyAll(), t.interrupt() 时
  - 竞争锁成功，t 线程从 `TIMED_WAITING` --> `RUNNABLE`
  - 竞争锁失败，t 线程从 `TIMED_WAITING` --> `BLOCKED`

------

- ## 情况 六： RUNNABLE <--> TIMED_WAITING

- 当前线程调用 `t.join(long n)` 方法时，当前线程从 `RUNNABLE` --> `TIMED_WAITING`
- 注意是当前线程在 t 线程对象的监视器上等待
- 当前线程等待时间超过了 n 毫秒，或 t 线程运行结束，或调用了当前线程的 `interrupt()` 时，当前线程从 `TIMED_WAITING` --> `RUNNABLE`

------

- ## 情况 7七：RUNNABLE <--> TIMED_WAITING

- 当前线程调用 `Thread.sleep(long n)`，当前线程从 `RUNNABLE` --> `TIMED_WAITING`
- 当前线程等待时间超过了 n 毫秒，当前线程从 `TIMED_WAITING` --> `RUNNABLE`

------

- ## 情况 八：RUNNABLE <--> TIMED_WAITING

- 当前线程调用 `LockSupport.parkNanos(long nanos)` 或 `LockSupport.parkUntil(long millis)` 时，当前线程从 `RUNNABLE` --> `TIMED_WAITING`

- 调用 `LockSupport.unpark(目标线程)` 或调用了线程的 `interrupt()`，或是等待超时，会让目标线程从 `TIMED_WAITING` --> `RUNNABLE`

------

- ## 情况 九： RUNNABLE <--> BLOCKED

- t 线程用 `synchronized(obj)` 获取了对象锁时如果竞争失败，从 `RUNNABLE` --> `BLOCKED`
- 持 obj 锁线程的同步代码块执行完毕，会唤醒该对象上所有 `BLOCKED` 的线程重新竞争，如果其中 t 线程竞争成功，从 `BLOCKED` --> `RUNNABLE`，其它失败的线程仍然 `BLOCKED`

------

- ## 情况 十： RUNNABLE <--> TERMINATED

- 当前线程所有代码运行完毕，进入 `TERMINATED`



# 十二。多把锁

```java
public class ThreadTestDemo25 {
    public static void main(String[] args) {
        BigRoom bigRoom = new BigRoom();
        new Thread(() -> bigRoom.sleep(), "睡觉线程").start();
        new Thread(() -> bigRoom.study(), "学习线程").start();
    }
}

@Slf4j
class BigRoom {

    public void sleep() {
        synchronized (this) {
            log.debug("睡觉");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
    }

    public void study() {
        synchronized (this) {
            log.debug("学习");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
    }
}
```

睡觉和学习本身是两件不相干的事情，但是为了安全性通过上锁的方式，使得他们只能串行操作，但是并发性能太低了。

只能是要先睡觉再学习，或者先学习在睡觉。

```cmd
2026-03-30 14:46:16.962 [睡觉线程] DEBUG JUC.JUCTest.Demo.D1.BigRoom - 睡觉
2026-03-30 14:46:17.968 [学习线程] DEBUG JUC.JUCTest.Demo.D1.BigRoom - 学习
```



可以使用多把锁，把**锁的粒度细分**，相当于把房间分成了卧室和书房，这样线程**只需要把自己的房间上锁，无需把整个房子锁住**，保证了安**全性还**提高了并发性。

```java
public class ThreadTestDemo25 {
    public static void main(String[] args) {
        BigRoom bigRoom = new BigRoom();
        new Thread(() -> bigRoom.sleep(), "睡觉线程").start();
        new Thread(() -> bigRoom.study(), "学习线程").start();

        new Thread(() -> bigRoom.sleep(), "睡觉线程2").start();
        new Thread(() -> bigRoom.study(), "学习线程2").start();
    }
}

@Slf4j
class BigRoom {
    private static final Object room1 = new Object();
    private static final Object room2 = new Object();

    public void sleep() {
        synchronized (room1) {
            log.debug("睡觉");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
    }

    public void study() {
        synchronized (room2) {
            log.debug("学习");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
    }
}
```

```cmd
2026-03-30 14:53:07.285 [睡觉线程] DEBUG JUC.JUCTest.Demo.D1.BigRoom - 睡觉
2026-03-30 14:53:07.285 [学习线程] DEBUG JUC.JUCTest.Demo.D1.BigRoom - 学习
2026-03-30 14:53:08.288 [学习线程2] DEBUG JUC.JUCTest.Demo.D1.BigRoom - 学习
2026-03-30 14:53:08.288 [睡觉线程2] DEBUG JUC.JUCTest.Demo.D1.BigRoom - 睡觉
```



## 活跃性

线程内的代码是有限的，但是因为某种原因，一直执行不完，这就是线程的活跃性。



### 死锁

两个线程都获得了各自的锁，但是双方在占有锁期间，又想获得对方的锁，导致锁一直得不到释放，线程双方一直被阻塞，产生了死锁。

例如：

t1获得锁A，接下来想获得锁B。

t2获得锁B，接下来想获得锁A。

```java
@Slf4j
public class ThreadTestDemo26 {
    public static void main(String[] args) {
        test1();
    }

    public static void test1() {
        Object A = new Object();
        Object B = new Object();

        new Thread(() -> {
            synchronized (A) {
                log.debug("获得A锁");
                synchronized (B) {
                    log.debug("获得B锁");
                }
            }
        }, "t1").start();

        new Thread(() -> {
            synchronized (B) {
                log.debug("获得B锁");
                synchronized (A) {
                    log.debug("获得A锁");
                }
            }
        }, "t2").start();
    }
}

```

```cmd
2026-03-30 15:36:17.145 [t2] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo26 - 获得B锁
2026-03-30 15:36:17.145 [t1] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo26 - 获得A锁
```

产生死锁



### 活锁

线程双方都想改变对方的结束条件，最后谁也无法结束。

与死锁不同的是，死锁是线程一直得不到对方线程占有的锁，导致线程一直被阻塞，无法运行。

而活锁是线程一直在运行，但是却一直在改变对方线程的结束条件，导致一直无法结束运行。

```java
@Slf4j
public class ThreadTestDemo27 {
    static int count = 10;
    public static void main(String[] args) {
        new Thread(() -> {
            while (count > 0){
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                count--;
                log.debug("count:{}",count);
            }
        }).start();

        new Thread(() -> {
            while (count < 20){
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                count++;
                log.debug("count:{}",count);
            }
        }).start();
    }
}
```

可以看到，一直无法结束运行

![](juc并发编程.assets/屏幕截图 2026-03-30 163241.png)



### 饥饿

是指一个可运行的线程尽管能继续执行，但被无限期地忽视，而不能被调度执行的情况。举个例子：线程1占用了资源R，线程2此时申请资源R需要等待，后来线程3来申请资源R的时候，线程1刚好释放了资源R，结果资源R又被线程3占用了。再后来线程4来了也是同样结果，导致线程2一直无法获得资源R，一直等待。

导致饥饿的原因有以下几点：

1. [线程优先级](https://zhida.zhihu.com/search?content_id=165661259&content_type=Article&match_order=1&q=线程优先级&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NzUwMzI2NjksInEiOiLnur_nqIvkvJjlhYjnuqciLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoxNjU2NjEyNTksImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.5Cd_1-sg5EVZxlDjLALNMZcUgk0Od7U8BYj_doJ0XmQ&zhida_source=entity)。有些低优先级的线程可能一直被高优先级线程占用资源，导致无法获取CPU资源。
2. 线程被永久堵塞在一个等待进入同步块的状态，因为其他线程总是能在它之前持续地对该同步块进行访问。
3. [线程](https://zhida.zhihu.com/search?content_id=165661259&content_type=Article&match_order=17&q=线程&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NzUwMzI2NjksInEiOiLnur_nqIsiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoxNjU2NjEyNTksImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MTcsInpkX3Rva2VuIjpudWxsfQ.YPim9VMGNyUBSYLe0xgaUZV9fvEZpP5_kshbxJDwBkg&zhida_source=entity)在等待一个本身也处于永久等待完成的对象(比如调用这个对象的 wait 方法)，因为其他线程总是被持续地获得唤醒。



# 十三。ReentrantLock 可重入锁

与synchronized对比：

- 可中断：使用synchronized加上锁之后不能中断，其他线程没法把持锁线程的锁破换掉。ReentrantLock 可以中断。
- 可以设置超时时间：规定时间内如果没有抢到锁，就不抢了，执行其他的逻辑。
- 可以设置为公平锁：可以防止饥饿现象。
- 支持多个条件变量：synchronized中只有一个 WaitSet，无论线程想要获取什么条件都得往一个WaitSet中等着。但是ReentrantLock可以有多个。
- synchronized是JVM层面实现的，ReentrantLock是Java层面实现的。
- 相同点：都支持可重入

## 使用

先创建ReentranLock对象，通过对象获取锁。

```java
ReentrantLock lock = new Reentrant Lock();
// 获取锁
reentratLock.unlock()
try {
	// 临界区代码
} finallly {
	// 释放锁操作
    reentratLock.unlock()
}
```



## 可重入

一个线程如果首次获得了对象锁，当他再次获取对象锁的时候依然能够获取到这把锁，这就是可重入.

```java
@Slf4j
public class ThreadTestDemo28 {
    static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        lock.lock();
        try {
            log.debug("第一次获取到锁");
            m1();
        } finally {
            lock.unlock();
        }
    }

    public static void m1() {
        lock.lock();
        try {
            log.debug("第二次获取到锁");
            m2();
        } finally {
            lock.unlock();
        }
    }

    public static void m2() {
        lock.lock();
        try {
            log.debug("第三次获取到锁");
        } finally {
            lock.unlock();
        }
    }
}
```

```cmd
2026-03-30 21:24:03.720 [main] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo28 - 第一次获取到锁
2026-03-30 21:24:03.722 [main] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo28 - 第二次获取到锁
2026-03-30 21:24:03.722 [main] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo28 - 第三次获取到锁
```



## 可打断

A线程如果没有抢到 lock 锁，就会进入lock的阻塞队列等待抢锁。如果不想让他死等，这时候其他线程调用interrupt方法可以打断A线程等待，就是让A被再等了。

前提是 A调用的是 lockInterruptibly（）方法，而不是lock，lock方法跟synchronized一样，没办法打断。

```java
public class ThreadTestDemo29 {
    static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            // 获取锁
            try {
                log.debug("尝试获取锁");
                lock.lockInterruptibly();
                log.debug("成功获取锁");
            } catch (InterruptedException e) {
                // 其他线程调用interrupt方法打断t1线程会抛出InterruptedException异常，进入catch块
                log.debug("获取锁失败，不再等锁了");
                throw new RuntimeException(e);
            } finally {
                lock.unlock();
                log.debug("释放锁");
            }
        }, "t1");

        // 先让主线程获取锁
        lock.lock();
        t1.start();

        log.debug("主线程获取到锁");
        Thread.sleep(2000);
        // 打断t1线程等待得得到锁
        log.debug("打断t1线程等锁");
        t1.interrupt();
    }
}
```

![](juc并发编程.assets/屏幕截图 2026-03-30 214807.png)



## 锁超时

lockInterruptbily 配合 interrupt确实能打断，但是是一种**被动的**方式，等待线程必须由其他线程调用打断方法完成打断。

**锁超时**是一种**主动**的方式，由等待线程**自己决定**等不等，等的话等多久，如果等待期间没有青抢到锁，就主动放弃抢锁。



### tryLock（）

没有传入超时时间，就等一次，第一次没抢到，就直接退出，不等了

成功情况：

```java
@Slf4j
public class ThreadTestDemo30 {
    private static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            // 尝试获取锁，没有传入超时时间，只尝试一次，没有抢到就主动的放弃
            boolean isGetLock = lock.tryLock();
            if (isGetLock) {
                try {
                    log.debug("成功抢到锁");
                } finally {
                    log.debug("释放锁");
                    lock.unlock();
                }
            } else {
                log.debug("没有抢到锁");
            }
        });
        t1.start();
    }
}
```

```cmd
2026-03-30 22:05:16.176 [Thread-0] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo30 - 成功抢到锁
2026-03-30 22:05:16.178 [Thread-0] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo30 - 释放锁
```

没抢到锁的情况：

```java
@Slf4j
public class ThreadTestDemo30 {
    private static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            // 尝试获取锁，没有传入超时时间，只尝试一次，没有抢到就主动的放弃
            boolean isGetLock = lock.tryLock();
            if (isGetLock) {
                try {
                    log.debug("成功抢到锁");
                } finally {
                    log.debug("释放锁");
                    lock.unlock();
                }
            } else {
                log.debug("没有抢到锁");
            }
        },"t1");
        lock.lock();
        t1.start();
    }
}

```

```cmd
2026-03-30 22:15:36.080 [t1] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo30 - 没有抢到锁
```



### tryLock（long timeout, TimeUnit unit）

等待时间内如果没有抢到锁就不等了。

```java
@Slf4j
public class ThreadTestDemo30 {
    private static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            // 尝试获取锁，没有传入超时时间，只尝试一次，没有抢到就主动的放弃
            try {
                if (lock.tryLock(2, TimeUnit.SECONDS)) {
                    try {
                        log.debug("t1 成功抢到锁");
                    } finally {
                        log.debug("t1 释放锁");
                        lock.unlock();
                    }
                } else {
                    log.debug("t1没有抢到锁");
                }
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        },"t1");
        log.debug("主线程获取锁");
        lock.lock();
        t1.start();
        Thread.sleep(1000);
        lock.unlock();
    }
}
```

```cmd
2026-03-30 22:23:35.839 [main] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo30 - 主线程获取锁
2026-03-30 22:23:36.848 [t1] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo30 - t1 成功抢到锁
2026-03-30 22:23:36.849 [t1] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo30 - t1 释放锁
```



## 公平锁

synchronized中的锁是不公共的，所有在EntryLIst中的线程，只要一抢锁都是一拥而上，不是说你是先进入阻塞队列中的就让你先获得锁。

ReentrantLock 默认是不公共的，但是可以通过构造函数设置为公平的。

```java
 public ReentrantLock(boolean fair) {
     sync = fair ? new FairSync() : new NonfairSync();
 }
```

但是最好别开，没必要，会降低并并发度。



## 条件变量

synchronized中的 WatiSet就是一个条件变量

ReentrantLock中的条件变量比synchronized更牛逼的地方在于，它支持多个条件变量。

比如说：synchronized中，有的线程等烟，有的等饭，但是他们是能在同一个WatiSet中等待。

但是ReentrantLock有多个，可以有专门等烟的，专门等饭的，这样唤醒线程的时候，只需要到专门的房间去唤醒就可以了，从而避免了虚假唤醒。

语法：

先创建条件变量对象，通过ReentrantLock对象创建

```java
static ReentrantLock lock = new ReentrantLock();
// 创建条件变量
Condition condition1 = lock.newCondition();
Condition condition2 = lock.newCondition();
```

### await

- **await**：调用之后，进房间阻塞等待，但是调用前**需要先获得 lock 锁**。
  - 调用await之后，会**释放掉lock锁**，进入ConditionObject等待。
- 被await的线程如果被其他线程**唤醒**或**打断**，会重新争抢lock锁。
- 抢到锁之后，接着await之后的没执行的代码**继续执行**。

### signal / singnalAll

- 调用signal之后，会去特定的条件变量中，也就是专门的房间中去唤醒某一个线程。
- signalAll是去特定的房间中唤醒全部线程。

```java
public class ThreadTestDemo31 {
    static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) throws InterruptedException {
        // 创建条件变量
        Condition condition1 = lock.newCondition();
        Condition condition2 = lock.newCondition();
        // 进入休息室前需要先获得锁
        lock.lock();
        // 进入休息室
        condition1.await();
    }
}

```

例如：

```java
@Slf4j
public class ThreadTestDemo32 {
    // 创建房间
    static ReentrantLock Room = new ReentrantLock();
    static boolean isHasSmoke = false;
    static boolean isHasMeal = false;

    // 创建条件变量
    static Condition smokeWaitSet = Room.newCondition(); // 等烟房间
    static Condition mealWaitSet = Room.newCondition(); // 等饭房间


    public static void main(String[] args) throws InterruptedException {
        // 等烟线程
        Thread t1 = new Thread(() -> {
            // 1.获取锁
            Room.lock();
            try {
                // 2.抢到锁之后，查看是否有烟
                while (!isHasSmoke) {
                    log.debug("是否有烟：{}", isHasSmoke);
                    try {
                        // 3.没有烟，进入等烟房间等待
                        smokeWaitSet.await();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("是否有烟：{}，开始吸烟", isHasSmoke);
            } finally {
                // 5.释放锁
                Room.unlock();
            }
        }, "吸烟者");

        // 等饭线程
        Thread t2 = new Thread(() -> {
            // 1.获取锁
            Room.lock();
            try {
                // 2.抢到锁之后，查看是否有饭
                while (!isHasMeal) {
                    log.debug("是否有饭：{}", isHasMeal);
                    try {
                        // 3.没有烟，进入等烟房间等待
                        mealWaitSet.await();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("是否有饭：{}，开始吃饭", isHasMeal);
            } finally {
                // 5.释放锁
                Room.unlock();
            }
        }, "吃饭者");

        t1.start();
        t2.start();
        Thread.sleep(1000);

        // 送烟线程
        new Thread(() -> {
            // 1.获取锁
            Room.lock();
            try {
                // 2.抢到锁之后，设置为有烟
                isHasSmoke = true;
                // 3.唤醒等烟线程
                log.debug("开始送烟");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                smokeWaitSet.signalAll();
            } finally {
                // 4.释放锁
                Room.unlock();
            }
        },"送烟者").start();

        Thread.sleep(1000);

        // 送饭线程
        new Thread(() -> {
            // 1.获取锁
            Room.lock();
            try {
                // 2.抢到锁之后，设置为有烟
                log.debug("开始送饭");
                isHasMeal = true;
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                // 3.唤醒等烟线程
                mealWaitSet.signalAll();
            } finally {
                // 4.释放锁
                Room.unlock();
            }
        },"送饭者").start();
    }
}
```

```
2026-03-31 21:32:28.903 [吸烟者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo32 - 是否有烟：false
2026-03-31 21:32:28.906 [吃饭者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo32 - 是否有饭：false
2026-03-31 21:32:29.921 [送烟者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo32 - 开始送烟
2026-03-31 21:32:30.933 [吸烟者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo32 - 是否有烟：true，开始吸烟
2026-03-31 21:32:30.933 [送饭者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo32 - 开始送饭
2026-03-31 21:32:31.939 [吃饭者] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo32 - 是否有饭：true，开始吃饭
```



# 十四。同步模式-控制线程运行次序



## 固定执行次序

线程t1想打印1，线程t2想打印2，现在要求必须的先打印2，在打印1.

如果不控制，谁都有可能先打印。

解决办法：



### **wait + notify**

可以设置一个变量，例如：t2HasRunned，代表t2已经运行打印完了。

执行流程：两个线程先去抢通一把锁：

抢到之后各自需要干的事情：

- t2 ：打印2，并且将 **t2HasRunned 设置为true**，代表自己已经执行过了，之后唤醒t1.

- t1：检查 t2HasRunned 的值
  - 为true，代表t2已经执行完了，自己打印1完事。
  - 为false，代表t2还没有执行完，调用wait，进入阻塞，等待t2执行完唤醒自己。

分两种情况：

- t1线程先抢到锁，发现t2HasRunned为false，就进入等待，等待被t2唤醒.
- t2线程先抢到锁，打印2，并将t2HasRunned设置为true，唤醒t1。

```java
public class ThreadTestDemo33 {
    private static final Logger log = LoggerFactory.getLogger(ThreadTestDemo33.class);
    static Object lock = new Object();
    static boolean t2HasRunned = false;

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            synchronized (lock) {
                // 判断t2线程是否已经执行完毕
                while (!t2HasRunned) {
                    // 没执行完成，进入等待
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("1");
            }
        }, "t1");
        Thread t2 = new Thread(() -> {
            synchronized (lock) {
                log.debug("2");
                t2HasRunned = true;
                lock.notify();
            }
        }, "t2");
        t1.start();
        t2.start();
    }
}
```



### await + signal

```java
@Slf4j
public class ThreadTestDemo34 {
    static ReentrantLock lock = new ReentrantLock();
    static Condition condition = lock.newCondition();
    static boolean t2HasRunned = false;

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            lock.lock();
            try {
                // 判断t2线程是否已经执行完毕
                while (!t2HasRunned) {
                    // 没执行完成，进入等待
                    try {
                        condition.await();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("1");
            } finally {
                lock.unlock();
            }
        }, "t1");
        Thread t2 = new Thread(() -> {
            lock.lock();
            try {
                log.debug("2");
                t2HasRunned = true;
                condition.signal();
            } finally {
                lock.unlock();
            }
        }, "t2");
        t1.start();
        t2.start();
    }
}
```



### park + unpark

分两种情况：

- t1先运行，执行park，，发现干粮没了，就停下休息，受到阻塞，等着t2补充干粮。
- t2先运行，打印2，执行unpark，补充干粮；下次t1调用park，发现干粮充足，无需等待，直接打印1.

```java
@Slf4j
public class ThreadTestDemo35 {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            LockSupport.park();
            log.debug("1");
        }, "t1");
        Thread t2 = new Thread(() -> {
            log.debug("2");
            LockSupport.unpark(t1);
        }, "t2");
        t1.start();
        t2.start();
    }
}
```

这种方式实现最简单。





# 十五。共享模型之内存

## 1.Java内存模型

JMM，定义了**主存**（所有线程都共享的数据），**工作内存**（线程私有的局部变量）这些抽象概念，底层就是对应着CPU，寄存器，缓存器这些。

JMM规定，所有的变量（实例变量和静态变量）都必须存储在**主存**中；每个线程都会有自己的工作内存，工作内存中用到的数据变量是主存中的数据的拷贝副本。

线程对数据的读写都是在自己的工作内存中。

<img src="juc并发编程  1.assets/579981643-7e89c9e5096439ee_fix732.webp" style="zoom:67%;" />

JMM的体现：

- **原子性**：保证指令不受线程上下文切换的影响
- **可见性**：保证指令不受cpu缓存的影响
- **有序性**：保证指令不受cpu指令并行优化的影响



## 2.可见性

内存的可见性问题：一个线程对主存中的数据进行了修改，但是这个修改对于另外一个线程不可见，其他的线程无法感知数据已经更改了，他会一直使用自己的工作内存中的旧数据。

例子：

### 退不出的循环

```java
@Slf4j
public class ThreadTestDemo36 {
    static boolean run =true;

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            while (run) {

            }
            log.debug("结束运行");
        }).start();

        Thread.sleep(1000);
        log.debug("尝试停止线程");
        run = false;
    }
}

```

按理来说，1s之后，主线程将a设置为了false，副线程应该停止循环，打印“结束运行”，并结束程序。

但是实际上并没有用结束运行。

并没由打印“结束运行”，程序也没有停止运行。

![](juc并发编程  1.assets/屏幕截图 2026-04-01 175936.png)

分析：

1.初始状态，t线程从主从中读取 run 的值到自己的工作内存中。

![](juc并发编程  1.assets/屏幕截图 2026-04-01 180210.png)

2.但是t线程中有个 while 循环，这就需要不断地从主存读取数据到线程的工作内存中。JIT编译器就认为这样读取效率太低了，他就会把这个 run 缓存到线程的工作内存中，之后再读这个数据就直接从线程自己的工作内存中读就行了，省事了。

![](juc并发编程  1.assets/屏幕截图 2026-04-01 180822.png)

3.一秒之后，main线程从主内存中读取 run到自己的工作内从中，并且修改了run的值，之后在写回主内存中。

可是，t线程读取到的 run 此时都是基于自己的工作内存中的缓存值，读取到的永远都是旧的。

![](juc并发编程  1.assets/屏幕截图 2026-04-01 181311.png)



### 解决办法

#### volatile

volatile：可以修饰成员变量，静态成员变量。

作用：让线程读数据的时候，只能到主存中读，不能去自己的工作内存中读数据。

这就保证了线程读取到的数据是最新的，虽然性能有所损失。

```java
@Slf4j
public class ThreadTestDemo36 {
    volatile static boolean a =true;

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            while (a) {

            }
            log.debug("结束运行");
        }).start();

        Thread.sleep(1000);
        log.debug("尝试停止线程");
        a = false;
    }
}
```

![屏幕截图 2026-04-01 181807](juc并发编程  1.assets/屏幕截图 2026-04-01 181807.png)

已经能结束了，并且读到了最新的 run 的值。



#### synchronized

使用synchronized也可以解决：

1. **进入同步块时**：线程会清空工作内存，从主内存重新加载共享变量的最新值
2. **退出同步块时**：线程会把修改过的共享变量刷新到主内存

```java
@Slf4j
public class ThreadTestDemo36 {
    static Object obj = new Object();
    static boolean run =true;

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            while (run) {
                synchronized (obj) {
                    if (!run) {
                        break;
                    }
                }
            }
            log.debug("结束运行");
        }).start();

        Thread.sleep(1000);
        synchronized (obj) {
            log.debug("尝试结束 t 线程运行");
            run = false;
        }
    }
}

```

![](juc并发编程  1.assets/屏幕截图 2026-04-01 183354.png)



## 3.可见性 & 原子性

volatile能够保证可见性，但是不能保证原子性。

它适用于一个线程负责写数据，其他多个线程负责读数据的场景，它能够保证线程每次读到的数据都是主存中最新的数据。

但是多个线程对同一个数据进行写操作的场景，volatiie就不能保证原子性了。

比如说：两个线程要进行++操作，一个数据初始值为0，两次++操作，最终应该为2。

两个线程同时进来，都读取到了最新的数据为0，同时执行了++，都为1，并且写回主存，结果最终结果为1，而不是2。

因为 volatie 他只能保证可见性，保证不了原子性。

如果既想保证可见性又想保证原子性，还是得用synchronized。

虽然 synchronized 性能稍差，但是安全。



## 4.两阶段终止模式（二）

可以设置一个标志变量 isStop，代表是否中断线程。

- 监控线程执行监控之间先判断 isStop 的值，如果为 false，代表没有被终止，继续执行监控。
  - 如果为true，代表已经被中断了，就去料理后事，并退出循环监控
- 终止线程将isStop的值设置为 true，代表要中断监控线程，这样下次监控线程拿到 isStop一看为true，就退出循环。
- 为了保证可见性，使得监控线程读取到的 isStop 是最新的值，要给 isStop 添加 volatile 关键字。

```java
@Slf4j
public class ThreadTestDemo37 {
    public static void main(String[] args) throws InterruptedException {
        TwoPhaseTermination2 twoPhaseTermination = new TwoPhaseTermination2();
        twoPhaseTermination.start();

        Thread.sleep(3500);

        //停止监控
        twoPhaseTermination.stop();
    }
}

@Slf4j
class TwoPhaseTermination2 {
    private Thread monitor;
    private volatile boolean isStop = false;

    // 启动监控线程
    public void start() {
        monitor = new Thread(() -> {
            // 进行监控
            while (true) {
                // 获取当前线程
                Thread currentThread = Thread.currentThread();
                if (isStop) {
                    log.debug("料理后事");
                    break;
                }
                log.debug("持续监控");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        });
        //启动线程
        monitor.start();
    }

    // 停止监控线程
    public void stop() {
        log.debug("停止监控");
        isStop = true;
    }
}
```

![](juc并发编程  1.assets/屏幕截图 2026-04-01 190541.png)



## 5.犹豫模式

**Balking** ，**犹豫模式**，它是指 ：如果一个线程要去做某一件事（比如调用某个方法），但是他发现这件事已经被其他线程做过了，那他就不做了，直接结束。

比如：监控这件事，只要有一个线程去做就可以了。

但是如果多次调用 twoPhaseTermination.start() 方法就会创建多个线程同时去监控。

![](juc并发编程  1.assets/屏幕截图 2026-04-01 210705.png)

![屏幕截图 2026-04-01 210803](juc并发编程  1.assets/屏幕截图 2026-04-01 210803.png)

解放方法：犹豫模式。

实现：

可以设置一个标记 hasRunned ，代表这件事或者这个方法是否已经执行过了。

- hasRunned  = false，就是没执行过
- hasRunned  = true，就是已经执行过了

start方法内部，当有线程调用了 start 方法，先检查 hasRunned  的值

- 如果 hasRunned = false，代表其他线程还没有执行过这件事，就让这个线程正常执行即可。并且同时将hasRunned 设置为 true，这样其他线程调用的时候也检查hasRunned，发现为true，就知道这件事已经执行过了，不用再做了，直接结束。
- 如果 hasRunned  = true，代表其他线程已经执行过了，直接结束即可。
- 为了原子性与可见性，使用synchronized进行加锁，防止两个线程同时调用start方法，发现 hasRunned都为false，结果都执行了start方法，造成重复执行。
- 而且不能用 volatie 修饰 hasRunned，因为 volattie只能保证可见性不能保证原子性。

```java
@Slf4j
class TwoPhaseTermination2 {
    // 监控线程
    private Thread monitor;
    // 停止标记
    private volatile boolean isStop = false;
    // 执行标识
    private boolean hasRunned = false;

    // 启动监控线程
    public void start() {
        synchronized (this) {
            // 判断是否已经执行过了
            if (hasRunned) {
                return;
            }
            // 未执行，将hasRunned设置为true，代表已经执行过了
            hasRunned = true;
        }
        monitor = new Thread(() -> {
            // 进行监控
            while (true) {
                // 获取当前线程
                Thread currentThread = Thread.currentThread();
                if (isStop) {
                    log.debug("料理后事");
                    break;
                }
                log.debug("持续监控");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        });
        //启动线程
        monitor.start();
    }

    // 停止监控线程
    public void stop() {
        log.debug("停止监控");
        isStop = true;
    }
}
```

![](juc并发编程  1.assets/屏幕截图 2026-04-01 212928.png)



## 6.有序性

JVM会在不影响正确性的前提下，可以调整语句的执行顺序

```java
static int i;
static int j;

i = 1;
j = 2;
```

这两个赋值操作，无论先执行哪个，对结果都没有影响。

所以可能先执行 i=1，也有可能先执行 j=1。

这种调整了语句的执行顺序，也叫 **指令重排**。

单线程模式下指令重排没啥问题，但是多线程模式下就有问题了。



## 7.volatile原理

volatile的底层实现原理是**内存屏障**

- 对volatile变量的写操作后会加入写屏障
- 对volatile变量的读操作前会加入读屏障



### 保证可见性

#### 1.**写屏障**

- 写屏障之前的，对共享变量的改动，都会同步到主存当中，保证了主存中永远是最新的数据。

- ready被volatie修饰，num没有，但是写屏障会顺带把 num 也写回主存

```java
public void write() {
	num++;
	ready = true;
    // 写屏障
}
```

#### 2.读屏障

- 读屏障之后的，对共享变量的读取，都是加载的主存当中的最新数据。

```java
public void read() {
    // 读屏障
	if(ready) {
		...
	} else {
		num++;
	}
}
```



### 保证有序性

- 写屏障能保障指令重拍的时候，不会将屏障之前的代码排在屏障之后
- num++不会跑到 ready = true 之后

```java
public void write() {
	num++;
	ready = true;
    // 写屏障
}
```

- 读屏障能偶保证指令重排的时候，屏障之后的代码不会跑到屏障之前

```java
public void read() {
    // 读屏障
	if(ready) {
		...
	} else {
		num++;
	}
}
```



## 8.double-checkled locking

```java
public final class Singleton {
    private Singleton() { }
    private static Singleton INSTANCE = null;
    
    public static Singleton getInstance() {
        synchronized(Singleton.class) {
            if (INSTANCE == null) {
                INSTANCE = new Singleton();
            }
            return INSTANCE;
        }
    }
}
```

这是个懒惰实列化，INSTANCE初始化时为空，之后通过getInstance进行实例化。

为了实现单例模式，通过synchronized进行上锁，并且校验INSTANCE是否为空，只有为空才实例化，保证了单例模式。

但是问题是每调用一次getInstance，就进行一次synchronized，实际上只有第一次进行实例化的时候才这真正用到 synchronized。这性能就很低。

double-checkled locking 单列模式

```java
public final class Singleton {
    private Singleton() { }
    private static Singleton INSTANCE = null;
    public static Singleton getInstance() {
        // 首次访问会同步，而之后的使用没有 synchronized
        if(INSTANCE == null) {
            synchronized(Singleton.class) {
                if (INSTANCE == null) {
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }
}
```

特点：

- 懒惰实例化
- 首次使用getInstance才使用synchronized，后续调用无需加锁
- 隐秘的一点：第一个if使用的INSTANCE实在同步代码块之外的

这种方法在多线程环境下有问题。

主要是因为第一个if使用的INSTANCE实在同步代码块之外的，他不受synchronized的保护，又由于指令重排序的可能，会造成问题，



## 9.double-checkled locking 问题解决

可以在INSTANCE加上volatile，保证顺序性（禁止指令重排序）

```java
![屏幕截图 2026-04-02 162623](../../Pictures/Screenshots/屏幕截图 2026-04-02 162623.png)public final class Singleton {
    private Singleton() { }
    private static volatile Singleton INSTANCE = null;
    public static Singleton getInstance() {
        // 首次访问会同步，而之后的使用没有 synchronized
        if(INSTANCE == null) {
            synchronized(Singleton.class) {
                if (INSTANCE == null) {
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }
}
```



## 10.happense-before规则

happense-before 规定了**线程对共享变量的写操作对其他线程对该变量的读操作是可见的的情况**。

如果不遵守happense-before ，JMM就不能保证线程对某个共享变量的写操作对其他线程的可见性。



- 使用synchronized加锁，线程**解锁之前对变量的写**，对于接下来**线程加锁之后对变量的读**，是可见的

```java
static int x;
static Object m = new Object();

new Thread(() -> {
    synchronized(m) {
        x = 10;
    }
}, "t1").start();

new Thread(() -> {
    synchronized(m) {
        System.out.println(x);
    }
}, "t2").start();
```



- 变量加了 **volatile** 修饰，写操作之后对读线程是可见的。

```java
volatile static int x;

new Thread(() -> {
    x = 10;
}, "t1").start();

new Thread(() -> {
    System.out.println(x);
}, "t2").start();
```

- **调用start（）启动线程之前**，对共享变量的写，对线程是**可见的**。

```java
 static int x;
 x = 10;

 new Thread(() -> {
    System.out.println(x);
}, "t2").start();
```

- 线程**结束之前，对变量写**，当**线程结束之后**，对其他线程的读是**可见的**。

  比如 t 线程写x，主线程等待 t 线程结束，结束之后，主线程读读到的 x 是修改之后的u最新值

```java
static int x;

Thread t1 = new Thread(() -> {
    x = 10;
}, "t1");
t1.start();

t1.join();
System.out.println(x);
```

- 线程 t1 调用 interrupt 打断 t2 之前，对共享变量进行了写操作，当调用 interrupt 之后，对于被中断线程在检测到中断状态**之后**的读操作是可见的。

  就是说中断之前进行了改，中断之后中断线程要是读，能够读到这个值。但是如果中断之后，又改了，就不能保证可见性了。

  - 中断**之前**的所有写操作，对于中断**之后**的**读**操作是可见的
  - 但中断**之后**的写操作，没有建立相应的同步关系

```java
static int x;

public static void main(String[] args) {
    Thread t2 = new Thread(() -> {
        while(true) {
            if(Thread.currentThread().isInterrupted()) {
                System.out.println(x);
                break;
            }
        }
    }, "t2");
    t2.start();

    new Thread(() -> {
        sleep(1);
        x = 10;
        t2.interrupt();
    }, "t1").start();

    while(!t2.isInterrupted()) {
        Thread.yield();
    }
    System.out.println(x);
}
```

- 对变量的默认值（0，false，null）进行写操作，对其他线程的读是可见的。
- 具有传递性，如果操作A happens-before操作B，且操作B happens-before操作C，那么操作A happens-before操作C。A → B 且 B → C，则 A → C

```java
volatile static int x;
static int y;

new Thread(() -> {
    y = 10;
    x = 20;
}, "t1").start();

new Thread(() -> {
    // x=20 对 t2 可见，同时 y=10 也对 t2 可见
    System.out.println(x);
}, "t2").start();
```



# 十六。共享模型之无锁

## 1.问题

```java
public class ThreadTestDemo38 {
    public static void main(String[] args) {
        Account account = new AccountUnsafeImpl(10000);
        Account.demo(account);
    }
}

class AccountUnsafeImpl implements Account {
    // 余额
    private Integer balance;

    public AccountUnsafeImpl(Integer balance) {
        this.balance = balance;
    }

    @Override
    public Integer getBalance() {
        return this.balance;
    }

    @Override
    public void withdraw(Integer amount) {
        this.balance -= amount;
    }
}

interface Account {
    // 获取余额
    Integer getBalance();

    // 扣减余额
    void withdraw(Integer amount);

    // 演示方法
    static void demo(Account account) {
        // 创建一个集合，用于存储1000个线程你
        List<Thread> accountThreads = new ArrayList<>();
        // 创建1000个线程，并调用扣减余额的方法，并将线程添加到集合中
        for (int i = 0; i < 1000; i++) {
            accountThreads.add(new Thread(() -> {
                // 每个线程扣减10元
                account.withdraw(10);
            }));
        }
        // 记录启动线程开始扣减余额的时间点
        long beginTime = System.nanoTime();
        // 启动集合中的1000个线程
        for (Thread accountThread : accountThreads) {
            accountThread.start();
        }
        // 等带1000个线程执行完毕
        for (Thread accountThread : accountThreads) {
            try {
                accountThread.join();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
        // 记录线程扣减余额结束的时间点
        long endTime = System.nanoTime();
        System.out.println(account.getBalance() + "扣减余额花费：" + (endTime - beginTime) + "纳秒时间。");
    }
}

```

Account当中有两个抽象方法，getBalance获取余额，withdraw扣减余额。

Demo演示方法：创建1000个线程，都执行扣减余额的操作，每个线程扣减10元，总的余额为10000元，理论上最终余额为0。但是这种方式存在线程安全问题。

![](juc并发编程  1.assets/屏幕截图 2026-04-03 160817.png)

结局办法：在扣减余额的临界区添加synchronized上锁。

### 加锁实现

![](juc并发编程  1.assets/屏幕截图 2026-04-03 160927.png)

![](juc并发编程  1.assets/屏幕截图 2026-04-03 161127.png)

### 无锁实现

```java
public class ThreadTestDemo38 {
    public static void main(String[] args) {
        Account account = new AccountCas(10000);
        Account.demo(account);
    }
}

class AccountCas implements Account {
    // 定义一个原子整数，用于存储余额
    private AtomicInteger balance;

    public AccountCas(int balance) {
        this.balance = new AtomicInteger(balance);
    }

    @Override
    public Integer getBalance() {
        return this.balance.get();
    }

    @Override
    public void withdraw(Integer amount) {
        while (true) {
            // 获取当前余额的最新值
            int prev = this.balance.get();
            // 计算需要修改的余额值
            int next = prev - amount;
            // 到主存当中校验余额是否已经被修改，若未修改，修改为需要修改的余额值
            boolean success = balance.compareAndSet(prev, next);
            if (success) {
                break;
            }
        }
    }
}
```

没加任何锁，但是没有发生线程安全问题。

![](juc并发编程  1.assets/屏幕截图 2026-04-03 163157.png)



## 2.CAS工作方式

上面无锁的实方式就是使用了CAS

```java
 public void withdraw(Integer amount) {
        while (true) {
            int prev = this.balance.get();
            int next = prev - amount;
            // 比较值并且设置值
            boolean success = balance.compareAndSet(prev, next);
            if (success) {
                break;
            }
        }
   }
```

这段代码的执行：

首先获得当前的余额值，然后计算扣减之后的余额值，之后将余额值写回。

关键点就在于这个 **compareAndSet** （也叫 CAS），它是一个原子操作。

CAS的工作方式：

1. **读取阶段**：读取共享变量的当前值到工作内存（本地变量）

2. **计算阶段**：基于当前值计算新值

3. 比较交换阶段

   ：原子性地比较主存中的值与之前读取的值

   - **如果相同**：说明在此期间没有被其他线程修改，可以安全更新，并且退出循环
   - **如果不同**：说明在此期间被其他线程修改了，更新失败，进行下一次CAS

<img src="juc并发编程  1.assets/屏幕截图 2026-04-03 164201.png" style="zoom:67%;" />

核心思想：采用不断尝试不断校验，直至成功的思想。



## 3.CAS 与 volatile

AtomicInteger 中用来存储数值的变量被 volatile 修饰。

![](juc并发编程  1.assets/屏幕截图 2026-04-03 170102.png)

CAS必须借助 volatile才能保证读取到的变量的最新值，从而实现 比较和交换。



### CAS的特点

- CAS基于乐观锁的思想，synchronized基于悲观锁的思想。
- CAS 体现的是无锁并发，无阻塞并发。
  - CAS不会让线程陷入阻塞状态，从而能够提升效率。
  - 但是如果线程竞争非常激烈，使用CAS必然会陷入频繁的重试，这反而会影响效率。



## 4.原子整数

- AtomicInteger

- AtomicBoolean

- AtomicLong

### 简单API

```java
// 创建一个原子整数，可以传入参数
AtomicInteger atomicInteger = new AtomicInteger(0);

// 先获取再自增：0，类似于i++
int getIncrement = atomicInteger.getAndIncrement();

// 先自增在获取：2，类似于++i
int incrementGet = atomicInteger.incrementAndGet();

// 先获取再加：4，类似于i+2
int getAdd = atomicInteger.getAndAdd(2);

// 先加再获取：6，类似于2+i
int addGet = atomicInteger.addAndGet(2);
```



### getAndUpadate 和 updateAndGet

如果想实现乘法，除法，或者一些其他复杂运算，可以通过getAndUpadate实现。

getAndUpadate（），需要通过构造函数传入一个IntUnaryOperator，这是一个函数式接口，里面有一个抽象方法，applyAsInt，用来实现对数据的操作方式。

因为是函数式接口，所以可以通过lambada表达式进行简化。

![屏幕截图 2026-04-03 190531](juc并发编程  1.assets/屏幕截图 2026-04-03 190531.png)

getAndUpadate 和 updateAndGet 这两个的区别就是

getAndUpadate 是先获得数据，再操作。

updateAndGet 是先操作数据，在获得操作之后的数据。

```java
// 使用匿名内部类来创建
public class ThreadTestDemo39 {
    public static void main(String[] args) {
        // 创建一个原子整数，可以传入参数
        AtomicInteger atomicInteger = new AtomicInteger(1);

        int andUpdate = atomicInteger.updateAndGet(new IntUnaryOperator() {
            @Override
            public int applyAsInt(int operand) {
                return operand * 2;
            }
        });

        System.out.println(andUpdate);
    }
}

// 通过lambada表达式来简化书写
public class ThreadTestDemo39 {
    public static void main(String[] args) {
        // 创建一个原子整数，可以传入参数
        AtomicInteger atomicInteger1 = new AtomicInteger(1);
        AtomicInteger atomicInteger2 = new AtomicInteger(1);

        int andUpdate1 = atomicInteger1.updateAndGet(operand -> operand * 2); // 2
        int andUpdate2 = atomicInteger2.getAndUpdate(operand -> operand * 2); // 1

        System.out.println(andUpdate1);
        System.out.println(andUpdate2);
    }
}

```



### updateAndGet原理

updateAndGet底层也是使用CAS，先获取到最新值，然后计算新值，最后比较。

比较结果相同，就交换数据。

不同，则交换失败，继续循环检查。

为了通用性，可以将updateAndGet抽取成一个通用方法，并且参数传递一个IntUnaryOperator，调用updateAndGet，传递参数的时候，通过lambada表达式指定计算规则。

```java
public class ThreadTestDemo39 {
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(10);
        System.out.println(updateAndGet(atomicInteger, operand -> operand * 2));

    }

    public static int updateAndGet(AtomicInteger i,IntUnaryOperator operator) {
        while (true) {
            // 获取数据
            int prev = i.get();
            // 计算新值
            int next = operator.applyAsInt(prev);
            // 比较并交换
            boolean success = i.compareAndSet(prev, next);
            if (success) {
                return next;
            }
        }
    }
}
```



## 5.原子引用

### AtomicReference 

AtomicReference（原子引用） 是最基本的原子引用类，它可以保证对一个引用类型的单个读写操作是原子的。

```java
public class ThreadTestDemo41 {
    public static void main(String[] args) {
        BigDecimalAccount bigDecimalAccount = new BigDecimalAccountCas(BigDecimal.valueOf(10000));
        BigDecimalAccount.demo(bigDecimalAccount);
    }
}


class BigDecimalAccountCas implements BigDecimalAccount {
    // 对余额进行原子操作
    private AtomicReference<BigDecimal> balance;

    public BigDecimalAccountCas(BigDecimal balance) {
        this.balance = new AtomicReference<>(balance);
    }

    @Override
    public BigDecimal getBalance() {
        return balance.get();
    }

    @Override
    public void withdraw(BigDecimal amount) {
        while (true) {
            BigDecimal prev = balance.get();
            BigDecimal next = prev.subtract(amount);
            boolean success = balance.compareAndSet(prev, next);
            if (success) {
                break;
            }
        }
    }
}

interface BigDecimalAccount {
    // 获取余额
    BigDecimal getBalance();

    // 扣减余额
    void withdraw(BigDecimal amount);

    // 演示方法
    static void demo(BigDecimalAccount account) {
        // 创建一个集合，用于存储1000个线程你
        List<Thread> accountThreads = new ArrayList<>();
        // 创建1000个线程，并调用扣减余额的方法，并将线程添加到集合中
        for (int i = 0; i < 1000; i++) {
            accountThreads.add(new Thread(() -> {
                // 每个线程扣减10元
                account.withdraw(BigDecimal.TEN);
            }));
        }
        // 记录启动线程开始扣减余额的时间点
        long beginTime = System.nanoTime();
        // 启动集合中的1000个线程
        for (Thread accountThread : accountThreads) {
            accountThread.start();
        }
        // 等带1000个线程执行完毕
        for (Thread accountThread : accountThreads) {
            try {
                accountThread.join();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
        // 记录线程扣减余额结束的时间点
        long endTime = System.nanoTime();
        System.out.println(account.getBalance() + "扣减余额花费：" + (endTime - beginTime) + "纳秒时间。");
    }
}

```

![](juc并发编程  1.assets/屏幕截图 2026-04-03 215017.png)



#### 原子引用 ABA 问题

ABA问题：某个线程读取到的值A，在操作过程中被修改为了B，之后又修改为了A，导致最终CAS认为值A并没有变化过，但其实已经修改过了，只是值又变回了之前的样子。

```java
public class ThreadTestDemo42 {
    static AtomicReference<String> ref = new AtomicReference<>("A");

    public static void main(String[] args) throws InterruptedException {
        log.debug("start 开始");
        // 获取ref的值
        String prev = ref.get();
        // 调用other方法
        other();
        // 睡眠1s，等待other执行完毕
        Thread.sleep(1000);
        // 修改ref的值
        boolean success = ref.compareAndSet(prev, "C");
        log.debug("修改A->C是否成功：{}", success);
    }

    public static void other() {
        new Thread(() -> {
            log.debug("修改A->B是否成功：{}", ref.compareAndSet(ref.get(), "B"));
        }, "thread1").start();
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        new Thread(() -> {
            log.debug("修改B->A是否成功：{}", ref.compareAndSet(ref.get(), "A"));
        }, "thread2").start();
    }
}
```

比如：主线程先获取到A，但是在操作的过程中A先后被其他线程修改为B，之后又修改为A。

最终主线程调用compareAndSet进行比较并交换，但是此时数据值仍为A，CAS就认为A从未被改变过，这些改变对他是未知的。

AtomicReference 无法感知到是否有线程已经改变了数据值。



### AtomicStampedReference 

AtomicStampedReference  原子标记引用：

额外维护了一个**版本号**，添加数据的时候，不仅添加数据，还额外添加一个**版本号**，可以解决ABA问题。

获取数据的时候：获取 **值 + 版本号**。

更新数据的时候，拿着 值和版本号 和最新的数据的值和版本号做比较：

- 如果都相同，就说明在此期间并没有其他线程对共享便变量进行过修改。这个时候进行交换，并且更新版本号把版本号+1，代表已经有线程对共享变量进行过了一次修改操作。
- 如果版本号不同，就代表该线程从获取数据到交换数据的期间，其他线程对该共享变量进行过修改，并且更新过版本号，此时更新就会失败。

![](juc并发编程  1.assets/屏幕截图 2026-04-03 224900.png)

copareAndSet 需要传入四个参数：初始值，修改值，版本号，更新后的版本号。只有值和版本号都相同才交换。

getReference 获取当前用对象的数据。

getStamp：获取版本号。

```java
public class ThreadTestDemo42 {
    // 添加数据，并且指定初始版本号
    static AtomicStampedReference<String> ref = new AtomicStampedReference<>("A", 0);

    public static void main(String[] args) throws InterruptedException {
        log.debug("start 开始");
        // 获取ref引用
        String prev = ref.getReference();
        // 获取版本号
        int stamp = ref.getStamp();
        // 调用other方法
        other();
        // 睡眠1s，等待other执行完毕
        Thread.sleep(1000);
        // 修改ref的值
        boolean success = ref.compareAndSet(prev, "C", stamp, stamp + 1);
        log.debug("修改A->C是否成功：{}", success);

        log.debug("ref的值：{}", ref.getReference());
    }

    public static void other() {
        new Thread(() -> {
            int stamp = ref.getStamp();
            log.debug("版本号：{}", stamp);
            log.debug("修改A->B是否成功：{}", ref.compareAndSet(ref.getReference(), "B", stamp, stamp + 1));
        }, "thread1").start();
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        new Thread(() -> {
            int stamp = ref.getStamp();
            log.debug("版本号：{}", stamp);
            log.debug("修改B->A是否成功：{}", ref.compareAndSet(ref.getReference(), "A", stamp, stamp + 1));
        }, "thread2").start();
    }
}

```

```cmd
2026-04-03 22:58:28.575 [main] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo42 - start 开始
2026-04-03 22:58:28.578 [thread1] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo42 - 版本号：0
2026-04-03 22:58:28.578 [thread1] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo42 - 修改A->B是否成功：true
2026-04-03 22:58:29.088 [thread2] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo42 - 版本号：1
2026-04-03 22:58:29.088 [thread2] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo42 - 修改B->A是否成功：true
2026-04-03 22:58:30.099 [main] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo42 - 修改A->C是否成功：false
2026-04-03 22:58:30.099 [main] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo42 - ref的值：A
```

主线程执行失败，因为主线程最初获得的版本号是0，但是随着两个线程的修改，版本号被更新为了2，校验时版本号与最新的的版本号不一致，所以更新失败。





### **AtomicMarkableReference** 

AtomicStampedReference 通过额外维护了一个版本号，记录共享变量是否被修改过以及修改的次数。

AtomicMarkableReference 只关心共享变量是否被修改过，不关心修改过几次。

AtomicMarkableReference 内维护了一个 布尔标记，初始可以为false，该布尔标记与引用关联了起来。

进行数据交换的时候，会将获得的标记与最新的标记进行比较

- 如果标记相同，就代表在此期间，没有其他线程修改过该共享变量，可以进行数据交换，之后将标记设置为true，代表已经有线程对该变量进行过修改了。
- 如果标记不同，则代表在此期间，有其他线程修改过该共享变量，那么数据交换就会失败。

compareAndSet 传入四个参数：数据值，更新后的数据值，获取的标记位，更新之后的标记位。

```java
public class ThreadTestDemo42 {
    // 添加数据，并且指定初始版本号
    static AtomicMarkableReference<String> ref = new AtomicMarkableReference<>("A", false);

    public static void main(String[] args) throws InterruptedException {
        log.debug("start 开始");
        // 获取ref引用
        String prev = ref.getReference();
        // 获取当前的标记
        boolean marked = ref.isMarked();
        // 调用other方法
        other();
        // 睡眠1s，等待other执行完毕
        Thread.sleep(1000);
        // 修改ref的值
        boolean success = ref.compareAndSet(prev, "C", marked, true);
        log.debug("修改A->C是否成功：{}", success);

        log.debug("ref的值：{}", ref.getReference());
    }

    public static void other() {
        new Thread(() -> {
            log.debug("修改A->B是否成功：{}", ref.compareAndSet(ref.getReference(), "B", ref.isMarked(), true));
        }, "thread1").start();

    }
}

```

```cmd
2026-04-03 23:26:01.029 [main] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo42 - start 开始
2026-04-03 23:26:01.032 [thread1] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo42 - 修改A->B是否成功：true
2026-04-03 23:26:02.036 [main] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo42 - 修改A->C是否成功：false
2026-04-03 23:26:02.037 [main] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo42 - ref的值：B

```



## 6.原子数组

- AtomicIntegerArray：对 int[ ] 数组元素的**原子性更新操作**
- AtomicLongArray：对 Long[] 数组元素的**原子性更新操作**
- AtomicReferenceArray：对 引用类型[] 数组元素 的**原子性更新操作**

可以保护数组内的元素的线程安全。

### 常用API

### AtomicIntegerArray

- legnth（）：返回数组长度
- get（int i）：返回索引位置元素
- set（int i，int newValue）：更新指定位置元素的值
- getAndSet(int i, int newValue)：原子更新，先返回旧值，再更新
- compareAndSet(int i, int expect, int update)：对指定索引位置元素进行CAS操作

普通数组：

```java
@Slf4j
public class ThreadTestDemo43 {
    static int[] array = {10};
    public static void main(String[] args) throws InterruptedException {
        log.debug("main 线程开始操作");
        other(array);
        for (int i = 0; i < 100000; i++) {
            array[0]--;
        }
        Thread.sleep(500);
        log.debug("最终结果: {}", array[0]);
    }

    private static void other(int[] array) {
        log.debug("t1 线程开始操作");
        new Thread(() -> {
            for (int i = 0; i < 100000; i++) {
                // 自增10次
                array[0]++;
            }
        },"t1").start();
    }
}

```

理论上最终应该为0，但是发生线程安全问题:

```java
2026-04-04 15:16:40.260 [main] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo43 - main 线程开始操作
2026-04-04 15:16:40.264 [main] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo43 - t1 线程开始操作
2026-04-04 15:16:40.772 [main] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo43 - 最终结果: -12148
```

使用AtomicIntegerArray

```java
@Slf4j
public class ThreadTestDemo43 {
    static int[] array = {10};
    static AtomicIntegerArray safeArray = new AtomicIntegerArray(array);
    public static void main(String[] args) throws InterruptedException {
        log.debug("main 线程开始操作");
        other(safeArray);
        for (int i = 0; i < 100000; i++) {
            int prev = safeArray.get(0);
            int end = prev + 1;
            safeArray.compareAndSet(0,prev,end);
        }
        Thread.sleep(500);
        log.debug("最终结果: {}", array[0]);
    }

    private static void other(AtomicIntegerArray array) {
        log.debug("t1 线程开始操作");
        new Thread(() -> {
            for (int i = 0; i < 100000; i++) {
                int prev = array.get(0);
                int end = prev + 1;
                safeArray.compareAndSet(0,prev,end);
            }
        },"t1").start();
    }
}

```

```cmd
2026-04-04 15:22:10.550 [main] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo43 - main 线程开始操作
2026-04-04 15:22:10.552 [main] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo43 - t1 线程开始操作
2026-04-04 15:22:11.075 [main] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo43 - 最终结果: 10
```



## 7.原子字段更新器

字段更新器可以保护对象的某个字段，对字段进行原子操作。

- ### AtomicIntegerFieldUpdater 原子更新整型字段

- ### AtomicLongFieldUpdater  原子更新长整型字段

- ### AtomicReferenceFieldUpdater  原子更新引用类型字段

  

```java
public class ThreadTestDemo44 {
    public static void main(String[] args) {
        Student student = new Student();
        AtomicReferenceFieldUpdater updater = AtomicReferenceFieldUpdater.newUpdater(Student.class,String.class,"name");
        boolean success = updater.compareAndSet(student, null, "张三");
        System.out.println(student);
    }
}
class Student {
    volatile String name;

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                '}';
    }
}

```

将名字中途改了，所以最终修改失败。

![](juc并发编程  1.assets/屏幕截图 2026-04-04 154507.png)



## 8.原子累加器

### **LongAdder**

专门用于做累加操作的原子类，相较于AtomicLong 在高并发场景下性能更好。

- LongAdder longAdder = new LongAdder();  创建一个LongAdder的实例，默认值为0

- longAdder.sum();   获取当前的总和

- adder.increment();        // +1
- adder.add(5);            // +5
- adder.decrement();        // -1

```java

public class HighConcurrencyTest {

    static AtomicLong atomicLong = new AtomicLong(0);
    static LongAdder longAdder = new LongAdder();

    public static void main(String[] args) throws InterruptedException {
        testAtomicLong();
        testLongAdder();
    }

    // 线程数
    static final int THREAD_COUNT = 20;
    // 每个线程执行次数
    static final int TIMES = 100_000;

    public static void testAtomicLong() throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(THREAD_COUNT);

        long start = System.nanoTime();

        for (int i = 0; i < THREAD_COUNT; i++) {
            new Thread(() -> {
                for (int j = 0; j < TIMES; j++) {
                    atomicLong.getAndIncrement();
                }
                latch.countDown();
            }).start();
        }

        latch.await(); // 等待所有线程执行完

        long end = System.nanoTime();

        System.out.println("AtomicLong耗时: " + (end - start) / 1_000_000 + " ms");
        System.out.println("结果: " + atomicLong.get());
    }

    public static void testLongAdder() throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(THREAD_COUNT);

        long start = System.nanoTime();

        for (int i = 0; i < THREAD_COUNT; i++) {
            new Thread(() -> {
                for (int j = 0; j < TIMES; j++) {
                    longAdder.increment();
                }
                latch.countDown();
            }).start();
        }

        latch.await();

        long end = System.nanoTime();

        System.out.println("LongAdder耗时: " + (end - start) / 1_000_000 + " ms");
        System.out.println("结果: " + longAdder.sum());
    }

```

```cmd
AtomicLong耗时: 57 ms
结果: 2000000
LongAdder耗时: 45 ms
结果: 2000000
```

AtomicLong在做累加操作的时候，CAS需要使用while循环进行校验，竞争激烈的时候，校验的次数就会变得很多，这是影响性能的关键。



### 性能提升的原因

LongAdder ：

**在进行累加时，内部维护了一个 base 值和一个 Cell 数组。**

![](juc并发编程  1.assets/v2-50655ffc03a5ae2bb499542735e9376e_r.jpg)

- 在低并发时：
  - cas失败的情况还比较低，所以直接通过 CAS 更新 base，类似 AtomicLong
  
  <img src="juc并发编程  1.assets/v2-d9f122540d34373ac8e7b5ad5a539860_r.jpg" style="zoom:67%;" />
  
- 在高并发时：
  - 多个线程竞争 base 非常激烈，cas失败的概率就会变大
  - 之后会创建 Cell 数组
  - 不同线程通过哈希分散到不同 Cell 中进行 CAS 操作
  
  ![](juc并发编程  1.assets/v2-8b487106f8f9b040f9e7f7fbc69efb4d_r.jpg)

在对一个共享变量做累加操作的时候，会给这个变量设置多个累加单元，会把这个变量分散到多个 **cell** 中进行存储，竞争激烈的时候，不同的线程会到不同的cell中进行CAS累加操作，这就避免了多个线程去访问同一个内存地址，最终再将这些cell中的值进行汇总。因为是在不同的cell中进行cas，因为减少了cas失败概率，所以性能上来了。



### CAS锁

不要用于实践

```java
class CasLock {
    // 加锁标识
    private AtomicInteger state = new AtomicInteger(0);

    // 加锁
    public void lock() {
        while (true) {
            if (state.compareAndSet(0,1)) {
                break;
            }
        }
    }

    // 解锁
    public void unlock() {
        state.set(0);
    }
}
```

```java
@Slf4j
public class ThreadTestDemo45 {
    static CasLock casLock = new CasLock();

    public static void main(String[] args) {
        new Thread(() -> {
            log.debug("线程开始操作");
            casLock.lock();
            try {
                log.debug("开始。。。");
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                casLock.unlock();
            }
        },"t1").start();

        new Thread(() -> {
            log.debug("线程开始操作");
            casLock.lock();
            try {
                log.debug("开始。。。");
            } finally {
                casLock.unlock();
            }
        },"t2").start();
    }
}
```

```cmd
2026-04-05 17:22:50.125 [t1] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo45 - 线程开始操作
2026-04-05 17:22:50.125 [t2] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo45 - 线程开始操作
2026-04-05 17:22:50.127 [t1] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo45 - 开始。。。
2026-04-05 17:22:51.128 [t2] DEBUG JUC.JUCTest.Demo.D1.ThreadTestDemo45 - 开始。。。
```



### LongAdder源码 - add

```java
// 累加单元数组，懒惰初始化，一开始无竞争的情况为null，竞争发生的时候尝试创建
transient volatile Cell[] cells;

// 基础值，如果没有竞争，则用 cas 累加这个域
transient volatile long base;

// 在 cells 创建或扩容时，置为 1，表示加锁
transient volatile int cellsBusy;
```

```java
   public void add(long x) {
        Cell[] cs; long b, v; int m; Cell c;
        if ((cs = cells) != null || !casBase(b = base, b + x)) {
            int index = getProbe();
            boolean uncontended = true;
            if (cs == null || (m = cs.length - 1) < 0 ||
                (c = cs[index & m]) == null ||
                !(uncontended = c.cas(v = c.value, v + x)))
                longAccumulate(x, null, uncontended, index);
        }
    }
```

![](juc并发编程  1.assets/屏幕截图 2026-04-05 180010.png)

- 先判断累加单元cells是否为空（无竞争的时候为null）
  - 为空，说明无竞争，对基础变量base，做cas累加操作
    - 累加成功，为true，直接返回
    - 累加失败，为false，进入longAccumulate
  - 不为空，说明有竞争，cells已经创建完成，之后会判断，当前想要执行累加操作的线程是否已经有了一个对应的cell被创建了
    - 未创建，cell为空，进入longAccumulate
    - 创建了，就对当前的cell进行cas累加操作
      - 累加成功，直接return
      - 累加失败，进入longAccumulate





### LongAdder源码 - longAccumulate

cas累加失败、cells、cell未创建都会进入longAccumulate

#### cells还未创建的时候：

![](juc并发编程  1.assets/屏幕截图 2026-04-05 181917.png)

cells不存在，并且没有锁（还无线程要创建cells），并且未空

- 尝试cas加锁创建cells（将状态位改为1，参考上面写的cas锁）
  - 加锁成功，将标志位设置为1
    - 创建一个cells数组，初始大小为2
    - 之后创建累加单元cell，并且把base的值作为初始值赋值给cell，并将累加单元添加到cells数组当中
    - 把创建好的cells数组赋值给成员变量cells
    - 之后把标志位设置为0，代表解锁
    - 退出循环
  - 加锁失败，就对base做cas累加
    - 累加成功，直接return
    - 累加失败，重新循环



#### cells创建了，但是cell还未创建：

![](juc并发编程  1.assets/屏幕截图 2026-04-05 184024.png)

cells数组创建好了，但是新的线程进来，对应的cell还未创建。

cells已经创建，但是cell还未创建

- 创建一个新的cell累加单元，并把base的值赋值给cell，尝试加锁将cell添加到cells中
  - 上锁成功（将标志位设置为1）
    - 判断该槽位是否为空（再次检查cells不为空，线程对应的cell为空）
      - 为空，将cell添加到cells中，添加成功，退出循环
      - 不为空，说明槽位已经被其他线程占有，循环chong'shi
  - 上锁失败，循环重试



#### cells、cell都创建了

![](juc并发编程  1.assets/屏幕截图 2026-04-05 205013.png)

cells、cell都已经创建了

尝试对cell累加单元进行cas累加操作

- 累加成功，直接结束
- 累加失败，查看是否超过了CPU上限（cells数组长度是否大于CPU核心数）
  - 超过了，就给当前线程换一个cell累加单元进行累加操作
  - 没超过，触发加锁扩容
    - 加锁成功，进行扩容，将cells数组变为原来的2倍（底层是对数组长度左移一位），之后进行数据迁移，循环将元素拷贝到新数组，并且将新数组赋值给变量cells，释放锁。重新进行循环
    - 加锁失败，也是给当前线程换一个cell累加单元进行累加操作。



### 

# 十七。共享模型之不可变

SimpleDateFormat 可变类的问题

```java
@Slf4j
public class ThreadTestDemo1 {
    public static void main(String[] args) {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                try {
                    log.debug("{}",simpleDateFormat.parse("2023-01-01"));
                } catch (ParseException e) {
                    throw new RuntimeException(e);
                }
            }).start();
        }
    }
}
```

10个线程去解析这个日期，结果出现异常

![](juc并发编程  1.assets/屏幕截图 2026-04-05 224034.png)



## 1.不可变类

DateTimeFormatter 是一个不可变类，是线程安全的。

```java
@Slf4j
public class ThreadTestDemo1 {
    public static void main(String[] args) {
        DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                TemporalAccessor parse = dateTimeFormatter.parse("2023-01-01");
                log.debug("{}", parse);
            }).start();
        }
    }
}
```

```cmd
2026-04-05 22:51:01.081 [Thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo1 - {},ISO resolved to 2023-01-01
2026-04-05 22:51:01.081 [Thread-9] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo1 - {},ISO resolved to 2023-01-01
2026-04-05 22:51:01.081 [Thread-3] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo1 - {},ISO resolved to 2023-01-01
2026-04-05 22:51:01.081 [Thread-0] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo1 - {},ISO resolved to 2023-01-01
2026-04-05 22:51:01.081 [Thread-4] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo1 - {},ISO resolved to 2023-01-01
2026-04-05 22:51:01.081 [Thread-6] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo1 - {},ISO resolved to 2023-01-01
2026-04-05 22:51:01.081 [Thread-5] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo1 - {},ISO resolved to 2023-01-01
2026-04-05 22:51:01.081 [Thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo1 - {},ISO resolved to 2023-01-01
2026-04-05 22:51:01.081 [Thread-7] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo1 - {},ISO resolved to 2023-01-01
2026-04-05 22:51:01.081 [Thread-8] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo1 - {},ISO resolved to 2023-01-01
```



## 2.String 不可变

```java
public final class String {
        private final byte[] value;
}
```

- String类被**final**修饰，无法被继承，防止了子类去重写覆盖String类内部的行为。

- 字节数组被**final**修饰，保证了只能读不能修改。

  

## 3.保护性拷贝

String中的substring()方法可以对字符串进行截取修改操作，实际上并不是对原String进行了修改，因为String是不可变的，根本修改不了。

substring实际上创建了一个新的String对象，并且把原String对象中的字节数组拷贝赋值给这个新的String对象中的字节数组 。

这种通过**创建副本对象来避免共享**的手段就是**保护性拷贝**。



## 4.享元模式

适用于对象需要重用的时候。

如果大量的对象非常相似甚至相同，就没必要重复创建，可以只创建一个，需要的时候就共享使用，从而减少了内存的占用。

比如：包装类：Long、Integer、Boolean中的valueOf()方法，Long.valueOf（）会将-128~127之间的Long对象提前缓存起来，当需要创建这个范围的对象的时候，就直接拿来用，而无需重复创建。当超过这个范围的时候再去创建新的对象。

```java
    @IntrinsicCandidate
    public static Long valueOf(long l) {
        final int offset = 128;
        if (l >= -128 && l <= 127) { // will cache
            return LongCache.cache[(int)l + offset];
        }
        return new Long(l);
    }
```



## 5.自定义连接池	

使用享元模式+多线程自定义数据库连接池：

```java
public class ThreadTestDemo2 {
    public static void main(String[] args) {
        Pool pool = new Pool(2);
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                Connection connection = pool.borrow();
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                pool.release(connection);
            }, "线程" + i).start();
        }
    }
}

@Slf4j
class Pool {
    // 连接池大小
    private final int poolSize;

    // 连接对象数组
    private Connection[] connections;

    // 连接状态数组：0-标识空闲，1-标识繁忙
    private AtomicIntegerArray states;

    // 繁忙
    private final int RUNNING = 1;
    // 空闲
    private final int RELAXED = 0;

    // 构造方法初始化
    public Pool(int poolSize) {
        this.poolSize = poolSize;
        this.connections = new Connection[poolSize];
        this.states = new AtomicIntegerArray(new int[poolSize]);
        for (int i = 0; i < poolSize; i++) {
            connections[i] = new MockConnection("连接" + i);
        }
    }

    // 借连接
    public Connection borrow() {
        while (true) {
            for (int i = 0; i < poolSize; i++) {
                int prev = states.get(i);
                if (prev == this.RELAXED) {
                    boolean success = states.compareAndSet(i, prev, RUNNING);
                    if (success) {
                        log.debug("借出连接:{}", connections[i]);
                        return connections[i];
                    }
                }
            }
            // 没有空闲连接
            synchronized (this) {
                try {
                    log.debug("没有空闲连接，等待...");
                    this.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }

    // 归还连接
    public void release(Connection conn) {
        for (int i = 0; i < poolSize; i++) {
            if (connections[i] == conn) {
                states.set(i, RELAXED);
                synchronized (this) {
                    log.debug("归还链接:{}", conn);
                    this.notifyAll();
                }
                break;
            }
        }
    }
}

class MockConnection implements Connection {
    /*...*/
}
```

```cmd
2026-04-06 15:53:01.945 [线程2] DEBUG JUC.JUCTest.Demo.D2.Pool - 没有空闲连接，等待...
2026-04-06 15:53:01.947 [线程1] DEBUG JUC.JUCTest.Demo.D2.Pool - 没有空闲连接，等待...
2026-04-06 15:53:01.947 [线程4] DEBUG JUC.JUCTest.Demo.D2.Pool - 没有空闲连接，等待...
2026-04-06 15:53:01.945 [线程0] DEBUG JUC.JUCTest.Demo.D2.Pool - 借出连接:MockConnection{name='连接0'}
2026-04-06 15:53:01.945 [线程3] DEBUG JUC.JUCTest.Demo.D2.Pool - 借出连接:MockConnection{name='连接1'}
2026-04-06 15:53:02.962 [线程0] DEBUG JUC.JUCTest.Demo.D2.Pool - 归还链接:MockConnection{name='连接0'}
2026-04-06 15:53:02.962 [线程2] DEBUG JUC.JUCTest.Demo.D2.Pool - 借出连接:MockConnection{name='连接0'}
2026-04-06 15:53:02.962 [线程1] DEBUG JUC.JUCTest.Demo.D2.Pool - 没有空闲连接，等待...
2026-04-06 15:53:02.962 [线程4] DEBUG JUC.JUCTest.Demo.D2.Pool - 借出连接:MockConnection{name='连接1'}
2026-04-06 15:53:02.962 [线程3] DEBUG JUC.JUCTest.Demo.D2.Pool - 归还链接:MockConnection{name='连接1'}
2026-04-06 15:53:02.962 [线程1] DEBUG JUC.JUCTest.Demo.D2.Pool - 没有空闲连接，等待...
2026-04-06 15:53:03.962 [线程2] DEBUG JUC.JUCTest.Demo.D2.Pool - 归还链接:MockConnection{name='连接0'}
2026-04-06 15:53:03.963 [线程1] DEBUG JUC.JUCTest.Demo.D2.Pool - 借出连接:MockConnection{name='连接0'}
2026-04-06 15:53:03.970 [线程4] DEBUG JUC.JUCTest.Demo.D2.Pool - 归还链接:MockConnection{name='连接1'}
2026-04-06 15:53:04.974 [线程1] DEBUG JUC.JUCTest.Demo.D2.Pool - 归还链接:MockConnection{name='连接0'}
```



## 6.final原理

### 设置变量

```java
final int a = 10;
```

final修饰的变量赋值之后就是常量了，无法对他的值在进行修改。

字节码层面：final通过**putfield**指令进行赋值操作：

进行putfield的时候会在putfield指令之后加入一个**写屏障**：保证了putfield之前的指令不会被重排序到putfield之后；并且保证了写屏障之前的赋值操作在写屏障之后被同步到主存当中，保证了其他线程对该数据的可见性，保证了线程读它的时候不会出现0的情况。



# 十八。并发工具



## 1.线程池

每创建一个线程，就要占用一块内存，如果创建大量的线程实际上，内存占用就很大。而且线程频繁的发生上下文切换是非常消耗性能的。

线程池提前创建好一批线程，可以控制线程的数量，而且拿来即用，用完再放回去，避免了频繁创建线程与销毁线程带来的性能开销，而且也避免了大量的线程发生上下文切换带来的性能开销。

## 2.自定义线程池

<img src="juc并发编程  1.assets/屏幕截图 2026-04-06 164750.png" style="zoom:67%;" />



<img src="juc并发编程 .assets/屏幕截图 2026-04-07 162408.png" style="zoom:67%;" />

### 阻塞对列实现

```java
// 阻塞对列
class BlockingQueue<T> {
    // 任务队列
    private Deque<T> queue = new ArrayDeque<>();

    // 锁
    private ReentrantLock lock = new ReentrantLock();

    // 生产者条件变量，队列满时，进入等待
    private Condition fullWaitSet = lock.newCondition();

    // 消费者条件变量，对列空时，进入等待
    private Condition emptyWaitSet = lock.newCondition();

    // 容量
    private int capacity;

    // 阻塞获取
    public T take() {
        // 加锁获取
        lock.lock();
        try {
            // 队列为空，进入等待
            while (queue.isEmpty()) {
                try {
                    emptyWaitSet.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            // 获取对列头部元素
            T t = queue.removeFirst();
            // 唤醒等待线程
            fullWaitSet.signalAll();
            return t;
        } finally {
            lock.unlock();
        }
    }

    // 带超时的阻塞获取
    public T poll(long timeOut, TimeUnit timeUnit) {
        // 加锁获取
        lock.lock();
        try {
            // 同一时间单位：统一为纳秒
            long nanos = timeUnit.toNanos(timeOut);
            // 队列为空，进入等待
            while (queue.isEmpty()) {
                try {
                    if (nanos <=0) {
                        // 没等到，返回null
                        return null;
                    }
                    // 返回的是剩余时间；例如等待时间为2s，结果等了1s就被虚假唤醒了，回来接着等待1s即可，而不是再等待2s
                    nanos = emptyWaitSet.awaitNanos(nanos);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            // 获取对列头部元素
            T t = queue.removeFirst();
            // 唤醒等待线程
            fullWaitSet.signalAll();
            return t;
        } finally {
            lock.unlock();
        }
    }

    // 阻塞添加
    public void put(T element) {
        // 加锁添加
        lock.lock();
        try {
            // 对列已满，进入等待
            while (queue.size() == capacity) {
                try {
                    fullWaitSet.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            // 添加元素
            queue.addLast(element);
            // 唤醒等待线程发
            emptyWaitSet.signalAll();
        } finally {
            lock.unlock();
        }
    }

    // 获取对列大小
    public int size() {
        lock.lock();
        try {
            return queue.size();
        } finally {
            lock.unlock();
        }
    }
}
```

### 线程池实现

```java
@Slf4j
public class ThreadTestDemo3 {
    public static void main(String[] args) {
        ThreadPool threadPool = new ThreadPool(2, 1000, TimeUnit.MILLISECONDS, 10);
        for (int i = 0; i < 5; i++) {
            int i1 = i;
            threadPool.execute(() -> {
                log.debug("{}",i1);
            });
        }
    }
}

// 线程池
@Slf4j
class ThreadPool {
    // 任务对列
    private BlockingQueue<Runnable> taskQueue;

    // 线程集合
    private HashSet<Worker> workers = new HashSet();

    // 核心线程数
    private int corePoolSize;

    // 超时时间
    private long timeOut;

    // 时间单位
    private TimeUnit timeUnit;

    public ThreadPool(int corePoolSize, long timeOut, TimeUnit timeUnit, int queueCapacity) {
        this.corePoolSize = corePoolSize;
        this.timeOut = timeOut;
        this.timeUnit = timeUnit;
        this.taskQueue = new BlockingQueue<>(queueCapacity);
    }

    // 执行任务
    public void execute(Runnable task) {
        synchronized (workers) {
            // 判断任务数是否大于核心线程数
            if (workers.size() < corePoolSize) {
                // // 如果小于，加入线程集合
                Worker worker = new Worker(task);
                log.debug("新增worker：{}, {}", worker, task);
                workers.add(worker);
                worker.start();
            } else {
                // 如果大于，加入阻塞队列
                log.debug("加入任务对列：{}", task);
                taskQueue.put(task);
            }
        }
    }

    class Worker extends Thread {
        private Runnable task;

        public Worker(Runnable task) {
            this.task = task;
        }

        @Override
        public void run() {
            // 执行任务
            // 当task不为空，直接执行任务
            // 当task执行完毕，从任务对列获取任务，并执行
            while (task != null || (task = taskQueue.poll(timeOut,timeUnit)) != null) {
                try {
                    log.debug("正在执行...{}", task);
                    task.run();
                } catch (Exception e) {
                    throw new RuntimeException(e);
                } finally {
                    task = null;
                }
            }
            synchronized (workers) {
                log.debug("worker 被移除：{}", this);
                workers.remove(this);
            }
        }
    }
}
```

```cmd
2026-04-06 23:27:37.467 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadPool - 新增worker：Thread[#22,Thread-0,5,main], JUC.JUCTest.Demo.D2.ThreadTestDemo3$$Lambda/0x000001f281008c90@32a1bec0
2026-04-06 23:27:37.470 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadPool - 新增worker：Thread[#23,Thread-1,5,main], JUC.JUCTest.Demo.D2.ThreadTestDemo3$$Lambda/0x000001f281008c90@51521cc1
2026-04-06 23:27:37.470 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadPool - 加入任务对列：JUC.JUCTest.Demo.D2.ThreadTestDemo3$$Lambda/0x000001f281008c90@deb6432
2026-04-06 23:27:37.470 [Thread-0] DEBUG JUC.JUCTest.Demo.D2.ThreadPool - 正在执行...JUC.JUCTest.Demo.D2.ThreadTestDemo3$$Lambda/0x000001f281008c90@32a1bec0
2026-04-06 23:27:37.470 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadPool - 加入任务对列：JUC.JUCTest.Demo.D2.ThreadTestDemo3$$Lambda/0x000001f281008c90@28ba21f3
2026-04-06 23:27:37.471 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadPool - 加入任务对列：JUC.JUCTest.Demo.D2.ThreadTestDemo3$$Lambda/0x000001f281008c90@694f9431
2026-04-06 23:27:37.470 [Thread-0] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo3 - 0
2026-04-06 23:27:37.471 [Thread-0] DEBUG JUC.JUCTest.Demo.D2.ThreadPool - 正在执行...JUC.JUCTest.Demo.D2.ThreadTestDemo3$$Lambda/0x000001f281008c90@deb6432
2026-04-06 23:27:37.471 [Thread-0] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo3 - 2
2026-04-06 23:27:37.471 [Thread-0] DEBUG JUC.JUCTest.Demo.D2.ThreadPool - 正在执行...JUC.JUCTest.Demo.D2.ThreadTestDemo3$$Lambda/0x000001f281008c90@28ba21f3
2026-04-06 23:27:37.471 [Thread-0] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo3 - 3
2026-04-06 23:27:37.471 [Thread-0] DEBUG JUC.JUCTest.Demo.D2.ThreadPool - 正在执行...JUC.JUCTest.Demo.D2.ThreadTestDemo3$$Lambda/0x000001f281008c90@694f9431
2026-04-06 23:27:37.471 [Thread-0] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo3 - 4
2026-04-06 23:27:37.473 [Thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadPool - 正在执行...JUC.JUCTest.Demo.D2.ThreadTestDemo3$$Lambda/0x000001f281008c90@51521cc1
2026-04-06 23:27:37.473 [Thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo3 - 1
2026-04-06 23:27:38.478 [Thread-0] DEBUG JUC.JUCTest.Demo.D2.ThreadPool - worker 被移除：Thread[#22,Thread-0,5,main]
2026-04-06 23:27:38.478 [Thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadPool - worker 被移除：Thread[#23,Thread-1,5,main]
```

如果将核心线程数设置为2，阻塞队列设置为10，这时候创建15个线程去提交任务，而每个拿到线程的任务又迟迟执行不完的话，那必然还有3个线程既拿不到线程也进入不到阻塞队列当中，此时则会陷入等待。

为了避免线程一直处于等待的状态，可以写一个带超时的添加任务的方法。如果此时核心线程数有剩余，线程直接拿走执行任务即可。如果核心线程数已经满了，但是阻塞对列还没满，那就到阻塞队列当中，进入等待。但是如果，核心线程数和阻塞队列都满了，都拿不到了，那就给他设置一个时效，时效之内进入条件变量当中等待，如果阻塞队列在时效之内有空余位置了，就让线程进入到阻塞队列当中，返回true；但是如果时效之内，阻塞队列仍然是满的，线程就不再等待了，就直接返回false。

带超时的添加任务的方法

### offer增强

```java
	// 带超时的阻塞添加
    public boolean offer(T task, long timeOut, TimeUnit timeUnit) {
        // 加锁获取
        lock.lock();
        try {
            // 统一时间单位：统一为纳秒
            long nanos = timeUnit.toNanos(timeOut);
            // 对列满了，进入等待
            while (queue.size() == capacity) {
                if (nanos <= 0) {
                    return false;
                }
                try {
                    // 返回剩余阻塞时间（避免虚假唤醒导致下次等待时间又为 nanos）
                    log.debug("等待加入任务对列...：{}", task);
                    nanos = fullWaitSet.awaitNanos(nanos);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            // 添加元素
            log.debug("加入任务对列：{}", task);
            queue.addLast(task);
            // 唤醒等待线程发
            emptyWaitSet.signalAll();
            return true;
        } finally {
            lock.unlock();
        }
    }
```



### 拒绝策略

如果核心线程数+阻塞队列都满了，可以使用拒绝策略。就是让线程自己决定去还是留，留的话又干些什么。

首先定义拒绝策略接口：

```java
@FunctionalInterface
interface RejectPolicy<T> {
    void reject(BlockingQueue<T> queue, T task);
}
```

可以通过ThreadPool的构造方法来来指定阻塞队列，以及拒绝策略，并对其赋值

```java
    public ThreadPool(int corePoolSize, long timeOut, TimeUnit timeUnit,
                      int queueCapacity, RejectPolicy<Runnable> rejectPolicy) {
        this.corePoolSize = corePoolSize;
        this.timeOut = timeOut;
        this.timeUnit = timeUnit;
        this.taskQueue = new BlockingQueue<>(queueCapacity);
        this.rejectPolicy = rejectPolicy;
    }
```

执行任务的时候调用execute方法

```java
// 执行任务
    public void execute(Runnable task) {
        synchronized (workers) {
            // 判断任务数是否大于核心线程数
            if (workers.size() < corePoolSize) {
                // // 如果小于，加入线程集合
                Worker worker = new Worker(task);
                log.debug("新增worker：{}, {}", worker, task);
                workers.add(worker);
                worker.start();
            } else {
                taskQueue.tryPut(rejectPolicy, task);
            }
        }
    }
```

方法内调用tryPut方法：

```java
public void tryPut(RejectPolicy<T> rejectPolicy, T task) {
        // 获取锁
        lock.lock();
        try {
            // 判断对列是否已满
            if (queue.size() == capacity) {
                rejectPolicy.reject(this, task);
            } else {
                log.debug("加入任务对列：{}", task);
                queue.addLast(task);
                // 唤醒等待线程发
                emptyWaitSet.signalAll();
            }
        } finally {
            lock.unlock();
        }
    }
```

通过构造方法指定拒绝策略：

![](juc并发编程  1.assets/屏幕截图 2026-04-07 161953.png)





## 3.ThreadPoolExecutor

![](juc并发编程 .assets/屏幕截图 2026-04-07 163136.png)



### 线程池状态

ThreadPoolExecutor：使用整数**int的高三位**来表示线程池状态，使用**低29位来表示线程数量**。

- **RUNNING**:

  - 高三位：111 

  - 处于正常运行中的线程池，线程**既可以接收新的任务**，也可以**到阻塞队列当拿任务去执行**

- **SHUTDOWN**:

  - 高三位：000
  - 这是一种**温和的停止方式**，线程**不再接收新的任务了**，但是仍**然会处理阻塞队列当中的未执行的任务**

- **STOP**:

  - 高三位：001
  - 这是一种暴力的停止方式，他**不仅不会接收新的任务**，**还会终止线程正在执行的任务**，并且**会把阻塞队列中的所有任务抛弃掉不执行**。

- **TIDYING**:

  - 高三位：010
  - 这是一种过渡状态，**他会等到任务都执行完毕，活动的线程也为0了**，即将进入终结状态

- **TERMNATED**:

  - 高三位：011

  - 终结状态，线程池彻底不工作了。

    

- ### 为什么线程池状态和线程数量需要使用一个整数来存储，而不是使用两个整数来分别存储不是更好吗？

为了保证线程状态和线程数量更新时候的**原子性**，会将这两个信息存储到一个原子变量ctl当中，目的就是为了将线程状态和线程数合二为一，这样更新的时候，只需要进行一次CAS操作就可以了。



### 构造方法

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```

- corePoolSize 核心线程数目 (最多保留的线程数)
- maximumPoolSize 最大线程数目
- keepAliveTime 生存时间 - 针对救急线程
- unit 时间单位 - 针对救急线程
- workQueue 阻塞队列
- threadFactory 线程工厂 - 可以为线程创建时起个好名字
- handler 拒绝策略

<img src="juc并发编程 .assets/屏幕截图 2026-04-07 165957.png" style="zoom: 67%;" />

- 刚开始的时候，核心线程处于空闲状态，有新的任务提交，就先拿核心线程，去执行任务

<img src="juc并发编程 .assets/屏幕截图 2026-04-07 170149.png" style="zoom: 80%;" />

- 如果核心线程都处于忙状态，新来的任务拿不到核心线程了，就进入阻塞队列当中进行等待。

<img src="juc并发编程 .assets/屏幕截图 2026-04-07 172735.png" style="zoom:67%;" />

如果任务量太大，导致**核心线程**，**阻塞队列都满了**，这时就会创建救急线程进行救急，新来的任务就会拿着新创建的救急线程去执行任务。

正因为是救急的吗，所以执行完任务之后，如果没有新的任务提交，这个救急线程就会在一段时间之后被销毁。当下一次高峰期到来的时候，才会又会被考虑重新创建。与之相比，核心线程会一直保留在线程池当中，核心线程没有存活时间，随着线程池的存活而存活，但是救急线程有存活时间。

- keepAliveTime 生存时间 - 针对救急线程
- unit 时间单位 - 针对救急线程

通过keepAliveTime + unit 可以控制救急线程的存活时间，当没有新的任务提交的时候，救急线程会存活一段时间之后，被销毁。

- 如果任务量太大，导致核心线程，阻塞对列，还有救急线程都满了，这个时候会执行拒绝策略。

<img src="juc并发编程 .assets/屏幕截图 2026-04-07 173650.png" style="zoom:67%;" />

#### 注意

- **救急线程**要配合**有界的阻塞队列**来实现，只有说阻塞对列满了，才会临时创建maximumPoolSize - corePoolSize 的救急线程。如果是无界的阻塞队列，就会**无上限**的让任务进入阻塞队列当中等待线程去执行让他。



### 拒绝策略

JDK提供的拒绝策略：

- **AbortPolicy** 让调用者抛出 RejectedExecutionException 异常，这是默认策略
- **CallerRunsPolicy** 让调用者运行任务
- **DiscardPolicy** 放弃本次任务
- **DiscardOldestPolicy** 放弃队列中最早的任务，本任务取而代之

![](juc并发编程 .assets/屏幕截图 2026-04-07 174704.png)

第三方框架:

- Dubbo 的实现，在抛出 RejectedExecutionException 异常之前会记录日志，并 dump 线程栈信息，方便定位问题

- Netty 的实现，是创建一个新线程来执行任务

- ActiveMQ 的实现，带超时等待（60s）尝试放入队列，类似我们之前自定义的拒绝策略

- PinPoint 的实现，它使用了一个拒绝策略链，会逐一尝试策略链中每种拒绝策略





### newFixedThreadPool

固定大小线程池

newFixedThreadPool 是Executors中的一个静态方法，可以构造一个线程池

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
     return new ThreadPoolExecutor(nThreads, nThreads,
                                   0L, TimeUnit.MILLISECONDS,
                                   new LinkedBlockingQueue<Runnable>());
}
```

特点：

- 可以看到ThreadPoolExecutor的构造函数的位置，corePoolSize 和 maximumPoolSize 传进去的参数都是nThreads（线程数），这就意味着 救急线程 = maximumPoolSize  - corePoolSize = 0 ，因此压根不会创建救急线程。
- 而且创建的**阻塞队列没有指定大小**，是**无界的阻塞队列**，可以放置任意数量的任务，所以根本不需要创建救急线程。
- 适用于：任务**数量一定的**，而且任务执行比较**耗时**的任务

```java
@Slf4j
public class ThreadTestDemo4 {
    public static void main(String[] args) {
        // 创建线程池，核心线程数为2
        ExecutorService pool = Executors.newFixedThreadPool(2);

        // 执行任务
        pool.execute(() -> {
            log.debug("1");
        });

        pool.execute(() -> {
            log.debug("2");
        });

        pool.execute(() -> {
            log.debug("3");
        });
    }
}
```

核心线程数为2，所以先执行了1 2，随后执行3

```cmd
2026-04-07 21:49:53.107 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo4 - 2
2026-04-07 21:49:53.107 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo4 - 1
2026-04-07 21:49:53.109 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo4 - 3
```

线程池里的线程都是非守护线程，即使主线程已经结束了，线程池当中的线程也不会结束

![](juc并发编程 .assets/屏幕截图 2026-04-07 215217.png)

线程池，线程默认已经有了名字，是因为在构建线程池的时候，内部会创建默认的线程工厂。

并且给线程池和线程起好名字。

<img src="juc并发编程 .assets/屏幕截图 2026-04-07 215449.png" style="zoom:67%;" />



也可以自定义线程工厂，newFixedThreadPool的第二参数可以传递一个线程工厂，是一个接口，有一个newThread抽象方法，可以通过匿名内部类来实现

```java
@Slf4j
public class ThreadTestDemo4 {
    public static void main(String[] args) {
        // 创建线程池，核心线程数为2
        ExecutorService pool = Executors.newFixedThreadPool(2, new ThreadFactory() {
            AtomicInteger atomicInteger = new AtomicInteger(1);
            @Override
            public Thread newThread(@NotNull Runnable r) {
                return new Thread(r,"zzb-Thread-" + atomicInteger.getAndIncrement());
            }
        });

        // 执行任务
        pool.execute(() -> {
            log.debug("1");
        });

        pool.execute(() -> {
            log.debug("2");
        });

        pool.execute(() -> {
            log.debug("3");
        });
    }
}
```

```cmd
2026-04-07 22:04:48.829 [zzb-Thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo4 - 1
2026-04-07 22:04:48.829 [zzb-Thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo4 - 2
2026-04-07 22:04:48.832 [zzb-Thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo4 - 3
```



### newCachedThreadPool

带缓冲的线程池

也是Executors中的一个静态方法，可以创建一个线程池。

```java
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

特点：

- 核心线程数为0，最大线程数为Integer.MAX_VALUE，救急线程的存活时间为60s，也就是说
  - 创建的线程全都是**救急线程**，而且没任务提交的时候存活时间是60s
  - 救急线程可以无限创建
- 阻塞队列采用的是 SynchronousQueue，特点是：他没有容量，但是线程要是想把任务放进这个对列就有点特殊了，如果放的时候，发现没有线程来到对列中取任务，那添加任务的这个线程就会阻塞住，是放不进去的。
- 这就很适合newCachedThreadPool，因为只要有任务了才去创建救急线程
- 适合任务数量比较大，但是执行时间比较短的任务。



### newSingleThreadExecutor

单线程线程池

```java
public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
    return new AutoShutdownDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>(),
                                threadFactory));
}
```

特点：

- 核心线程数和最大线程数都为1，这就意味着线程池当中只有一个线程。当任务数量大于1的时候，会进入无界队列当中排队等待，执行完毕任务之后，这个唯一的线程也不会释放掉。
- 适用于有多个任务，但是想要这多个任务排队执行，也就是串行执行。

**区别：**

- 自己手动创建一个线程，也能执行任务，但是如果任务执行失败了，那这个线程可就结束了，接下来如果还有别的任务可就执行不下去了，没有任何的补救措施。但是线程池他还会重新创建一个新的线程，从而保证正常工作。
- newFixedThreadPool （1）也会只创建一个线程，但是后期可以修改线程数量。但是newSingleThreadExecutor 的线程个数始终为1，不能对其进行修改。
  - newFixedThreadPool 对外返回的是 ThreadPoolExecutor 对象
  - 而newSingleThreadExecutor 的返回应用了装饰器模式，他不直接返回ThreadPoolExecutor 对象，而是在后返回的时候对 ThreadPoolExecutor  进行了一层封装，使用 AutoShutdownDelegatedExecutorService，这样对外只会暴露 ExecutorService 中的接口，而无法调用 ThreadPoolExecutor  中的方法，这样就防止了修改外界修改 ThreadPoolExecutor  的核心线程数和最大线程数这些参数。

```java
@Slf4j
public class ThreadTestDemo5 {
    public static void main(String[] args) {
        method();
    }

    private static void method() {
        ExecutorService pool = Executors.newSingleThreadExecutor();
        pool.execute(() -> {
            log.debug("1");
            // 此时出现异常
            int a = 1 / 0;
            log.debug("看看我执行了吗😎");
        });

        pool.execute(() -> {
            log.debug("2");
        });

        pool.execute(() -> {
            log.debug("3");
        });
    }
}
```

![](juc并发编程 .assets/屏幕截图 2026-04-07 224044.png)

可以看到，虽然出现了异常，线程1结束了，但是线程池又重新创建了一个线程2，去执行剩下的任务。



### 提交任务

#### 1.执行单个任务

```java
// 执行任务
void execute(Runnable command);

// 与 execute 不同的是，submit可以接收一个带返回值的任务 Callable，这个返回值会被 Future 所获得，用于返回
<T> Future<T> submit(Callable<T> task)
```

```java
@Slf4j
public class ThreadTestDemo6 {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        // 创建一个固定大小的线程池
        ExecutorService pool = Executors.newFixedThreadPool(1);
        // 提交任务
        pool.execute(() -> {
            log.debug("第一个任务");
        });

        Thread.sleep(1000);

        // 提交任务，并获得返回值
        Future<String> future = pool.submit(() -> {
            log.debug("第二个任务");
            // 返回值
            return "返回第二个任务";
        });

        // 获得返回值
        String returnValur = future.get();
        log.debug("返回值：{}", returnValur);
    }
}
```

```cmd
2026-04-08 15:28:57.393 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo6 - 第一个任务
2026-04-08 15:28:58.395 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo6 - 第二个任务
2026-04-08 15:28:58.395 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo6 - 返回值：返回第二个任务
```



#### 2.执行任务集合

```java
// 接收参数是一个任务集合，会去执行任务集合中的所有的方法，返回值是一个 Future 集合
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
      throws InterruptedException;

// 带超时的，如果在规定时间内任务没有执行完，后续的任务就不执行了，返回值也是一个 Future 集合
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                              long timeout, TimeUnit unit)
      throws InterruptedException;
```

```java
@Slf4j
public class ThreadTestDemo7 {
    public static void main(String[] args) throws InterruptedException {
        // 任务集合
        List<Callable<String>> taskList = new ArrayList<>();
        // 添加任务
        for (int i = 0; i < 3; i++) {
            int number = i;
            taskList.add(() -> {
                log.debug("{}", number);
                Thread.sleep(1000L);
                return String.valueOf(number);
            });
        }
        // 创建固定大小线程池
        ExecutorService pool = Executors.newFixedThreadPool(1);
        // 执行任务集合
        List<Future<String>> futures = pool.invokeAll(taskList);
        // 获取任务返回结果
        futures.forEach(future -> {
            try {
                log.debug("{}", future.get());
            } catch (InterruptedException | ExecutionException e) {
                throw new RuntimeException(e);
            }
        });
    }
}
```

```cmd
2026-04-08 15:50:51.920 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo7 - 0
2026-04-08 15:50:52.931 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo7 - 1
2026-04-08 15:50:53.947 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo7 - 2
2026-04-08 15:50:54.964 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo7 - 0
2026-04-08 15:50:54.964 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo7 - 1
2026-04-08 15:50:54.964 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo7 - 2
```



#### 3.只执行一个任务，其他取消执行

```java
// 接收一个任务集合，但是并不会全部执行，他会找到第一个执行完成的任务，并把结果返回。其他未执行完的任务，一律取消执行
<T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;
// 带超时的        
<T> T invokeAny(Collection<? extends Callable<T>> tasks,
                long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
```

```java
@Slf4j
public class ThreadTestDemo7 {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        // 任务集合
        List<Callable<String>> taskList = new ArrayList<>();
        // 添加任务，任务的休眠时间逐渐增大，则任务执行时间逐渐变大
        for (int i = 1; i <= 3; i++) {
            int number = i;
            taskList.add(() -> {
                log.debug("begin : {}", number);
                Thread.sleep(number * 1000);
                log.debug("end : {}", number);
                return String.valueOf(number);
            });
        }
        // 创建固定大小线程池，返回值是第一个完成的任务的返回值
        ExecutorService pool = Executors.newFixedThreadPool(3);
        // 执行任务集合，任务一最先执行完毕，直接返回了任务一的结果
        String only = pool.invokeAny(taskList);
        log.debug("唯一被执行的任务：{}", only);
    }
}
```

```cmd
2026-04-08 16:03:23.914 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo7 - begin : 1
2026-04-08 16:03:23.914 [pool-1-thread-3] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo7 - begin : 3
2026-04-08 16:03:23.914 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo7 - begin : 2
2026-04-08 16:03:24.925 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo7 - end : 1
2026-04-08 16:03:24.925 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo7 - 唯一被执行的任务：1
```



### 关闭线程池

#### 1.shutdowm

```java
/*
将线程池的状态改为 shutdown
此时线程不再接收新的任务，但是会把正在执行的任务和阻塞队列中的任务执行完
调用shutdown的线程，比如main线程，则不会受到线程池的阻塞，例如 main 调用完shutdown之后就会去执行剩余的代码，如果线程此时还有未完的任务，main线程也不会等待他执行完再结束。
**/
void shutdown();
```

实现方式：

```java
    public void shutdown() {
        final ReentrantLock mainLock = this.mainLock;
        // 获取锁
        mainLock.lock();
        try {
            checkShutdownAccess();
            // 修改线程池的状态
            advanceRunState(SHUTDOWN);
            // 只会打断空闲线程
            interruptIdleWorkers();
            onShutdown(); 
        } finally {
            mainLock.unlock();
        }
        // 尝试终结，空闲线程会立即终结，运行的线程也不会等，自己啥时候运行完自己结束就行了
        tryTerminate();
    }
```

测试：

```java
@Slf4j
public class ThreadTestDemo8 {
    public static void main(String[] args) {
        ExecutorService pool = Executors.newFixedThreadPool(2);

        pool.submit(() -> {
            log.debug("第一个任务执行了...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug("第一个任务执行完毕...");
            return 1;
        });

        pool.submit(() -> {
            log.debug("第二个任务执行了...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug("第二个任务执行完毕...");
            return 2;
        });

        pool.submit(() -> {
            log.debug("第三个任务执行了...");
            try {
                // 任务三休眠100秒，模拟任务三执行时间较长
                Thread.sleep(100000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug("第三个任务执行完毕...");
            return 3;
        });

        // 主线程关闭线程池
        log.debug("主线程关闭线程池...");
        // 任务三依然能够被执行
        pool.shutdown();
        // 任务三始终执行不完成，但是主线程可不会受到阻塞
        log.debug("看看我被阻塞了吗");
    }
```

```cmd
结果
2026-04-08 16:49:58.435 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 第一个任务执行了...
2026-04-08 16:49:58.435 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 主线程关闭线程池...
2026-04-08 16:49:58.435 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 第二个任务执行了...
2026-04-08 16:49:58.437 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 看看我被阻塞了吗
2026-04-08 16:49:59.439 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 第一个任务执行完毕...
2026-04-08 16:49:59.439 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 第二个任务执行完毕...
2026-04-08 16:49:59.439 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 第三个任务执行了...
2026-04-08 16:51:39.440 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 第三个任务执行完毕...
```



#### 2.shutdowmNow

```java
/*
将线程池的状态改为stop
此时不再接受新的任务，而且会把正在执行的任务和阻塞队列中的任务都停下来
正在执行的任务会使用 interrupt 的方式打断
阻塞队列中未执行的任务作为结果被返回
**/
List<Runnable> shutdownNow()
    
```

实现方式：

```java
    public List<Runnable> shutdownNow() {
        List<Runnable> tasks;
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            checkShutdownAccess();
            // 修改线程池状态
            advanceRunState(STOP);
            // 打断所有线程
            interruptWorkers();
            // 获取剩余的未执行的任务
            tasks = drainQueue();
        } finally {
            mainLock.unlock();
        }
        // 尝试终结
        tryTerminate();
        return tasks;
    }
```

测试：

```java
Slf4j
public class ThreadTestDemo8 {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService pool = Executors.newFixedThreadPool(2);

        pool.submit(() -> {
            log.debug("第一个任务 begin...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug("第一个任务执行完毕...");
            return 1;
        });

        pool.submit(() -> {
            log.debug("第二个任务 begin...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug("第二个任务执行完毕...");
            return 2;
        });

        pool.submit(() -> {
            log.debug("第三个任务 begin...");
            try {
                // 任务三休眠100秒，模拟任务三执行时间较长
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug("第三个任务执行完毕...");
            return 3;
        });

        // 主线程关闭线程池
        log.debug("主线程关闭线程池...");
        // 任务1，2，3都无法执行完毕
        List<Runnable> runnables = pool.shutdownNow();
        log.debug("看看我被阻塞了吗");

        log.debug("被阻塞的任务：{}", runnables);
    }
}

```

结果：

```java
2026-04-08 17:06:35.294 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 第一个任务 begin...
2026-04-08 17:06:35.294 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 第二个任务 begin...
2026-04-08 17:06:35.294 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 主线程关闭线程池...
2026-04-08 17:06:35.297 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 看看我被阻塞了吗
2026-04-08 17:06:35.297 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 被阻塞的任务：[java.util.concurrent.FutureTask@2957fcb0[Not completed, task = JUC.JUCTest.Demo.D2.ThreadTestDemo8$$Lambda/0x0000017e48008c88@50134894]
```



#### 3.其他方法

```java
// 判断线程池是否被停止，只要不在 RUNNING 状态的线程池，调用此方法都会返回true
boolean isShutdown();

// 判断线程池的状态是 终结
boolean isTerminated();

// 调用 shutdown 之后，主线程并不会等待所有任务执行结束，如果想在线程池被终结之后做一些事情，可以调用这个方法等待
boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;
```

测试：

```java
@Slf4j
public class ThreadTestDemo8 {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService pool = Executors.newFixedThreadPool(2);

        pool.submit(() -> {
            log.debug("第一个任务执行了...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug("第一个任务执行完毕...");
            return 1;
        });

        pool.submit(() -> {
            log.debug("第二个任务执行了...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug("第二个任务执行完毕...");
            return 2;
        });

        pool.submit(() -> {
            log.debug("第三个任务执行了...");
            try {
                // 任务三休眠100秒，模拟任务三执行时间较长
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug("第三个任务执行完毕...");
            return 3;
        });

        // 主线程关闭线程池
        log.debug("主线程关闭线程池...");
        // 任务三依然能够被执行
        pool.awaitTermination(10, TimeUnit.SECONDS);
        // 主线程会等带10s，足够等待任务三执行完毕
        log.debug("看看我被阻塞了吗");
    }
}
```

结果：

```java
2026-04-08 16:56:37.497 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 主线程关闭线程池...
2026-04-08 16:56:37.497 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 第二个任务执行了...
2026-04-08 16:56:37.497 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 第一个任务执行了...
2026-04-08 16:56:38.503 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 第二个任务执行完毕...
2026-04-08 16:56:38.503 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 第一个任务执行完毕...
2026-04-08 16:56:38.504 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 第三个任务执行了...
2026-04-08 16:56:43.505 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 第三个任务执行完毕...
2026-04-08 16:56:47.505 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo8 - 看看我被阻塞了吗
```



## 4.异步模式-工作线程

### 定义

让有限的工作线程（Worker Thread）来轮流异步处理无限多的任务。也可以将其归类为分工模式，它的典型实现就是线程池，也体现了经典设计模式中的享元模式。

例如，海底捞的服务员（线程），轮流处理每位客人的点餐（任务），如果为每位客人都配一名专属的服务员，那么成本就太高了（对比另一种多线程设计模式：Thread-Per-Message）

注意，不同任务类型应该使用不同的线程池，这样能够避免饥饿，并能提升效率
例如，如果一个餐馆的工人既要招呼客人（任务类型 A），又要到后厨做菜（任务类型 B）显然效率不咋地，分成服务员（线程池 A）与厨师（线程池 B）更为合理，当然你能想到更细致的分工



### 饥饿

固定大小的线程池会出现饥饿现象

两个工人是同一个线程池中的两个线程

他们要做的事情是：为客人点餐和到后厨做菜，这是两个阶段的工作

- 客人点餐：必须先点完餐，等菜做好，上菜，在此期间处理点餐的工人必须等待
- 后厨做菜：没啥说的，做就是了

比如工人 A 处理了点餐任务，接下来它要等着 工人 B 把菜做好，然后上菜，他俩也配合的蛮好

但现在同时来了两个客人，这个时候工人 A 和工人 B 都去处理点餐了，这时没人做饭了，饥饿 



正常情况

线程1先点菜，然后线程2负责做菜，做完之后线程一调用get得到才菜品，随后线程1上菜。

```java
@Slf4j
public class ThreadTestDemo9 {
    static List<String> meals = Arrays.asList("米饭", "炸鸡", "炒菜");
    // 随机做一个菜
    static Random r = new Random();

    public static void main(String[] args) {
        // 创建一个固定大小的线程池
        ExecutorService pool = Executors.newFixedThreadPool(2);

        pool.execute(() -> {
            //线程一负责点菜
            log.debug("点菜...");

            Future<String> future = pool.submit(() -> {
                // 线程二负责做菜
                log.debug("做菜...");
                return cooking();
            });

            try {
                log.debug("上菜：{}", future.get());
            } catch (InterruptedException | ExecutionException e) {
                throw new RuntimeException(e);
            }
        });
    }

    // 做菜
    public static String cooking() {
        // 随机返回一个菜品
        return meals.get(r.nextInt(meals.size()));
    }
}

```

结果：

```java
2026-04-08 20:58:30.664 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo9 - 点菜...
2026-04-08 20:58:30.669 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo9 - 做菜...
2026-04-08 20:58:30.669 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo9 - 上菜：米饭
```



饥饿情况：

两个线程都去点菜了，没人能够做菜，结果双方都调用get等着拿菜，结果出现饥饿。

```java
@Slf4j
public class ThreadTestDemo9 {
    static List<String> meals = Arrays.asList("米饭", "炸鸡", "炒菜");
    static Random r = new Random();

    public static void main(String[] args) {
        // 创建一个固定大小的线程池
        ExecutorService pool = Executors.newFixedThreadPool(2);

        pool.execute(() -> {
            //线程一负责点菜
            log.debug("点菜...");

            Future<String> future = pool.submit(() -> {
                // 线程二负责做菜
                log.debug("做菜...");
                return cooking();
            });

            try {
                log.debug("上菜：{}", future.get());
            } catch (InterruptedException | ExecutionException e) {
                throw new RuntimeException(e);
            }
        });

        pool.execute(() -> {
            log.debug("点菜...");

            Future<String> future = pool.submit(() -> {
                log.debug("做菜...");
                return cooking();
            });

            try {
                log.debug("上菜：{}", future.get());
            } catch (InterruptedException | ExecutionException e) {
                throw new RuntimeException(e);
            }
        });
    }

    // 做菜
    public static String cooking() {
        // 随机返回一个菜品
        return meals.get(r.nextInt(meals.size()));
    }
}
```

```java
2026-04-08 21:02:12.321 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo9 - 点菜...
2026-04-08 21:02:12.321 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo9 - 点菜...
```



#### 解决办法

给不同的任务设置不同类型的线程池。

比如：点餐就到点餐线程池当中去获取线程执行任务；做菜你就到做菜线程池当中去获取线程执行任务。这样，双方互不干扰。

```java
@Slf4j
public class ThreadTestDemo9 {
    static List<String> meals = Arrays.asList("米饭", "炸鸡", "炒菜");
    static Random r = new Random();

    public static void main(String[] args) {
        // 创建一个固定大小的线程池
        ExecutorService waiterPool = Executors.newFixedThreadPool(1);
        ExecutorService cookPool = Executors.newFixedThreadPool(1);

        waiterPool.execute(() -> {
            //线程一负责点菜
            log.debug("点菜...");

            Future<String> future = cookPool.submit(() -> {
                // 线程二负责做菜
                log.debug("做菜...");
                return cooking();
            });

            try {
                log.debug("上菜：{}", future.get());
            } catch (InterruptedException | ExecutionException e) {
                throw new RuntimeException(e);
            }
        });

        waiterPool.execute(() -> {
            log.debug("点菜...");

            Future<String> future = cookPool.submit(() -> {
                log.debug("做菜...");
                return cooking();
            });

            try {
                log.debug("上菜：{}", future.get());
            } catch (InterruptedException | ExecutionException e) {
                throw new RuntimeException(e);
            }
        });
    }

    // 做菜
    public static String cooking() {
        // 随机返回一个菜品
        return meals.get(r.nextInt(meals.size()));
    }
}
```

结果：；

```java
2026-04-08 21:12:14.840 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo9 - 点菜...
2026-04-08 21:12:14.843 [pool-2-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo9 - 做菜...
2026-04-08 21:12:14.843 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo9 - 上菜：炒菜
2026-04-08 21:12:14.843 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo9 - 点菜...
2026-04-08 21:12:14.843 [pool-2-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo9 - 做菜...
2026-04-08 21:12:14.845 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDemo9 - 上菜：炸鸡
```



### 线程池大小

- 线程池太小，会导致系统资源利用不充分

- 太大，上下文切换太频繁，占用更多内存



需要根据业务需求来决定线程池的大小。

#### 1.CPU密集型

将线程池的核心数设为 `cpu核心数 + 1`，充分利用cpu，+1是保证，操作系统可能会出现页缺失或者其他故障原因导致暂停，这个时候多出来的线程就能起到作用了。



#### 2.I/O 密集型运算

CPU 不总是处于繁忙状态，例如，当你执行业务计算时，这时候会使用 CPU 资源，但当你执行 I/O 操作时、远程 RPC 调用时，包括进行数据库操作时，这时候 CPU 就闲下来了，你可以利用多线程提高它的利用率。

经验公式如下

线程数 = 核数 * 期望 CPU 利用率 * 总时间 (CPU 计算时间 + 等待时间) / CPU 计算时间

例如 4 核 CPU 计算时间是 50%，其它等待时间是 50%，期望 cpu 被 100% 利用，套用公式

```
4 * 100% * 100% / 50% = 8
```

例如 4 核 CPU 计算时间是 10%，其它等待时间是 90%，期望 cpu 被 100% 利用，套用公式

```
4 * 100% * 100% / 10% = 40
```



### 任务调度线程池

如果希望某个任务是延时执行的，或者被反复的周期性执行，**就可以使用任务调度线程池**



#### ScheduledThreadPoolExecutor

可以实现实现任务调度线程池。

**特点：**

1. 线程复用：通过线程池复用机制，避免了线程频繁创建和销毁的开销。

2. 多任务并发支持：支持同时调度多个任务。

3. 定时精度：基于系统时间，调度精确且高效。

4. 任务中断机制：能够在需要时灵活关闭或取消任务。

   

- #### 创建方式：

```java
// 两种创建方式

// 1.通过构造方法，自定义参数创建 ScheduledThreadPoolExecutor 对象
new ScheduledThreadPoolExecutor(int corePoolSize,
                                ThreadFactory threadFactory,
                                RejectedExecutionHandler handler);

// 通过 Executors 创建ScheduledThreadPoolExecutor 对象
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
     return new ScheduledThreadPoolExecutor(corePoolSize);
}
```

比如：

```java
ScheduledExecutorService pool1 = Executors.newScheduledThreadPool(2);
ScheduledThreadPoolExecutor pool2 = new ScheduledThreadPoolExecutor(2);
```



- #### 常用API

```java
// 创建并执行在给定延迟后启用的一次性操作，delay：延迟的时间，unit: 时间单位
schedule(Runnable command, long delay, TimeUnit unit)

/**
特点：按照固定的频率执行任务，不考虑任务执行时间

执行逻辑：任务的开始时间是固定的
如果任务执行时间 < 间隔时间：按固定间隔执行
如果任务执行时间 > 间隔时间：任务会立即连续执行（无间隔）
适合场景：对执行频率要求严格，任务执行时间相对固定的场景
**/
scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)
    
/**
特点：按照固定的延迟执行任务，考虑任务执行时间

执行逻辑：任务的结束时间 + 延迟时间 = 下一次开始时间
无论任务执行多长时间，都会等待固定延迟后再执行下一次
适合场景：需要保证任务之间有最小间隔的场景
**/
scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit)
```

测试：

1.延时执行

```java
@Slf4j
public class ThreadTestDem10 {
    public static void main(String[] args) {
        ScheduledExecutorService pool = Executors.newScheduledThreadPool(2);

        // 1s之后执行
        pool.schedule(() -> {
            log.debug("1s之后执行");
        },1, TimeUnit.SECONDS);
        
        // 2s之后执行
        pool.schedule(() -> {
            log.debug("2s之后执行");
        },2, TimeUnit.SECONDS);

        log.debug("开始执行");
    }
}

2026-04-08 22:07:25.610 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem10 - 开始执行
2026-04-08 22:07:26.616 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem10 - 1s之后执行
2026-04-08 22:07:27.617 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem10 - 2s之后执行
```

- #### scheduleAtFixedRate：

```java
@Slf4j
public class ThreadTestDem10 {
    public static void main(String[] args) {
        ScheduledExecutorService pool = Executors.newScheduledThreadPool(2);

        log.debug("start");
        pool.scheduleAtFixedRate(() -> {
            log.debug("running...");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        },1,1,TimeUnit.SECONDS);
    }
}

2026-04-08 22:32:28.371 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem10 - start
2026-04-08 22:32:29.381 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem10 - running...
2026-04-08 22:32:31.383 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem10 - running...
2026-04-08 22:32:33.394 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem10 - running...
2026-04-08 22:32:35.406 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem10 - running...
2026-04-08 22:32:37.415 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem10 - running...
2026-04-08 22:32:39.416 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem10 - running...
```

- #### scheduleWithFixedDelay：

```java
@Slf4j
public class ThreadTestDem10 {
    public static void main(String[] args) {
        ScheduledExecutorService pool = Executors.newScheduledThreadPool(2);

        log.debug("start");
        pool.scheduleWithFixedDelay(() -> {
            log.debug("running...");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        },1,1,TimeUnit.SECONDS);
    }
}
```

![](juc并发编程 .assets/屏幕截图 2026-04-08 223430.png)



## 5.Tomcat 线程池

Tomcat的连接器：

### 工作方式

![](juc并发编程 .assets/屏幕截图 2026-04-09 170733.png)

- LimitLatch：是一个限流器，用于限流，可以控制想要连接Tomcat服务器的最大连接数量。
- Acceptor：用于接收新的scoket请求，就是一个线程，里面不断循环，检查是否有新的连接请求。
- Pooler：只负责监听新的 scoket 请求中是否包含 可读的I/O请求，如果包含，就会封装一个新的任务对象（socketProcessor），然后将这个任务对象交给后续的线程池Executor去处理。
- Executor：线程池有多个**工作线程**，负责处理前面所有工作传过来的**处理请求**。



### 实现方式

Tomcat 线程池是由JDK的 ThreadPoolExecutor 扩展而来的。

但是 Tomcat 线程池又和ThreadPoolExecutor有所不同。

**ThreadPoolExecutor**中，如果总的线程数达到了最大线程数的时候

- 这时候再提交新的任务，就会执行拒策略，默认是抛出 RejectedExecutionException异常，任务提交就会失败。

但是 Tomact 对 ThreadPoolExecutor做了扩展，底层对 execute 方法进行了重写

提交任务的时候，如果发现：

- 总线程数已经达到了最大线程数

  - 这个时候他不着急，还不会立即就抛出 RejectedExecutionException 异常。

  - 他会尝试再次将这个任务尝试加入到阻塞队列当中，如果此时对列有空位了，那他加入即可；如果没有，此时才会抛出 RejectedExecutionException 异常。

    

```java
/**
* 重写了 execute 方法
**/
public void execute(Runnable command, long timeout, TimeUnit unit) {
    submittedCount.incrementAndGet();
    try {
        super.execute(command);
    } catch (RejectedExecutionException rx) { // 将 RejectedExecutionException 异常捕捉住了
        if (super.getQueue() instanceof TaskQueue) {
            // 获取到阻塞队列
            final TaskQueue queue = (TaskQueue)super.getQueue();
            // 尝试将任务再次加入到阻塞队列当中
            try {
                // 如果还是失败了，就会抛出 RejectedExecutionException 异常
                // force 底层调用了带超时的的提交任务方法，时效之内未提交成功，就会返回false
                if (!queue.force(command, timeout, unit)) {
                    submittedCount.decrementAndGet();
                    throw new RejectedExecutionException("Queue capacity is full.");
                }
            } catch (InterruptedException x) {
                submittedCount.decrementAndGet();
                Thread.interrupted();
                throw new RejectedExecutionException(x);
            }
        } else {
            submittedCount.decrementAndGet();
            throw rx;
        }
    }
}
```



## 6.Fork/Join 线程池

Fork / Join 是一线程池的实现，体现的是一种分治的思想。

比如：有一个非常大的任务，但是我们能够将这个非常大的任务进行拆分，拆分成多个计算方法相同的小任务，直到不能再拆了，然后对这些小任务使用多线程的方式进行并行计算，把每个任务分给不同的线程执行，最终汇总这些任务的结果，得到最终结果。

Fork / Join 适用于CPU密集型的运算任务。

<img src="juc并发编程 .assets/261bcb8c40ad4a1a8144da8b87d4ebbftplv-73owjymdk6-jj-mark-v1_0_0_0_0_5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5byC5bi45ZCb_q75.webp" style="zoom: 50%;" />



### 使用方式

- #### 创建 Fork/Join 线程池

```java
// 通过构造函数创建，构造函数中参数为空，默认创建的核心线程数与电脑的核心数一样
ForkJoinPool pool = new ForkJoinPool();

// 创建指定并行度的ForkJoinPool
ForkJoinPool pool = new ForkJoinPool(4);

// 使用Java 8引入的公共池
ForkJoinPool commonPool = ForkJoinPool.commonPool();
```

- #### 方法

```java
任务方法：

// 指定执行的任务类型：包含两种
// 子类可以继承这两类任务类型，并重写 compute() 方法，指定任务执行逻辑

RecursiveTask ：//有返回值的任务，需要重写抽象方法 compute()，指定执行逻辑，并返回值

RecursiveAction ：//无返回值的任务，需要重写抽象方法 compute()，指定执行逻辑，无返回值、

             compute() //子类必须实现的方法，包含任务的具体计算逻辑

fork()：// 让线程异步执行任务

join()：// 等待任务完成并获取结果
```



```java
线程池方法：

invoke（invoke(ForkJoinTask<T> task)）：执行任务
```

测试：

```java
@Slf4j
public class ThreadTestDem11 {
    public static void main(String[] args) {
        // 创建 ForkJoinPool 线程池
        ForkJoinPool pool = new ForkJoinPool(4);
        // 创建任务
        Task task = new Task(5);
        // 执行任务
        Integer result = pool.invoke(task);
        log.debug("result:{}", result);
    }
}

class Task extends RecursiveTask<Integer> {
    private Integer m;

    public Task(Integer m) {
        this.m = m;
    }

    // 实现 1~m 的和计算
    @Override
    protected Integer compute() {
        // 终止条件
        if (m == 1) {
            return 1;
        }
        // 递归调用
        Task task = new Task(m - 1);
        // 让新的线程去执行任务
        task.fork();
        // 获取任务结果
        int result = m + task.join();
        return result;
    }
}
```



# 十九。JUC工具包



## 1.AQS 原理

### 概述

全称是 AbstractQueuedSynchronizer，是阻塞式锁和相关的同步器工具的框架

特点：

- 内部维护了一个 state 属性，用来表示资源的状态，资源状态：分为独占模式 和 共享模式。而这个状态需要由子类去维护，子类控制锁的获取以及释放的操作方式。

  比如：0 代表没有获取锁，1代表已经获取到锁了。

  - getState：获取 state 状态
  - setState：设置 state 状态
  - compareAndSetState -：使用 CAS 机制来设置state的状态，保证 state 的原子性
  - `独占模式`是只有`一个线程`能够访问资源，而`共享模式`可以允许`多个线程`访问资源

- 提供了基于 `FIFO`（先进先出） 的等待队列，类似于 Monitor 的 EntryList，用于存放还未获取的锁的线程

- `条件变量`来实现等待、唤醒机制，支持多个条件变量，类似于 Monitor 的 WaitSet（存放已经获取锁但又释放了的线程）

想要使用AQS，需要由子类继承，并重写以下方法（以下方法默认抛出 `UnsupportedOperationException`）

- `tryAcquire`
- `tryRelease`
- `tryAcquireShared`
- `tryReleaseShared`
- `isHeldExclusively`

#### 获取锁

```java
// 成功返回true，失败返回false
tryAcquire(arg); 

if (!tryAcquire(arg)) {
    // 获取锁失败，进入阻塞队列，可以选择阻塞当前线程  
    // AQS中让线程进入阻塞或者唤醒线程使用的是 park 和 unpark
}
```

#### 释放锁

```java
// 成功返回true，失败返回false
tryRelease(arg);

if (tryRelease(arg)) {
    // 让阻塞线程恢复运行
}
```



### 自定义锁

#### 定义同步器

AQS中维护了 state 变量来表示锁状态，默认为 0 ，代表未占有锁。

```java
/**
* 同步状态
*/
private volatile int state;
```

先定义一个同步器，实现一个不可重入锁（如果一个线程已经获得了该锁，再想获得这把锁就会失败，只能获得一次）

同步器内实现AQS中的 `tryAcquire`获取锁，`tryRelease`释放锁，`isHeldExclusively` 判断是否当前线程是独占锁的线程。

```java
    /**
     * 自定义同步器，独占锁
     */
    class MySync extends AbstractQueuedSynchronizer {

        // 加锁
        @Override
        protected boolean tryAcquire(int arg) {
            // 使用 CAS 操作加锁，此 compareAndSetState 方法源于 AbstractQueuedSynchronizer
            int state = this.getState(); // 如果未加锁，默认为0
            if (this.compareAndSetState(state,1)) {
                // 加锁成功，将此锁的 owner 设置为当前线程
                this.setExclusiveOwnerThread(Thread.currentThread());
                // 返回 true
                return true;
            }
            // 加锁失败，返回false
            return false;
        }

        // 释放锁
        @Override
        protected boolean tryRelease(int arg) {
            // 将锁的持有者设置为 null
            this.setExclusiveOwnerThread(null);
            // 将 state 的状态改为 0（未加锁状态），这里没有使用 CAS 操作是因为此锁已经被该线程持有，不存在并发安全问题
            // 先将持有者设置为 null，再将 state 改为 0，能够确保可见性
            this.setState(0);
            // 释放锁成功，返回 true
            return true;
        }

        // 判断是否当前线程是独占锁的线程
        @Override
        protected boolean isHeldExclusively() {
            // 判断 state的值是否为 1，如果为 1 代表锁已经被占有
            return this.getState() == 1;
        }

        public Condition newCondition() {
            return new ConditionObject();
        }
    }
```

内部使用的 `compareAndSetState`，`setExclusiveOwnerThread`等方法，都是父类 AQS 中的方法。

这里有一个细节，`tryRelease` 释放锁的操作中，应该先将持有者设置为 null，再将 state 改为 0，因为`setExclusiveOwnerThread`方法会将 `exclusiveOwnerThread`这个变量这是为 null，但是这个变量没有被 volatile 所修饰，所以如果最后才将持有者设置为 null，可能会出现可见性问题，所以应该先将持有者设置为 null，再将 state 改为 0，因为 state 被 volatile 修饰，更新值的时候，会在最后加入写屏障，保证了前面所有共享变量的可见性。

![](juc并发编程 .assets/屏幕截图 2026-04-12 155831-1775981548816-6.png)

![屏幕截图 2026-04-12 155942](juc并发编程 .assets/屏幕截图 2026-04-12 155942-1775981548815-5.png)



#### 使用同步器，自定义锁

同步器定义好之后，就可以使用了

```java
/**
 * 自定义锁（不可重入锁）
 */
class MyLock implements Lock {

    /**
     * 自定义同步器，独占锁
     */
    class MySync extends AbstractQueuedSynchronizer {

        // 加锁
        @Override
        protected boolean tryAcquire(int arg) {
            // 使用 CAS 操作加锁，此 compareAndSetState 方法源于 AbstractQueuedSynchronizer
            // 如果未加锁，默认为0
            if (this.compareAndSetState(0,1)) {
                // 加锁成功，将此锁的 owner 设置为当前线程
                this.setExclusiveOwnerThread(Thread.currentThread());
                // 返回 true
                return true;
            }
            // 加锁失败，返回false
            return false;
        }

        // 释放锁
        @Override
        protected boolean tryRelease(int arg) {
            // 将锁的持有者设置为 null
            this.setExclusiveOwnerThread(null);
            // 将 state 的状态改为 0（未加锁状态），这里没有使用 CAS 操作是因为此锁已经被该线程持有，不存在并发安全问题
            // 先将持有者设置为 null，再将 state 改为 0，能够确保可见性
            this.setState(0);
            // 释放锁成功，返回 true
            return true;
        }

        // 判断是否当前线程是独占锁的线程
        @Override
        protected boolean isHeldExclusively() {
            // 判断 state的值是否为 1，如果为 1 代表锁已经被占有
            return this.getState() == 1;
        }

        public Condition newCondition() {
            return new ConditionObject();
        }
    }

    private MySync sync = new MySync();

    // 加锁，失败后进入等待对列等待
    @Override
    public void lock() {
        sync.acquire(1);
    }

    // 加锁，但是可以被其他线程打断
    @Override
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    // 尝试加锁，只尝试一次，失败后返回false
    @Override
    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    // 带超时的尝试加锁
    @Override
    public boolean tryLock(long time, @NotNull TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(time));
    }

    // 解锁
    @Override
    public void unlock() {
        sync.release(1);
    }

    // 创建新的条件变量
    @NotNull
    @Override
    public Condition newCondition() {
        return sync.newCondition();
    }
}
```

` sync.acquire(1);` `sync.acquireInterruptibly(1);`  `sync.tryAcquireNanos(1, unit.toNanos(time));` 都是间接的调用了我们重写的 `tryAcquire` 方法，只不过我们自定义的只是尝试加锁一次，失败就返回false，而这些方法都能够对于加锁失败做出一些相应操作。

<img src="juc并发编程 .assets/屏幕截图 2026-04-12 161559.png" style="zoom:67%;" />

<img src="juc并发编程 .assets/屏幕截图 2026-04-12 161821.png" alt="屏幕截图 2026-04-12 161821" style="zoom:67%;" />

测试：

```java
@Slf4j
public class ThreadTestDem12 {
    public static void main(String[] args) {
        MyLock lock = new MyLock();
        new Thread(() -> {
            lock.lock();
            try {
                log.debug("我获取到了锁");
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }

            } finally {
                log.debug("我释放了锁");
                lock.unlock();
            }
        },"t1").start();

        new Thread(() -> {
            lock.lock();
            try {
                // 等待 t1 线程释放锁 - 5s之后会获得锁
                log.debug("我也获取到了锁");
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }

            } finally {
                log.debug("我也释放了锁");
                lock.unlock();
            }
        },"t2").start();
    }
}

2026-04-12 16:58:53.156 [t1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem12 - 我获取到了锁
2026-04-12 16:58:58.172 [t1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem12 - 我释放了锁
2026-04-12 16:58:58.173 [t2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem12 - 我也获取到了锁
2026-04-12 16:59:03.178 [t2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem12 - 我也释放了锁

```

因为是不可重入锁，所以同一个线程要是想再次获得这把锁，就会陷入阻塞。

```java
 new Thread(() -> {
            lock.lock();
            try {
                log.debug("我获取到了锁");
                // 尝试再次获得这把锁
                lock.lock();
                log.debug("我再次获取到了锁");
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }

            } finally {
                log.debug("我释放了锁");
                lock.unlock();
            }
        },"t1").start();

2026-04-12 17:05:39.311 [t1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem12 - 我获取到了锁
```



## 2.ReentrantLock 底层原理

ReentrantLock 实现依靠 AQS + CAS 实现加锁、解锁的操作。

![](juc并发编程 .assets/屏幕截图 2026-04-12 171650.png)

ReentrantLock 也是实现了 Lock 接口，  内部也是维护了一个同步器 sync（内部类），这个同步器 sync 也是继承了AQS（AbstractQueuedSynchronizer），但是这个 sync 是一个抽象类，他有两个实现类`NonfairSync`，`FairSync`，分别是 **非公平锁** 和 **公平锁**。

- 公平锁：多个线程按照申请锁的顺序去获得锁，线程会直接进入队列去排队，永远都是队列的第一位才能得到锁。

- 非公平锁：多个线程去获取锁的时候，会直接去尝试获取，获取不到，再去进入等待队列，如果能获取到，就直接获取到锁。

![](juc并发编程 .assets/屏幕截图 2026-04-13 150843.png)

以下是非公平锁的实现方式。

### 非公平锁实现原理

#### （1）加锁原理

当通过ReentrantLock  的构造函数来创建一个锁时，默认是创建一个非公平锁

```java
// 构造器
public ReentrantLock() { 
    // 默认创建非公平锁
    sync = new NonfairSync();
}
```

这个非公平锁 NonfairSync 也是继承了 AQS。

当调用了 ReentrantLock  中的`lock`方法尝试加锁的时候，ReentrantLock  中的 lock 方法中又会调用 sync 中的 `lock`方法，这个 sync 中的 lock 方法又会调用 NonfairSync 中的 `lock`方法，而这个 lock 方法的逻辑是这样的：

```java
        // ReentrantLock 中的 lock 方法
    	public void lock() {
                sync.lock();
         }

        // 非公平锁加锁
        final void lock() {
            // 获取锁资源，CAS 修改 AbstractQueuedSynchronizer 的 state 属性值由0改为1,设置成功
            // 设置AbstractQueuedSynchronizer的exclusiveOwnerThread为当前线程
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            // 获取失败，执行AbstractQueuedSynchronizer的acquire
            else
                acquire(1);
        }
```





- ### **当没有竞争的时候**

此时T-0会尝试将 `state`从0设置为1，并且肯定能设置成功，并且会将锁的持有者设置为当前线程，加锁成功。

![](juc并发编程 .assets/屏幕截图 2026-04-12 182618.png)





- ### **当出现第一个竞争**的时候

T-1 线程也尝试将 state 设置为1 来加锁，但是此时 state 的状态已经为1了，已经被 T-0 线程加锁了，所以 compareAndSetState 执行 CAS 加锁操作肯定会失败，此时就会走 lock 中 else 段中的 `acquire` 方法的逻辑。

而`acquire`中的逻辑是这样的，代码如下，首先会调用 `tryAcquire`，这是再次尝试进行一次加锁操作，因为有可能此时锁会被释放了，如果释放了那么此时这个 T-1 线程就有机会够通过这次 tryAcquire 获取到锁了。

但是如果再次获取锁失败，tryAcquire 就会返回 false，一取反，就会执行 && 的后半部分：`acquireQueued(addWaiter(Node.EXCLUSIVE), arg)`

```java
// 获取锁资源
public final void acquire(int arg) {
    // 尝试获取锁资源
    if (!tryAcquire(arg) &&
        // 当前线程未获取到锁资源时，会进入等待队列，同时线程被挂起，等待后续唤醒
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

这个就有点麻烦了:

![](juc并发编程 .assets/屏幕截图 2026-04-12 183421.png)

此时首先会进入`addWaiter`的逻辑内：

- 构造一个**Node**对列，刚开始 head 、tail 都为空。这是一个双向链表。

- 此时有竞争了，尝试会去添加一个 Node 节点，但是第一次去创建 Node 节点的时候会去创建两个Node节点

  <img src="juc并发编程 .assets/屏幕截图 2026-04-13 135324.png" style="zoom:67%;" />

- 第一个Node节点啥都没有，用来占位的，称为哑元。第二个Node节点，线程 T-1 会关联到他身上。

- 节点上方的三角形表示节点的 waitStatus 状态，0代表正常状态。

![](juc并发编程 .assets/屏幕截图 2026-04-12 232255.png)

创建完节点之后，会进入 `acquireQueued` 的逻辑内部：

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            // 获取前驱节点
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

- acquireQueued 内部是是一个死循环，不断尝试再次获取锁

- 首先会获取当前节点的前驱节点，并判断前驱节点是不是头节点

  - 如果是头节点，那就证明，我当前这个节点是第二个节点，排在第二位，那他就能够调用`tryAcquire`尝试再次获取锁，如果此时 state 的状态还是为1的话，那加锁还是会失败。

- 如果还是失败了，就会进入`shouldParkAfterFailedAcquire`内部

  - 它会将前驱节点的 **waitStatus** 设置为 -1，表示他有责任唤醒他的后继节点。这个方法会返回false

  

  ![](juc并发编程 .assets/屏幕截图 2026-04-12 233903.png)

  - 之后又进行下一次循环，但是加锁还是失败了，就又进入了 shouldParkAfterFailedAcquire，但这次，前驱节点的 **waitStatus** 已经是 -1了，这次就会返回true，并且进入`&& parkAndCheckInterrupt()`
  - parkAndCheckInterrupt 中会调用 LockSupport.park()，此时线程会陷入阻塞。

![](juc并发编程 .assets/屏幕截图 2026-04-12 234923.png)

所以线程T-1再经历第一次竞争失败的时候，还会尝试很多次加锁，在经历多次加锁失败之后，再会进入阻塞。

- ### 如果 T-0 线程占有锁之后迟迟不释放锁，这个时候又有多个线程尝试获取锁

那他们就都会竞争失败，被阻塞住，并且各自的 waitStatus 的状态都会被设置为 -1，代表他们有责任去唤醒他们的后继节点。但是最后一个节点的 waitStatus 的状态还是 0 ，因为他后继没有节点。

![](juc并发编程 .assets/屏幕截图 2026-04-13 135809.png)



#### （2）解锁流程

当调用 ReentrantLock中的 unlock 放法解锁的时候，内部会调用同步器 sync 内的 `release`方法，而 release 方法内部有会调用 `tryRelease`方法

```java
// ReentrantLock
public void unlock() {
    sync.release(1);
}

// sync 
public final boolean release(int arg) {
    // 当前线程释放锁资源的计数值,AbstractQueuedSynchronizer的state属性用于可重入
    if (tryRelease(arg)) {
        // 当前线程释放锁资源，获取等待队列头节点
        Node h = head;
        if (h != null && h.waitStatus != 0)
            // 唤醒等待队列中后继待唤醒的节点
            unparkSuccessor(h);
        // 释放锁资源
        return true;
    }
    // 当前线程未完全释放锁资源
    return false;
}

// 释放锁
protected final boolean tryRelease(int releases) {
    // 修改 AbstractQueuedSynchronizer 的 state
    int c = getState() - releases;
    // 当前线程若不是持有锁的线程，抛出异常
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    // 是否成功的将锁资源完全释放标识 （state == 0）
    boolean free = false;
    // state == 0，锁资源完全释放，不存在重入
    if (c == 0) {
        // 修改标识
        free = true;
        // 将占用锁资源的线程AbstractQueuedSynchronizer 的exclusiveOwnerThread属性设置为null
        setExclusiveOwnerThread(null);
    }
    // state赋值
    setState(c);
    // 返回true表示当前线程完全释放锁state =0；
    // 返回false标识当前线程是有锁，state -1
    return free;
}
```

`ryRelease` 内部的执行逻辑是这样的：

- 它会将当前锁的持有者 **exclusiveOwnerThread** 属性设置为空
- 并且将资源状态 state 设置为 0

![](juc并发编程 .assets/屏幕截图 2026-04-13 141758.png)

如果此时 tryRelease 执行成功，就会回到 release 中，因为 tryRelease 返回 true ，就会执行 if 段中的逻辑：

- 他会检查当前队列中的头节点 head 是否不为空，并且检查他的 waitStatus 是否不为 0 ，如果都满足，就进入 ` unparkSuccessor(h)`中，那他就能够去唤醒他的下一个被阻塞的节点。
- unparkSuccessor 中会找到离 head 最近的一个 Node 节点，并调用 Support.unpark 将T-1线程回复运行。
- T-1恢复运行，就又会回到 `acquireQueued`方法内部去执行逻辑（因为线程T-1是在 acquireQueued 不断尝试加锁操作，但是都失败了）



- ### 如果此时没有其他线程竞争，那T-1就能够获得锁，加锁成功

- 此时会将 exclusiveOwnerThread 设置为 T-1，并且将 state 设置为 1
- 然后会把 T-1线程从他关联的这个节点中清除，此时这个节点就会成为一个新的空节点 Node2
- head 就不再指向原来的那个空节点 Node1了，而是指向这个全新的空节点 Node2
- 原本的 Node1 就会从链表中断开，等待被垃圾回收

![](juc并发编程 .assets/屏幕截图 2026-04-13 144227.png)



- ### 如果这时有其他线程竞争

因为是非公平锁，所以这个时候又新来一个线程，他会直接去尝试获取锁，而不是进入等待对列，会和T-1去竞争同一把锁

结果T-1没竞争过，锁被T-4抢走了

- T-4就会被设置为 exclusiveOwnerThread  ，并且 state 被设置为 1
- T-1因为没竞争过，就又会再次进入 `acquireQueued`中，获取锁失败，就又会进入阻塞。

![](juc并发编程 .assets/屏幕截图 2026-04-13 150011.png)



### 可重入原理

非公平锁的获取是可重入的，即一个线程如果获取到了该锁，当他再次获取这把锁的时候，依然能够获取到。

非公锁最终都会调用 `nonfairTryAcquire`方法尝试获取锁

```java
// 非公平锁：加锁
final boolean nonfairTryAcquire(int acquires) {
     // 当前线程
    final Thread current = Thread.currentThread();
    // 获取AbstractQueuedSynchronizer的 state
    int c = getState();
    // 无线程占用锁
    if (c == 0) {
        // CAS 修改 state 的值，修改成功，设置线程属性为当前线程，返回占用锁资源标识
        if (compareAndSetState(0, acquires)) {
            //抢锁成功，同时无线程排队，将AbstractQueuedSynchronizer的exclusiveOwnerThread设置为当前线程
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // 有线程占用锁
    // 判断占用锁的线程exclusiveOwnerThread是否为当前线程
    else if (current == getExclusiveOwnerThread()) {
        // AbstractQueuedSynchronizer的 state + 1
        int nextc = c + acquires;
        // 超出锁重入的上限(int的最大值)，抛异常
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        // 将 state + 1 
        setState(nextc);
        return true;
    }
    return false;
}
```

nonfairTryAcquire 具有可重入性

执行流程是这样的：

- 首先会获取到当前线程，以及 AbstractQueuedSynchronizer 中的 state 的值
- 如果此时 state = 0，表明还未加锁，那么该线程就能够获取到锁
- 但是如果 state != 0，就表明该锁已经被占用
  - 之后会校验该锁的持有者是不是当前线程，如果是当前线程
    - 那就会把 **state 的值+1**，代表当前线程**重入次数+1**，当前线程获得了两次锁

当同一个线程释放锁的时候会执行 `tryRelease`，执行逻辑是这样的：

```java
// 释放锁
protected final boolean tryRelease(int releases) {
    // 修改 AbstractQueuedSynchronizer 的 state
    int c = getState() - releases;
    // 当前线程若不是持有锁的线程，抛出异常
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    // 是否成功的将锁资源完全释放标识 （state == 0）
    boolean free = false;
    // state == 0，锁资源完全释放，不存在重入
    if (c == 0) {
        // 修改标识
        free = true;
        // 将占用锁资源的线程AbstractQueuedSynchronizer 的exclusiveOwnerThread属性设置为null
        setExclusiveOwnerThread(null);
    }
    // state赋值
    setState(c);
    // 返回true表示当前线程完全释放锁state =0；
    // 返回false标识当前线程是有锁，state -1
    return free;
}
```

接受的参数release是1，代表减去一次

- 会将 state 的值减一，代表重入此时减去一次

- 内部维护了一个布尔标识 `free`，代表当前的这把锁是不是已经被完全释放了，默认为 false，还没有完全释放

- 只要 state 的值不为0，free 就一直为 false

- 当 state 的值为0了，代表已经完全释放掉了，这个时候就会把锁的持有者 exclusiveOwnerThread 设置为null

- 之后 free 就被设置为为 true，代表锁已经完全释放掉了，并且将 free 返回，用于去判断能不能去唤醒被阻塞的线程了

  

### 可打断原理

- ### 不可打断模式

ReentrantLock默认是不可打断的：

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            // 获取前驱节点
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
               parkAndCheckInterrupt()) {
                interrupted = true;
            }
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

线程要尝试获得锁的时候，就会进入 acquireQueued ，如果尝试一直不成功，就会进入 ` parkAndCheckInterrupt()`方法内等待，但是调用 park 方法被阻塞的线程，可以被其他线程调用interrupt打断阻塞，继续执行。

比如：

```java
private final boolean parkAndCheckInterrupt() {
    // 如果打断标记已经是 true，则 park 会失效
    LockSupport.park(this);
    // interrupted 会清除打断标记
    return Thread.interrupted();
}
```

调用LockSupport.park(this);进行了阻塞，但是其他线程调用 interrupt 将其打断，那他就会接着向后执行，执行`Thread.interrupted()`，会返回该线程是否被打断过，如果被打断过就会返回 true，并且他还会将打断标记进行清除，置为false，保证下一次在进行打断的时候不会受到影响。

因为 Thread.interrupted() 返回 true，就会进入

```java
if (shouldParkAfterFailedAcquire(p, node) &&
               parkAndCheckInterrupt()) {
                interrupted = true;
            }
```

此时会将 interrupted 打断标记置为true，并且返回到 `acquire ` 中

```java
public final void acquire(int arg) {
    // 尝试获取锁资源
    if (!tryAcquire(arg) &&
        // 当前线程未获取到锁资源时，会进入等待队列，同时线程被挂起，等待后续唤醒
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg)){
        selfInterrupt();
    }
}
```

因为打断标记是真，此时就会执行 if 中的逻辑，会调用 `selfInterrupt()`:

```java
static void selfInterrupt() {
    // 重新产生一次中断
    Thread.currentThread().interrupt();
}
```

这时候又会重新产生一次打断。

所以在此模式下，线程即使再阻塞当中被打断了，他还是会重新有进入到阻塞，仍然驻留在 AQS 的队列中，只有或得到锁之后才继续运行



- ### 可打断模式

```java
static final class NonfairSync extends Sync {
    public final void acquireInterruptibly(int arg) throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        // 如果没有获得到锁，进入（一）
        if (!tryAcquire(arg))
            doAcquireInterruptibly(arg);
    }
}

private void doAcquireInterruptibly(int arg) throws InterruptedException {
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt()) {
                // 在 park 过程中如果被 interrupt 会进入此
                // 这时候抛出异常，而不会再次进入 for (;;)
                throw new InterruptedException();
            }
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

当调用`doAcquireInterruptibly`时候，如果没有获得锁，进入park，如果被其他线程打断，这个时候会直接 throw new InterruptedException(); 抛出异常，而不会在进入循环等待。



### 公平锁实现原理

在非公平锁当中，线程一尝试加锁，就直接检查 state 是否等于0，如果等于0就直接尝试加锁，压根不会先进入AQS的对列等待。

公平锁的实现逻辑：和非公平锁的主要区别就在于 `tryAcquire`方法

tryAcquire 内部逻辑：

```java
protected final boolean tryAcquire(int acquires) {
    // 当前线程
    final Thread current = Thread.currentThread();
    // 获取AbstractQueuedSynchronizer的 state
    int c = getState();
    // state == 0 当前没有线程占用锁
    if (c == 0) {
        // 判断是否有线程在排队，若有线程在排队，返回true
        if (!hasQueuedPredecessors() &&
            // CAS抢锁
            compareAndSetState(0, acquires)) {
            // 抢锁成功，同时无线程排队，将AbstractQueuedSynchronizer的exclusiveOwnerThread设置为当前线程
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // state != 0  有线程占用锁
    // 判断占用锁的线程exclusiveOwnerThread是否为当前线程
    else if (current == getExclusiveOwnerThread()) {
        // state + 1，重入逻辑
        int nextc = c + acquires;
        // 锁重入超出最大限制 (int的最大值)，抛异常
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        // 将 state + 1 
        setState(nextc);
        // 当前线程拿到锁，返回true
        return true;
    }
    return false;
}
```

- 先检查 state 是否等于0，如果为0，代表此时还未有其他线程加锁

- 这个时候会再调用 `hasQueuedPredecessors`方法，他的内部逻辑是这样的

  ```java
  public final boolean hasQueuedPredecessors() {
      Node t = tail;
      Node h = head;
      Node s;
      // h != t 时表示队列中有 Node
      return h != t &&
          (
              // (s = h.next) == null 表示队列中还有没有老二
              (s = h.next) == null ||
              // 或者队列中老二线程不是此线程
              s.thread != Thread.currentThread()
          );
  }
  ```

  - 他会检查对列的 头 和 尾 是不是相等的，主要就是判断队列当中是已经有节点元素了
  - 如果不相等的话，检查队列中有没有第二个节点（也就是需要获取锁的节点），以及这个老二节点关联的线程是不是当前这个尝试加锁的线程
  - 只有都满足了，才能有资格去竞争锁



### 条件变量实现原理

当线程获取到锁之后，可能又需要其他线程给他提供一些资源，此时它可以调用 await 进入条件变量等待，而每一个条件变量实际上就是线程等待的时候进入的等待对列

其实现对应着 AQS 中的 ConditionObject，其实现了 Condition接口

![](juc并发编程 .assets/屏幕截图 2026-04-13 213339.png)



#### await 流程

刚开始 T-0 持有锁，但是其需要需要一些资源。

![](juc并发编程 .assets/屏幕截图 2026-04-13 135809-1776087846670-2.png)

所以 T-1线程调用了 await 方法进入等待，而await方法内部的逻辑是这样的：

```java
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    // 第一步
    Node node = addConditionWaiter();
    // 第二步
    int savedState = fullyRelease(node);
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        // 被 park 住
        LockSupport.park(blocker: this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}
```

- 第一步：此时又会进入到 ConditionObject 的 `addConditionWaiter()`方法内部，内部逻辑是这样的：

```java
private Node addConditionWaiter() {
    Node t = lastWaiter;
    // If lastWaiter is cancelled, clean out.
    if (t != null && t.waitStatus != Node.CONDITION) {
        unlinkCancelledWaiters();
        t = lastWaiter;
    }
    Node node = new Node(Thread.currentThread(), Node.CONDITION);
    if (t == null)
        firstWaiter = node;
    else
        t.nextWaiter = node;
    lastWaiter = node;
    return node;
}
```

- ConditionObeject 中为了保存等待线程，维护了一个等待队列，也是一个双向链表，有头节点 firstWaiter，尾节点 lastWaiter。
- `addConditionWaiter ` 会创建一个全新的 Node 节点，并且把当前进入等待的线程 T-0与之关联。
- 与加锁时创建阻塞队列之后创建节点不同的是，加锁的时候会额外创建一个哑元节点（即节点为null），但是条件变量中不会额外创建，只会创建一个与线程关联的Node节点
- 并且会把Node的waitStatus状态设置为 -2
- 紧接着会判断当前对列的头节点是否为空
  - 为空，则把当前这个新的Node节点加入到队列的头部
  - 不为空，则证明队列当中已经有节点了，则新的Node节点加入到队列尾部

![](juc并发编程 .assets/屏幕截图 2026-04-13 220432.png)

接下来就会进入到第二步 `fullyRelease(node)`:

```java
final int fullyRelease(Node node) {
    boolean failed = true;
    try {
        int savedState = getState();
        if (release(savedState)) {
            failed = false;
            return savedState;
        } else {
            throw new IllegalMonitorStateException();
        }
    } finally {
        if (failed)
            node.waitStatus = Node.CANCELLED;
    }
}
```



- 完全释放掉当前线程的锁，因为当前线程可能多次获取到了这把锁（也就是发生了锁重入）、
- 首先获取到 state 的值
- 之后一次性将 state 的值置为 0 
- 此时锁的持有者就为空了

![](juc并发编程 .assets/屏幕截图 2026-04-13 221700.png)

之后会调用 unpark 去唤醒AQS对列当中的下一个节点，去竞争锁，如果竞争成功，就将锁的持有者设置为T-1，state 又被重新设置为1，而且T-1关联的节点就会断开，等待被垃圾回收。

完成这些，T-1竞争锁成功

![](juc并发编程 .assets/屏幕截图 2026-04-13 222925.png)

之后 T-0 被 park 住

![](juc并发编程 .assets/屏幕截图 2026-04-13 222257-1776090599603-8.png)



#### signal 流程

T-0当前在条件便变量当中等待，此时锁的持有者唤醒T-1调用 signal 唤醒 T-0

![](juc并发编程 .assets/屏幕截图 2026-04-13 222925-1776091138039-11.png)

`signal`的逻辑是这样的:

```java
public final void signal() {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignal(first);
}
```

- 首先判断当前调用 signal 的线程是不是锁的**持有者**，只有是锁的持有者才有资格去唤醒等待线程
- 然后获取当前条件变量中的**第一个节点**，判断是否为空
- 不为空，之后调用`doSignal(first)`，逻辑是这样的

```java
private void doSignal(Node first) {
    do {
        if ((firstWaiter = first.nextWaiter) == null)
            lastWaiter = null;
        first.nextWaiter = null;
    } while (!transferForSignal(first) &&
             (first = firstWaiter) != null);
}
```

- 他会把当前 T-0 线程关联的节点从对列当中断开

![](juc并发编程 .assets/屏幕截图 2026-04-13 224515.png)

- 之后执行 `transferForSignal(first)`，这个方法的作用就是尝试将当前T-0关联的节点给他转移到阻塞对列当中，因为被唤醒的线程需要重新去竞争锁
- 他会尝试将 T-0关联的 Node 加入到AQS阻塞对列的尾部，并且将 T-0 的 waitStauts 设置为 0，因为阻塞队列尾部的节点 waitStauts  是0，那原先的T-3关联的节点就不再是尾节点了，所以 waitStauts  变为 -1

![](juc并发编程 .assets/屏幕截图 2026-04-13 225812.png)

之后线程T-1进入解锁流程。



## 3.ReentrantReadWriteLock 读写锁

如果某个场景是：对某个数据是读场景多，但是写场景少。

如果读或者写都是互斥进行的话，那并发性就非常低，因为读-读本身没有线程安全问题，多个线程读取同一个数据本身也没啥事，只要没有修改的话；只有当 读-写，或者 写-写时才有可能会出现线程安全问题。

**ReentrantReadWriteLock** 是 Java 并发包（java.util.concurrent.locks）中的一个类，它实现了一个可重入的读写锁。

它把锁分为两部分：读锁和写锁

读锁：就是当 **读-读**时，不加锁，多个线程可以并发的去读取同一个数据。

写锁：只有当 **读-写** 、**写-写**时，才会加锁，保证了同一时刻对同一个数据只能有一个线程获取到锁，从而互斥的访问该数据。

ReentrantReadWriteLock  本身实现了 ReadWriteLock 接口，ReadWriteLock 接口当中定义了两个抽象方法。

![](juc并发编程 .assets/屏幕截图 2026-04-14 143714.png)

### 使用方式

- #### 创建读写锁

```java
// 同通构造器创建读写锁
final ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();

// 通过读写锁获得读锁
final ReentrantReadWriteLock.ReadLock readLock = readWriteLock.readLock();

// 通过读写锁获得写锁
final ReentrantReadWriteLock.WriteLock writeLock = readWriteLock.writeLock();
```

- ### 使用

```java
// 配合try - finally 使用读写锁

// 1.读锁使用
readLock.lock();
try {
    // 业务代码...
} finally {
  readLock.unlock();
}

// 2.写锁使用
writeLock.lock();
try {
    // 业务代码...
} finally {
    writeLock.unlock();
}
```

- ### 测试 读-读：

```java
@Slf4j
public class ThreadTestDem13 {
    public static void main(String[] args) {
        ReadAndWriteObject readAndWriteObject = new ReadAndWriteObject();
        // 测试读
        new Thread(() -> {
            readAndWriteObject.read();
        },"t1").start();

        // 测试读
        new Thread(() -> {
            readAndWriteObject.read();
        },"t2").start();
    }
}

@Slf4j
class ReadAndWriteObject {
    // 创建读写锁
    private ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
    // 读锁
    private ReentrantReadWriteLock.ReadLock rLock = rwLock.readLock();
    // 写锁
    private ReentrantReadWriteLock.WriteLock wLock = rwLock.writeLock();
    // 共享变量
    private int value = 0;

    // 共享读方法
    public int read() {
        rLock.lock();
        try {
            log.debug("我获取到了读锁...,value:{}", value);
            Thread.sleep(1000);
            return value;
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            log.debug("我释放了读锁...");
            rLock.unlock();
        }
    }

    // 互斥写方法
    public void write() {
        wLock.lock();
        try {
            log.debug("我获取到了写锁...");
            this.value++;
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            log.debug("我释放了写锁...");
            wLock.unlock();
        }
    }
}
```

结果：可以看到，线程1、2同时获取到读锁，并没有受到阻塞，读-读不会被阻塞。

```java
2026-04-14 14:52:45.716 [t1] DEBUG JUC.JUCTest.Demo.D2.ReadAndWriteObject - 我获取到了读锁...,value:0
2026-04-14 14:52:45.716 [t2] DEBUG JUC.JUCTest.Demo.D2.ReadAndWriteObject - 我获取到了读锁...,value:0
2026-04-14 14:52:46.729 [t1] DEBUG JUC.JUCTest.Demo.D2.ReadAndWriteObject - 我释放了读锁...
2026-04-14 14:52:46.729 [t2] DEBUG JUC.JUCTest.Demo.D2.ReadAndWriteObject - 我释放了读锁...
```

- ### 测试写-写：

```java
@Slf4j
public class ThreadTestDem13 {
    public static void main(String[] args) {
        ReadAndWriteObject readAndWriteObject = new ReadAndWriteObject();
        // 测试读
        new Thread(() -> {
            readAndWriteObject.write();
        },"t1").start();

        // 测试读
        new Thread(() -> {
            readAndWriteObject.write();
        },"t2").start();
    }
}
2026-04-14 14:56:04.224 [t1] DEBUG JUC.JUCTest.Demo.D2.ReadAndWriteObject - 我获取到了写锁...
2026-04-14 14:56:04.225 [t1] DEBUG JUC.JUCTest.Demo.D2.ReadAndWriteObject - 更新value值：1
2026-04-14 14:56:05.236 [t1] DEBUG JUC.JUCTest.Demo.D2.ReadAndWriteObject - 我释放了写锁...
    
2026-04-14 14:56:05.236 [t2] DEBUG JUC.JUCTest.Demo.D2.ReadAndWriteObject - 我获取到了写锁...
2026-04-14 14:56:05.236 [t2] DEBUG JUC.JUCTest.Demo.D2.ReadAndWriteObject - 更新value值：2
2026-04-14 14:56:06.239 [t2] DEBUG JUC.JUCTest.Demo.D2.ReadAndWriteObject - 我释放了写锁...

```

可以看到线程t1先获取到写锁，完成了写操作，之后t2在获得到写锁，完成了写操作。

- ### 测试读-写：

```java
@Slf4j
public class ThreadTestDem13 {
    public static void main(String[] args) {
        ReadAndWriteObject readAndWriteObject = new ReadAndWriteObject();
        // 测试读
        new Thread(() -> {
            readAndWriteObject.read();
        },"t1").start();

        // 测试读
        new Thread(() -> {
            readAndWriteObject.write();
        },"t2").start();
    }
}
```

读-写也会加锁，可以看到线程t1先获取到读锁，读取数据，1s之后释放了读锁，之后线程2再获得写锁，对数据进行写操作

```java
2026-04-14 14:59:44.049 [t1] DEBUG JUC.JUCTest.Demo.D2.ReadAndWriteObject - 我获取到了读锁...,value:0
2026-04-14 14:59:45.065 [t1] DEBUG JUC.JUCTest.Demo.D2.ReadAndWriteObject - 我释放了读锁...

2026-04-14 14:59:45.065 [t2] DEBUG JUC.JUCTest.Demo.D2.ReadAndWriteObject - 我获取到了写锁...
2026-04-14 14:59:45.065 [t2] DEBUG JUC.JUCTest.Demo.D2.ReadAndWriteObject - 更新value值：1
2026-04-14 14:59:46.075 [t2] DEBUG JUC.JUCTest.Demo.D2.ReadAndWriteObject - 我释放了写锁...
```



### 注意事项

- 读锁不支持条件变量，这个很好理解，因为读-读的时候压根不会被阻塞，所以不需要条件变量；写锁支持条件变量
- 发生锁**重入**时候，**不支持锁升级**：当一个线程已经获取到了读锁，但是他又想获取写锁的话，就会永久等待获取写锁
- 但是锁**重入**的时候，**支持锁降级**：就是在已经获取了写锁的情况下，还能获取读锁



### ReentrantReadWriteLock 实现原理

### 加锁

读锁，写锁都使用 ReentrantReadWriteLock  中的同一个同步器 Sync，Sync 也是基于AQS实现的，因此他们的等待队列， state 资源状态也都是用的同一个。

<img src="juc并发编程 .assets/屏幕截图 2026-04-14 172057.png" style="zoom:67%;" />



- ### 场景一：t1加写锁 w.lock( )，t2加读锁 r.lock( )


此时没有竞争，t1就能成功的加写锁

![](juc并发编程 .assets/屏幕截图 2026-04-14 172955.png)

这个加锁的过程与 ReentrantLock 加锁的过程非常相似，区别在于资源状态 state 有所不同：

- ReentrantReadWriteLock  将 state 分成了两部分供读锁、写锁使用：写锁占用了 state 的低16位，读锁占用了 state 的高16位。同样 0 是未加锁，1是加锁，>1可能是锁重入，也可能是其他线程加了读锁。

执行逻辑：底层调用 tryAcquire

```java
@ReservedStackAccess
protected final boolean tryAcquire(int acquires) {
    Thread current = Thread.currentThread();
    int c = getState();
    int w = exclusiveCount(c);
    if (c != 0) {
        // (Note: if c != 0 and w == 0 then shared count != 0)
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        // Reentrant acquire
        setState(c + acquires);
        return true;
    }
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```

- 首先判断 state 等不等于0

  - 如果等于0，就代表当前没有任何线程加锁，之后会调用 `writerShouldBlock`方法，会判断线程获取锁的时候应不应该受到阻塞（即公平与非公平），因为此时使用的是非公平锁，所以不会受到阻塞，就会返回false，之后会就调用 `compareAndSetState`执行CAS操作，将state从0改成1，都成功了，就会将 exclusiveOwnerThread 设置为 t1 线程

  - 如果不等于0，那就分成两种情况了

    - 可能是加了写锁
    - 也可能是加了读锁

  - 这时会先获取**写锁**部分 w

    - 如果 **w=0**，但是又因为 state !=0，state 又由读锁和写锁共同组成，所以此时已经加的就是**读锁**，但是由于读锁和写锁是互斥的，不能先获得读锁再获得写锁，此时会返回false。

    - 如果 w !=0，就代表已经有线程加了写锁了，这时当前候线程再想加写锁，就得判断一下，写锁的持有者是不是当前线程，因为可能会发生锁重入，此时调用会`getExclusiveOwnerThread`得到锁的持有者，如果不是，也返回false

  - 如果都满足，就调用`setState(c + acquires)`将state+1，代表重入次数+1.

    

- ### T1此时加写锁成功，之后轮到T2加读锁：

调用 r.lock( )，会调用同步器 sync 的 `acquireShared `方法，`acquireShared` 又会调用 `tryAcquireShared`方法，这个方法会返回一个整数

```java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```

- 如果返回 -1，代表获取读锁失败
- 如果返回 0或者正数，都代表获取成功，但是有所不同
  - 0 成功，但是后继节点不会被唤醒
  - 正数 成功，是几，就代表有几个后继节点需要唤醒，但是读写锁比较简单，就会返回1

tryAcquireShared 的执行逻辑：

```java
@ReservedStackAccess
protected final int tryAcquireShared(int unused) {
    Thread current = Thread.currentThread();
    int c = getState();
    if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current)
        return -1;
    int r = sharedCount(c);
    if (!readerShouldBlock() &&
        r < MAX_COUNT &&
        compareAndSetState(c, c + SHARED_UNIT)) {
        if (r == 0) {
            firstReader = current;
            firstReaderHoldCount = 1;
        } else if (firstReader == current) {
            firstReaderHoldCount++;
        } else {
            HoldCounter rh = cachedHoldCounter;
            if (rh == null ||
                rh.tid != LockSupport.getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
    return fullTryAcquireShared(current);
}
```

先获取当前线程、state 状态

- 之后调用 `exclusiveCount(c)`，判断写锁部分是否为0，判断别人是不是加了写锁
  - 如果不为0，就是已经加了写锁了，之后会执行`getExclusiveOwnerThread() != current`，判断这个加写锁的线程是不是他自己，因为可能会出现锁降级的情况，也就是线程先获取了写锁，又想获取读锁。
  - 但是现在的情况是，加写锁的是线程 T1，T2获取读锁就失败了，所以这个方法就会返回 -1

![](juc并发编程 .assets/屏幕截图 2026-04-14 183310.png)

此时  tryAcquireShared 返回-1，就进入了 `doAcquireShared`方法内部：

```java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```

```java
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

- 首先也是调用`addWaiter(Node.SHARED)`向阻塞队列中添加节点，和ReentrantLock不同的是，此时的节点状态被设置为 Node.SHARED 模式，而不是 Node.EXCLUSIVE，并且将线程T-2与Node节点相关联，之后添加到队列当中，而且线程T-2此时仍处于活跃状态。、

![](juc并发编程 .assets/屏幕截图 2026-04-14 214003.png)

- 接下来就是一个死循环，一进来先获取他的前驱节点，并判断是不是头节点，如果是，证明自己是老二节点，那他仍然有资格去竞争锁，所以再次调用了`tryAcquireShared(arg)`方法尝试获取锁

- 如果还是没成功，tryAcquireShared就会返回-1，if (r >= 0) 这一段逻辑就不会执行，而是进入

  ```java
  if (shouldParkAfterFailedAcquire(p, node) &&
                  parkAndCheckInterrupt())
                  interrupted = true;
  ```

- 先执行`shouldParkAfterFailedAcquire(p, node)`，他会把T-2节点的前驱节点的 waitStatus 状态设置为-1，也就是他有资格唤醒他的后继节点，但是此时 T-2 线程还是不会被 park 住，因为第一次调用 shouldParkAfterFailedAcquire 返回的是 false，所以不会执行后面的  ` parkAndCheckInterrupt()`

- 接下来，他会在进行一次循环，如果还不成功，此时 shouldParkAfterFailedAcquire 就会返回true，那就真的会调用后面的 `parkAndCheckInterrupt`，将T-2 park 住

![](juc并发编程 .assets/屏幕截图 2026-04-14 220026.png)



- ### 后续：又来了 T-3线程加读锁，T-4加写锁

因为此时T-2仍然持有写锁，所以T-3，T-4都会被阻塞住，并且因为T-3是加读锁，所以他是SHARED模式，T-4加的是写锁，所以他是EXCLUSIVE

![](juc并发编程 .assets/屏幕截图 2026-04-14 221127.png)



### 解锁

- ### 写锁释放

此时t1 调用 w.unlock( )解锁

```java
public void unlock() {
    // 先调用 sync 的release
    sync.release(1);
}

public final boolean release(int arg) {
    // 之后调用 tryRelease
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}

@ReservedStackAccess
protected final boolean tryRelease(int releases) {
    // 1. 检查：当前线程是否持有锁
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    // 2. 计算新的同步状态（锁计数 - 释放量）
    int nextc = getState() - releases;
    // 3. 判断锁是否完全释放（可重入计数 == 0）
    boolean free = exclusiveCount(nextc) == 0;
    // 4. 如果锁完全释放，清空持有锁的线程
    if (free)
        setExclusiveOwnerThread(null);
    // 5. 更新同步状态
    setState(nextc);
    // 6. 返回：true=锁已完全释放，false=还持有锁（重入未放完）
    return free;
}
```

- 首先判断持锁线程是不是当前线程
- 如果是，就将state的值减1
- 之后会去检查，写锁部分是不是已经减为0了，因为之前可能会有锁重入
- 如果为0了，那就释放干净了，会将锁的持有者 exclusiveOwnerThread 置为空，之后返回true

![](juc并发编程 .assets/屏幕截图 2026-04-14 222822.png)

- 接下来就执行 if 段中的逻辑

```java
public final boolean release(int arg) {
    // 之后调用 tryRelease
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

- 获取头节点，判断是不是为空，检验状态是不是不等于0，都满足，就开始唤醒他的后继节点，进入` unparkSuccessor(h)`
- T-2线程是在`doAcquireShared`方法内的` parkAndCheckInterrupt()`处被park住的，所以此时会被唤醒，在此处恢复运行
- 之后会再来一次循环，执行`tryAcquireShared`尝试加锁了，如果成功，则计数+1

<img src="juc并发编程 .assets/屏幕截图 2026-04-14 223408.png" style="zoom: 50%;" />

![](juc并发编程 .assets/屏幕截图 2026-04-14 224717.png)

- 这是线程T-2已经恢复运行，接下来就会执行if段中的逻辑，调用`setHeadAndPropagate(node, r)`:他会把原先的头节点从队列当中断开，并且把T-2线程关联的节点置为头节点

<img src="juc并发编程 .assets/屏幕截图 2026-04-14 223408-1776178296727-7.png" style="zoom:50%;" />

![](juc并发编程 .assets/屏幕截图 2026-04-14 225253.png)

但是到现在还没完，`setHeadAndPropagate`方法内部还会检查当前节点的下一个节点，也就是T-3节点是不是 Shared 模式

- 如果是，就代表他要加读锁，而读锁是可以共享获取的，就会调用另一个方法 `doReleaseShared`方法，内部逻辑是这样的：

<img src="juc并发编程 .assets/屏幕截图 2026-04-14 230412.png" style="zoom:50%;" />



- 他会尝试将当前头节点head的**状态从-1改为0**，这一步主要是为了防止在改动的过程中其他线程的干扰

- 之后他还会接着唤醒老二节点，也就是T-3节点，此时T-3也会在`doAcquireShared`方法内的` parkAndCheckInterrupt()`处被唤醒，然后恢复运行

![](juc并发编程 .assets/屏幕截图 2026-04-14 230922.png)

同样是这个：

<img src="juc并发编程 .assets/屏幕截图 2026-04-14 223408-1776180055722-12.png" style="zoom:50%;" />

- 之后又来一遍循环，调用`tryAcquireShared`尝试加锁，肯定能成功，就会让读锁计数+1
- 这就是读锁的不同之处，他允许多个线程对state进行操作

![](juc并发编程 .assets/屏幕截图 2026-04-14 232450.png)

此时T-3线程已经正常恢复运行，接下来接着调用`setHeadAndPropagate`，将T-3节点设置为头节点，将原来的T2的那个头节点断开

![](juc并发编程 .assets/屏幕截图 2026-04-14 233019.png)

但是下一个T-4节点已经不是Shared模式了，而是独占模式，所以不会唤醒T-4线程。

**这就是为什么读-读能够实现并发的原因**，因为只要是shared模式的节点，在唤醒的时候，一唤醒就连着唤醒一长串，直到遇到独占模式的节点。



- ### 读锁释放

T-2调用 r.unlock（ ）,T-3调用 r.unlock（ ）释放读锁。

先调用 sync 的`releaseShared`，之后又调用 `releaseShared`

```java
public void unlock() {
    // 调用同步器 sync 的 releaseShared
    sync.releaseShared(1);
}

public final boolean releaseShared(int arg) {
    // 调用 tryReleaseShared
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```

tryReleaseShared 的逻辑是这样的：

主要是最后一段循环：

<img src="juc并发编程 .assets/屏幕截图 2026-04-14 234812.png" style="zoom: 67%;" />

- 先获取 state 的值，然后使用 CAS 操作让state的读锁部分值减1，如果减为0了，就返回true，否则返回false
- T2 先进入` sync.releaseShared(1);`中，之后调用 tryReleaseShared ，将state的读锁部分减1，但是由于计数还不为0，因为此时T-3还加着读锁，就会返回false

![](juc并发编程 .assets/屏幕截图 2026-04-15 174924.png)

- 之后T-3也调用tryReleaseShared ，将state的计数减去1，此时计数为0了，返回true，就会进入到`doReleaseShared();`

![](juc并发编程 .assets/屏幕截图 2026-04-15 175421.png)

- 也是会将头节点的状态从-1改为0，并唤醒他的后继节点，也就是老二节点

![](juc并发编程 .assets/屏幕截图 2026-04-15 180231.png)

- 之后T-4线程就会再 accquiredQueue被唤醒，并且进行下一次循环，再次尝试获取写锁，因为此时已经没有竞争了，所以肯定能够获得。之后也是将锁的持有者设置为 T-4，并且修改 state 的状态

<img src="juc并发编程 .assets/屏幕截图 2026-04-15 180756.png" style="zoom:50%;" />

- 然后也是，将原来的头节点分离，让T-4节点成为新的头节点

![](juc并发编程 .assets/屏幕截图 2026-04-15 181623.png)



## 4.StampedLock 印章锁

`ReadWriteLock`，会发现它有个潜在的问题：如果有线程正在读，写线程需要等待读线程释放锁后才能获取写锁，即读的过程中不允许写，这是一种 悲观读锁，而且也需要频繁的使用CAS操作去更新state的状态，虽然能够实现并发读，但是性能还是不够极致，显然还是比不上不加锁。

StampedLock 对此进行了改进，进一步优化了读的性能。

- 它的特点是：获取读锁之后，还允许获取写锁，之后写入数据。

但这就有一个问题，如果读锁和写锁都能同时获取，那就能对数据同时进行读和写了，肯定会有线程安全问题了，但是 StampedLock 使用了一点额外的手段，来判断读的过程中，是否有写入，这种读锁是一种乐观读锁。

StampedLock 也是适用于读多写少的场景。

- 它的特点是读锁、写锁都需要配合【戳】使用

StampedLock的缺点：

- 不支持条件变量
- 不支持锁重入，自己能给自己挡在外面



### 使用

ReadWriteLock 是提供了两种锁：读锁，写锁

而StampedLock提供了**三种模式的锁**：

- **写锁**：

  与 ReadWriteLock  一样，是独占锁，同一时刻只允许一个线程持有写锁，其他线程无论是加读锁、写锁，都加不了

  ```java
  // 加、解 写锁
  // 返回一个戳
  long stamp = lock.writeLock();
  lock.unlockWrite(stamp);
  ```

- **读锁**：

  与 ReadWriteLock  一样，是悲观读锁，允许多个线程获取读锁，但是获取不了写锁，想获取，就会阻塞

  ```java
  // 加、解读锁
  // 返回一个戳
  long stamp = lock.readLock();
  lock.unlockRead(stamp);
  ```

- **乐观读锁**：

  StampedLock提供了一个`tryOptimisticRead`方法，这是乐观读，读数据的时候不会真加锁。而是通过验证数据在读取期间是否被修改来保证一致性。

  调用 `tryOptimisticRead`也会返回一个戳，之后通过`validate`方法对戳进行`戳校验`

  - 如果通过，代表戳没变，这期间没有线程进行写操作，则可以安全的读数据；

  - 但是如果校验没通过，则证明戳更新了，数据被修改了，这时候会进行锁升级，**乐观读锁**会升级为**读锁**，从而阻塞写锁的获取，保证数据的安全。

  ```java
  long stamp = lock.tryOptimisticRead();
  // 验戳
  if(!lock.validate(stamp)){
      // 锁升级
  }
  ```

  

测试：

read 方法：线程先乐观读，如果不成功，升级为被关渡

write 方法：加写锁，写数据

```java
@Slf4j
class StampedLockTest {
    private int data;
    private final StampedLock lock = new StampedLock();

    public StampedLockTest(int data) {
        this.data = data;
    }

    // 读
    public int read(int readTime) {
        // 模式一：乐观读
        // 加乐观读锁
        long stamp = lock.tryOptimisticRead();
        log.debug("乐观读 ing...，戳：{}", stamp);
        // 读...
        try {
            Thread.sleep(readTime * 1000L);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        // 戳校验
        if (lock.validate(stamp)) {
            log.debug("乐观读 end...，戳：{}", stamp);
            // 乐观读成功，返回数据
            return data;
        }

        // 模式二：悲观读
        // 如果加盐失败，升级为悲观读锁
        log.debug("乐观读 ---> 悲观读...");
        long readStamp = lock.readLock();
        try {
            log.debug("悲观读 ing...，戳：{}", readStamp);
            // 读...
            try {
                Thread.sleep(readTime * 1000L);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug("悲观读 end...，戳：{}", readStamp);
            // 悲观读成功，返回数据
            return data;
        } finally {
            lock.unlock(readStamp);
        }
    }

    // 写
    public void write(int newData) {
        long stamp = lock.writeLock();
        log.debug("写 ing...，戳：{}", stamp);
        try {
            try {
                Thread.sleep(2000L);
                this.data = newData;
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        } finally {
            log.debug("写 end...，戳：{}", stamp);
            lock.unlock(stamp);
        }
    }
}

```

测试：乐观读

```java
@Slf4j
public class ThreadTestDem14 {
    public static void main(String[] args) throws InterruptedException {
        StampedLockTest test = new StampedLockTest(1);
        new Thread(() -> {
            // 读
            int data = test.read(1);
        },"t1").start();

        Thread.sleep(500);

        new Thread(() -> {
            // 读
            int data = test.read(0);
        },"t2").start();
    }
}
```

结果：

```cmd
2026-04-15 22:52:23.093 [t1] DEBUG JUC.JUCTest.Demo.D2.StampedLockTest - 乐观读 ing...，戳：256
2026-04-15 22:52:23.602 [t2] DEBUG JUC.JUCTest.Demo.D2.StampedLockTest - 乐观读 ing...，戳：256
2026-04-15 22:52:23.602 [t2] DEBUG JUC.JUCTest.Demo.D2.StampedLockTest - 乐观读 end...，戳：256
2026-04-15 22:52:24.106 [t1] DEBUG JUC.JUCTest.Demo.D2.StampedLockTest - 乐观读 end...，戳：256
```

可以看到 0.5s之后，t2执行，也是能过乐观读，t1，t2都没有悲观读，t2之后就执行完了，也没有受到t1阻塞。而且戳也都没变。

测试：写+悲观读

```java
@Slf4j
public class ThreadTestDem14 {
    public static void main(String[] args) throws InterruptedException {
        StampedLockTest test = new StampedLockTest(1);
        new Thread(() -> {
            // 读
           test.read(1);
        },"t1").start();

        Thread.sleep(500);

        new Thread(() -> {
            // 写
            test.write(2);
        },"t2").start();
    }
}

```

```cmd
2026-04-15 23:01:43.017 [t1] DEBUG JUC.JUCTest.Demo.D2.StampedLockTest - 乐观读 ing...，戳：256
2026-04-15 23:01:43.526 [t2] DEBUG JUC.JUCTest.Demo.D2.StampedLockTest - 写 ing...，戳：384
2026-04-15 23:01:44.026 [t1] DEBUG JUC.JUCTest.Demo.D2.StampedLockTest - 乐观读 ---> 悲观读...
2026-04-15 23:01:45.536 [t2] DEBUG JUC.JUCTest.Demo.D2.StampedLockTest - 写 end...，戳：384
2026-04-15 23:01:45.536 [t1] DEBUG JUC.JUCTest.Demo.D2.StampedLockTest - 悲观读 ing...，戳：513
2026-04-15 23:01:46.550 [t1] DEBUG JUC.JUCTest.Demo.D2.StampedLockTest - 悲观读 end...，戳：513
```

可以看到，t1先是乐观读，但是在读的过程中，t2线程要进行写操作，加了写锁，而且也没有被阻塞，之后写了数据，并且更新了戳。

之后t1读完数据了，就进行戳校验，但是此时戳已经被更新了，校验就失败了。于是锁升级为读锁，悲观读，并且也是更新了戳。



## 5.Semaphore 信号量

### 概念

Semaphore 信号量 ，用来限制能同时访问共享资源的线程的上限。

ReentrantLock 本质上独占锁，同一时刻只能有一个线程访问共享资源。

Semaphore **适用于有多个共享资源，也允许有多个线程访问这些资源，但是希望对这些线程数量加以上限。**

例子：

停车场中有很多车位，也允许多个车辆去停车，但是肯定不能无上限的停啊，那问题就来了，新来的车他怎么知道还有没有车位停车呢？

可以这样：在入口处放置一个告示牌或者显示牌，牌子上会显示剩余车位，进入一辆车就+1，出去就-1，那新来的车只需要看一眼告示牌，就能判断还能不能再进去停车，有剩余车位，就能停车；没有，只能等有车出去他才能进入。



### 使用

- 创建 Semaphore

```java
Semaphore 提供了两种构造方法
// 创建具有指定许可证数量的 Semaphore，，也就是线程的上限，这是非公平模式。
new Semaphore(int permits)
    
// 第二个参数为布尔值，指示是否使用公平模式。
new Semaphore(int permits, boolean fair)
```

- 常用方法

```java
// 获取一个许可证，许可数量减1，若无可用许可证，则线程进入等待。
void acquire() throws InterruptedException: 获取一个许可证，若无可用许可证，则线程进入等待。
    
// 归还一个许可证，许可数量加1
void release()
    
// 返回当前可用的许可证数量。
int availablePermits()
    
// 尝试获取一个许可证，若获取成功则返回 true，否则返回 false。
boolean tryAcquire()
```



测试：

```java
@Slf4j
public class ThreadTestDem15 {
    public static void main(String[] args) {
        // 创建信号量，许可数量为3
        Semaphore semaphore = new Semaphore(3);

        for (int i = 0; i < 6; i++) {
            int count = i;
            new Thread(() -> {
                // 获得许可
                try {
                    semaphore.acquire();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                try {
                    // 运行1s
                    log.debug("running...");
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                } finally {
                    // 释放许可
                    log.debug("end...");
                    semaphore.release();
                }
            },"t" + count).start();
        }
    }
}
```

结果：

```cmd
2026-04-16 16:51:30.240 [t1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem15 - running...
2026-04-16 16:51:30.240 [t0] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem15 - running...
2026-04-16 16:51:30.240 [t2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem15 - running...
2026-04-16 16:51:31.253 [t2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem15 - end...
2026-04-16 16:51:31.253 [t0] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem15 - end...
2026-04-16 16:51:31.253 [t1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem15 - end...
2026-04-16 16:51:31.254 [t3] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem15 - running...
2026-04-16 16:51:31.254 [t5] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem15 - running...
2026-04-16 16:51:31.254 [t4] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem15 - running...
2026-04-16 16:51:32.255 [t3] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem15 - end...
2026-04-16 16:51:32.261 [t4] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem15 - end...
2026-04-16 16:51:32.261 [t5] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem15 - end...
```

因为只有三个许可，所以只有前三个线程获取到了许可，并且执行了。1s之后释放了许可，此时有剩余的许可了，后面三个线程也获得了许可，去执行。



### 应用

Semaphore 可以做限流，在高峰期的时候，对线程进行限制阻塞，当高峰期过后，在释放许可。

但是Semaphore限制的是线程数量，而不是资源数量。

可以用 Semaphore  来改进数据库连接池。之前使用享元模式实现过一个数据库连接池，当时使用的是 wait - notify 机制，来实现的线程的阻塞与唤醒。

<img src="juc并发编程 .assets/屏幕截图 2026-04-16 170810.png" style="zoom:67%;" />

接下来是改进借链接，与归还链接

```java
// 借连接
    public Connection borrow() {
        // 获取许可链接
        try {
            semaphore.acquire(); // 未获得许可的线程会在此处等待
            for (int i = 0; i < poolSize; i++) {
                int prev = states.get(i);
                if (prev == this.RELAXED) {
                    boolean success = states.compareAndSet(i, prev, RUNNING);
                    if (success) {
                        log.debug("借出连接:{}", connections[i]);
                        return connections[i];
                    }
                }
            }
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        return null;
    }

    // 归还连接
    public void release(Connection conn) {
        for (int i = 0; i < poolSize; i++) {
            if (connections[i] == conn) {
                states.set(i, RELAXED);
                log.debug("归还链接:{}", conn);
                // 归还许可
                semaphore.release();
                break;
            }
        }
    }
```



### Semaphore 实现原理

Semaphore 内部的同步器 Sync 也是基于AQS实现的。

当调用 Semaphore  的构造方法，传入许可数量 permits 的创建 Semaphore 对象的时候，底层最终会调用 Sync 同步器中的构造方法，该构造方法会把 permits  传给`setState(permits)`方法，该方法就是 AQS 中的方法，就是把 permits 赋值给AQS中的 state 。

![](juc并发编程 .assets/屏幕截图 2026-04-16 172225.png)

- 初始，permits（state）为3，但是此时有 5 个线程来获取资源。

![](juc并发编程 .assets/屏幕截图 2026-04-16 172024.png)

#### acquire 

- 当调用 Semaphore 的 acquire 方法获取许可的时候，最终会调用`nonfairTryAcquireShared`方法：内部逻辑是这样的

```java
// 非公平方式尝试获取【共享锁】，final修饰不可重写
final int nonfairTryAcquireShared(int acquires) {
    // 自旋（死循环），直到获取成功或返回失败
    for (;;) {
        // 1. 获取AQS同步状态state的值（代表当前可用的共享资源数）
        int available = getState();
        // 2. 计算获取后剩余的可用资源数
        int remaining = available - acquires;
        // 3. 核心判断：
        // ① remaining < 0：资源不足，获取失败，直接返回负数
        // ② CAS更新state：从旧值available → 新值remaining，更新成功则获取成功
        if (remaining < 0 || compareAndSetState(available, remaining))
            return remaining;
    }
}
```

- 首先会获取 state 的初始值，比如现在是3，之后会计算，获取许可之后的剩余许可数
- 如果剩余许可数 > 0，就代表许可数充足，就会使用 CAS 操作将 state 更新，然后返回剩余的许可数，这就竞争成功了
- 比如 t1，t2，t4都竞争成功了
- 但是之后 t0，t3也竞争，但是此时 state 已经为0了，remaining = 0 - 1，为负数了，那 if 段中的 remaining < 0 就成立了，就不会执行后续的 CAS 操作了 ，就直接把 -1 返回了。
- 而 t0、t3就竞争失败了，进入到 AQS 的阻塞队列当中被 park 住。

![](juc并发编程 .assets/屏幕截图 2026-04-16 174859.png)



#### release

- 当调用Semaphore 的 release方法释放许可的时候，最终会调用 `tryReleaseShared`方法：内部逻辑是这样的：

```java
// 释放共享锁/共享资源，protected final 不能被重写
protected final boolean tryReleaseShared(int releases) {
    // 自旋：CAS 可能失败，必须循环重试直到成功
    for (;;) {
        // 1. 获取当前同步状态 state（当前可用的共享资源数）
        int current = getState();
        
        // 2. 计算释放后的新资源数：释放 = 归还资源，所以是加法
        int next = current + releases;
        
        // 3. 防溢出判断：int 溢出会变成负数，next < current 说明溢出了
        if (next < current)
            throw new Error("Maximum permit count exceeded");
        
        // 4. CAS 原子更新 state：从 current → next
        // 成功：释放成功；失败：说明其他线程也在修改，自旋重试
        if (compareAndSetState(current, next))
            return true;
    }
}
```

- 首先也是先获取 state 的值，之后计算释放一个许可之后的资源数，然后使用 CAS 操作更新 state 的值
- 比如 t4 已经释放了一个许可

![](juc并发编程 .assets/屏幕截图 2026-04-16 175216.png)

- 那接下来 t0 就能竞争成功，将 state 再次设置为 0 ，并且把自己关联的节点设置为头节点，断开原来的头节点，并且 unpark 唤醒它的后继节点 t3，接下来 t3会被唤醒，再次尝试获取许可，但是此时 state 的值还是为0，所以尝试一次之后，便又被 park 住



## 6.CountDownLatch

### 概念

CountDownLatch 是一个同步工具类，用来协调多个线程之间的同步，它能够让一个线程去等另外一些线程完成各自的工作之后，再继续执行。

核心原理如下： 

- **计数器机制**：创建时指定一个正整数作为计数器初始值，代表需要等待完成的事件或任务数量
- **等待机制**：调用`await()`方法的线程会被阻塞，直到计数器减到零
- **递减机制**：其他线程完成任务后调用`countDown()`方法使计数器减一
- **唤醒机制**：当计数器减至零时，所有等待的线程会被唤醒

CountDownLatch 使用一个**计数器**来实现同步效果，计数器的**初始值**是**线程的数量**，每当一个线程完成自己的任务之后，计数器的值就会减1，直到计数器的值为0，表示所有的线程都完成了自己的一些任务，那在 CountDownLatch  上等待的线程就能继续执行自己接下来的任务。

CountDownLatch 底层也是通过 AQS 实现的，内部维护了一个 Sync 同步器，继承自 AQS，state 的数值为线程数，通过 CAS 操作来保证计数器递减时候的安全性。

<img src="juc并发编程 .assets/屏幕截图 2026-04-16 214449.png" style="zoom:67%;" />

特点：

- 一次性使用：当计数器被减到0时，无法重置，无法被重用
- 非独占锁：所有线程共享计数器的 state 状态，不存在锁竞争问题。



### 使用

1. **构造方法**

```java
// 初始化计数器，count必须为非负整数。若为0，await()会立即返回
public CountDownLatch(int count)
```

2. **await( )**

```java
// 调用 await() 方法的线程会被阻塞，直到计数器减为 0 
public void await() throws InterruptedException
```

   **3. await(long timeout, TimeUnit unit)**

```java
// 带超时的等待，在指定时间内计数器未归零则返回false
public boolean await(long timeout, TimeUnit unit) throws InterruptedException
```

4. **countDown( )**

```java
// 将计数器减一，当减至零时唤醒所有等待线程
public void countDown()
```

  5. **getCount()**

```java
// 获取当前计数器的值（较少使用）
public long getCount()
```



测试：

主线程需要等待三个线程都执行完毕，再继续执行

```java
@Slf4j
public class ThreadTestDem16 {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(3);
        new Thread(() -> {
            log.debug("running...");
            try {
                Thread.sleep(1000L);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug("end...");
            latch.countDown();
        },"t1").start();

        new Thread(() -> {
            log.debug("running...");
            try {
                Thread.sleep(2000L);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug("end...");
            latch.countDown();
        },"t2").start();

        new Thread(() -> {
            log.debug("running...");
            try {
                Thread.sleep(3000L);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug("end...");
            latch.countDown();
        },"t3").start();

        log.debug("主线程 waiting...");
        latch.await();
        log.debug("主线程 continue...");
    }
}

```

结果：

```cmd
2026-04-16 22:09:30.236 [t1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - running...
2026-04-16 22:09:30.236 [t3] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - running...
2026-04-16 22:09:30.236 [t2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - running...
2026-04-16 22:09:30.236 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - 主线程 waiting...
2026-04-16 22:09:31.247 [t1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - end...
2026-04-16 22:09:32.250 [t2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - end...
2026-04-16 22:09:33.239 [t3] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - end...
2026-04-16 22:09:33.239 [main] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - 主线程 continue...
```

可以看到，3s之后，三个线程都执行完毕了，将计数器减为0了，主线程才恢复运行。

### 改进

可以配合线程池使用：

```java
@Slf4j
public class ThreadTestDem16 {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService pool = Executors.newFixedThreadPool(4);
        CountDownLatch latch = new CountDownLatch(3);

        pool.execute(() -> {
            log.debug("running...");
            try {
                Thread.sleep(1000L);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug("end...");
            latch.countDown();
        });

        pool.execute(() -> {
            log.debug("running...");
            try {
                Thread.sleep(2000L);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug("end...");
            latch.countDown();
        });

        pool.execute(() -> {
            log.debug("running...");
            try {
                Thread.sleep(3000L);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug("end...");
            latch.countDown();
        });

        pool.execute(() -> {
           log.debug("waiting...");
            try {
                latch.await();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug("continue...");
        });
    }
}
```

```cmd
2026-04-16 22:29:39.843 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - running...
2026-04-16 22:29:39.843 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - running...
2026-04-16 22:29:39.843 [pool-1-thread-4] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - waiting...
2026-04-16 22:29:39.843 [pool-1-thread-3] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - running...
2026-04-16 22:29:40.859 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - end...
2026-04-16 22:29:41.860 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - end...
2026-04-16 22:29:42.859 [pool-1-thread-3] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - end...
2026-04-16 22:29:42.860 [pool-1-thread-4] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - continue...
```



### 应用

- ### 主线程等待子任务完成

主线程在开始运行前等待n个线程执行完毕。

将CountDownLatch的计数器初始化为new CountDownLatch(n)。

每当一个任务线程执行完毕，就将计数器减1 countdownLatch.countDown()，当计数器的值变为0时，在CountDownLatch上await( )的线程就会被唤醒。最常见的场景是主线程需要等待多个子线程完成任务后再继续执行。例如系统启动时需要等待多个组件初始化完成。

```java
@Slf4j
public class ThreadTestDem16 {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService service = Executors.newFixedThreadPool(10);
        CountDownLatch latch = new CountDownLatch(10);
        Random r = new Random();
        String[] all = new String[10];
        for (int j = 0; j < 10; j++) {
            int k = j;
            service.submit(() -> {
                for (int i = 0; i <= 100; i++) {
                    try {
                        Thread.sleep(r.nextInt(100));
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    all[k] = i + "%";
                    System.out.print("\r" + Arrays.toString(all));
                }
                latch.countDown();
            });
        }
        latch.await();
        System.out.println("\n游戏开始");
        service.shutdown();
    }
}
```

10个线程分别执行自己的任务，主线程调用 `await`等待其他线程执行完毕。每个线程执行完任务之后，调用 `countDown`将计数器减1，都执行完毕之后，计数器减为0，主线程再继续执行。





## 7.cylicbarrier

假设现在有这样一个场景，有三个线程，分别执行三个任务，主线程也是需要等待他们各自执行完成之后，进行汇总，但是现在这三个线程需要循环执行三次，也就是说，每执行完一轮，主线程就汇总一次，可以使用 CountDownLatch 让主线程等待他们执行完。

但是由于 CountDownLatch 在使用上是一次性的，当三个线程完成一轮循环之后，CountDownLatch 的计数器就减为0了，这时候这个计数器就没办法使用了，只能再创建一个新的 CountDownLatch ，循环三轮，就相应的得创建三个，这就太麻烦了。

```java
@Slf4j
public class ThreadTestDem16 {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService pool = Executors.newFixedThreadPool(3);
        CountDownLatch latch = new CountDownLatch(3);

        // 循环3轮
        for (int i = 0; i < 3; i++) {
            pool.execute(() -> {
                log.debug("running...");
                try {
                    Thread.sleep(1000L);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                log.debug("end...");
                latch.countDown();
            });

            pool.execute(() -> {
                log.debug("running...");
                try {
                    Thread.sleep(2000L);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                log.debug("end...");
                latch.countDown();
            });

            pool.execute(() -> {
                log.debug("running...");
                try {
                    Thread.sleep(3000L);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                log.debug("end...");
                latch.countDown();
            });

            log.debug("waiting...");
            latch.await();
        }
    }
}
```

这样每次循环都要创建一个新的 CountDownLatch，循环三次，相当于创建了三个。因为 CountDownLatch 没办法重用



### 概念

cylicbarrier 循环栅栏，与 CountDownLatch 一样，都是用来进行线程之间协作的同步工具类，可以通过构造方法指定计数个数。允许一组线程互相等待，确保直到所有线程都达到预定屏障点后才能被放行继续执行后续的操作。

<img src="juc并发编程 .assets/屏幕截图 2026-04-17 171939-1776417649108-3.png" style="zoom:50%;" />

与 CountDownLatch 不同的是，cylicbarrier 支持屏障的重复使用，栅栏可以在使用完之后，恢复到初始值，以便于下次使用，这样就无需像 CountDownLatch  一样重复创建新的 CountDownLatch  对象了，非常适合多轮次的任务同步。

![](juc并发编程 .assets/24614072870d4c618120522dc6bafe33tplv-73owjymdk6-jj-mark-v1_0_0_0_0_5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5byC5bi45ZCb_q75.webp)



### 基本原理

​	CyclicBarrier可以使一定数量的线程反复地在栅栏位置处汇集，使用 cylicbarrier 的线程被叫做参与方，cylicbarrier 的内部维护了一个显式锁 ReentrantLock。

​	当线程到达栅栏位置时将调用`await`方法，也是将计数器减1，如果此时计数器不为 0，这些线程就会被暂停，直到最后一个线程也到达栅栏处，并调用 `await`方法，将计数器减为 0，并且唤醒其他暂停的线程，相当于把栅栏打开了，将线程放行了。而最后一个到达的线程是不会被暂停的。



### 使用

```java
//构造器，定义参与的线程数
CyclicBarrier cyclicBarrier = new CyclicBarrier(10);

//构造器，增加了屏障被打破时要执行的动作，当所有线程都通过栅栏之后，执行 barrierAction 任务
public CyclicBarrier(int parties, Runnable barrierAction);

//将屏障重置为其初始状态
void reset()

//进行等待
int await()

//进行等待，同时具备超时时间
public int await(long timeout, TimeUnit unit)
```



测试：

```java
@Slf4j
public class ThreadTestDem16 {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService pool = Executors.newFixedThreadPool(2);
        CyclicBarrier barrier = new CyclicBarrier(2);

        pool.execute(() -> {
            log.debug("running...");
            try {
                Thread.sleep(1000L);
                barrier.await();
            } catch (InterruptedException | BrokenBarrierException e) {
                throw new RuntimeException(e);
            }
            log.debug("end...");
        });

        pool.execute(() -> {
            log.debug("running...");
            try {
                Thread.sleep(2000L);
                barrier.await();
            } catch (InterruptedException | BrokenBarrierException e) {
                throw new RuntimeException(e);
            }
            log.debug("end...");
        });
    }
}
```

```cmd
2026-04-17 17:31:13.815 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - running...
2026-04-17 17:31:13.815 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - running...
2026-04-17 17:31:15.832 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - end...
2026-04-17 17:31:15.832 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - end...
```

可以看到，当第二个线程2s之后也调用 await 之后，双方才接着运行。

构造函数也可以传入需要额外执行的任务

```java
@Slf4j
public class ThreadTestDem16 {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService pool = Executors.newFixedThreadPool(2);
        CyclicBarrier barrier = new CyclicBarrier(2,() -> {
            log.debug("我来善后工作");
        });

        pool.execute(() -> {
            log.debug("running...");
            try {
                Thread.sleep(1000L);
                barrier.await();
            } catch (InterruptedException | BrokenBarrierException e) {
                throw new RuntimeException(e);
            }
            log.debug("end...");
        });

        pool.execute(() -> {
            log.debug("running...");
            try {
                Thread.sleep(2000L);
                barrier.await();
            } catch (InterruptedException | BrokenBarrierException e) {
                throw new RuntimeException(e);
            }
            log.debug("end...");
        });

        pool.shutdown();
    }
}
```

```cmd
2026-04-17 17:36:15.712 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - running...
2026-04-17 17:36:15.712 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - running...
2026-04-17 17:36:17.730 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - 我来善后工作
2026-04-17 17:36:17.731 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - end...
2026-04-17 17:36:17.731 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - end...
```

可重用性：

```java
@Slf4j
public class ThreadTestDem16 {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService pool = Executors.newFixedThreadPool(2);
        CyclicBarrier barrier = new CyclicBarrier(2);

        for (int i = 0; i < 2; i++) {
            pool.execute(() -> {
                log.debug("running...");
                try {
                    Thread.sleep(1000L);
                    barrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    throw new RuntimeException(e);
                }
                log.debug("end...");
            });

            pool.execute(() -> {
                log.debug("running...");
                try {
                    Thread.sleep(2000L);
                    barrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    throw new RuntimeException(e);
                }
                log.debug("end...");
            });
        }

        pool.shutdown();

    }
}
```

```cmd
2026-04-17 17:39:27.609 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - running...
2026-04-17 17:39:27.609 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - running...
2026-04-17 17:39:29.615 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - end...
2026-04-17 17:39:29.615 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - end...
2026-04-17 17:39:29.616 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - running...
2026-04-17 17:39:29.616 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - running...
2026-04-17 17:39:31.620 [pool-1-thread-1] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - end...
2026-04-17 17:39:31.620 [pool-1-thread-2] DEBUG JUC.JUCTest.Demo.D2.ThreadTestDem16 - end...
```



# 二十。并发安全集合类



## 1.HashMap 的并发安全问题



先来看看线程不安全的 HashMap：

HashMap 的并发安全问题：

HashMap底层是数组+链表+红黑树组成的，当发生哈希冲突的时候，底层采用链地址法来解决哈希冲突，即会在桶位上形成链表，但是根据JDK的版本不同，数据挂载在链表上的方式也有所不同。

JDK 8 采用的是尾插法，即新进来的元素会插在链表的尾部。

JDK 7 采用的是头插法，即新进来的元素会插在链表的头部。

<img src="juc并发编程 .assets/屏幕截图 2026-04-17 222636.png" style="zoom:67%;" />

但是，在JDK 7的时候，一旦 HashMap 在**多线程模式下进行扩容**（当元素个数大于扩容阈值、或者链表长度达到8但是数组长度小于64的时候），可能会产生**并发死链问题**，后果非常严重。



- ### 先来看看正常的扩容流程：

扩容逻辑：

简单来说：他会遍历原数组当中的每一个桶位，首先获取桶位上的头节点，并判断该桶位上是否有链表。如果有，将头节点的下一个 enxt节点保存起来，之后计算节点在新数组当中的位置，采用头插法将头节点插入到新数组当中即，之后处理下一个节点。

```java
// JDK7扩容核心代码
void transfer(Entry[] newTable) {
    Entry[] src = table;
    int newCapacity = newTable.length;
    
    for (int j = 0; j < src.length; j++) {
        Entry<K,V> e = src[j];
        if (e != null) {
            src[j] = null; // 清空原数组位置
            do {
                Entry<K,V> next = e.next; // 保存下一个节点
                int i = indexFor(e.hash, newCapacity); // 重新计算位置
                e.next = newTable[i]; // 头插法，每次都插入到链表的头部，也可以理解为插入当前桶位
                newTable[i] = e;      // 插入新位置
                e = next;             // 处理下一个节点
            } while (e != null);
        }
    }
}

```

假设原链表：A → B → C（在数组索引0位置） 扩容后新数组大小为32，假设A、B、C重新计算索引后仍在同一位置：

**单线程扩容过程：**

1. 处理A：先执行A.next = newTable[i]，因为此时新数组的为上还没有元素，所以A.next = null, newTable[0] = A
2. 处理B：B.next = A, newTable[0] = B
3. 处理C：C.next = B, newTable[0] = C
4. 最终链表：C → B → A

​	底层创建一个新的数组，容量为旧数组的两倍，然后将旧数组中的元素迁移到新数组当中。原先链表中的元素的顺序是A、B、C，因为JDK采用头插法将元素插入到链表当中，所以A先迁移，之后B迁移到链表头部，之后是C，所以新的 HashMap 链表中的元素的顺序就是C、B、A。

<img src="juc并发编程 .assets/屏幕截图 2026-04-17 223241.png" style="zoom:67%;" />

- ## 死循环执行步骤1

​	多线程模式下，t1和t2线程都发现，此时HashMap已经触发了扩容条件，那他俩都会去执行扩容，并且将旧数组中的元素进行迁移。

此时，t1与t2都指向了链表的头节点A（e），t1.next与t2.next都指向了B（e.next）

<img src="juc并发编程 .assets/屏幕截图 2026-04-17 224819.png" style="zoom:67%;" />

- ## 死循环执行步骤2

此时线程t2因为时间片用完了，于是进入了休眠状态，线程t1去执行扩容操作。因为此时还只是线程t1一个线程去执行扩容嘛，所以没有安全问题，成功的将数据进行了迁移，迁移过程：

1. 处理A：A.next = null, newTable[0] = A
2. 处理B：B.next = A, newTable[0] = B
3. 处理C：C.next = B, newTable[0] = C
4. 最终链表结构：newTable[0] = C，C.next = B，B.next = A，A.next = null

<img src="juc并发编程 .assets/屏幕截图 2026-04-17 230721.png" style="zoom:67%;" />

- ## 死循环执行步骤3

就在t1完成扩容之后，t2恢复运行，但是t1的扩容过程与结果对t2压根不可见，他根本不知道已经完成扩容了，所以他还会继续进行扩容。

扩容前t2指向A，t2.next也就是 A.next 指向B，扩容之后没变，还是这样。

```java
e = A                    // 当前处理节点A
next = e.next = B        // 保存A的下一个节点B

```

<img src="juc并发编程 .assets/屏幕截图 2026-04-17 230721-1776438993555-5.png" style="zoom:67%;" />

此时t2开始执行剩余代码：

![](juc并发编程 .assets/屏幕截图 2026-04-17 232659.png)

**迁移A：**

**t2继续执行A的剩余步骤：**

```java
// t2认为链表还是 A → B → C → null
// 但实际上链表已经被t1改为 C → B → A → null

i = indexFor(A.hash, newCapacity) = 0
e.next = newTable[0]     // A.next = C（因为Thread1完成后newTable[0]的头节点是C）
newTable[0] = e          // newTable[0] = A（覆盖了C）
e = next                 // e = B（准备处理下一个节点）
```

**关键错误操作：**

- `e.next = newTable[0]` 执行后：`A.next = C`

**当前状态：**

- t1创建的：`C → B → A → null`

- t2修改后：

  `A.next = C`，所以现在有：

  - `C → B → A`（t1的部分）
  - `A → C`（t2添加的）
  - 形成：`A → C → B → A`（部分循环）

**迁移B：**

```java
e = B                    // 当前处理节点B
next = e.next = A        // B的下一个节点是A（因为t1让B指向A）
i = indexFor(B.hash, newCapacity) = 0
e.next = newTable[0]     // B.next = A（因为newTable[0]当前头节点是A）
newTable[0] = e          // newTable[0] = B
e = next                 // e = A
```

**当前状态：**

- `B.next = A`（t2设置）
- `A.next = C`（Thread1之前设置）
- `C.next = B`（Thread2设置）
- 形成：`A → C → B → A`（完整循环）



- ## Thread2继续处理（陷入死循环）

**Thread2处理A（第二次遇到A）：**

```java
e = A                    // 当前处理节点A（第二次遇到）
next = e.next = C        // A的下一个节点是C
i = indexFor(A.hash, newCapacity) = 0
e.next = newTable[0]     // A.next = B（因为newTable[0]当前头节点是B）
newTable[0] = e          // newTable[0] = A
e = next                 // e = C
```

**Thread2处理C**

```java
e = C
next = e.next = B        // C的下一个节点是B
i = indexFor(C.hash, newCapacity) = 0
e.next = newTable[0]     // C.next = A
newTable[0] = e          // newTable[0] = C
e = next                 // e = B
```

**Thread2处理B（第二次遇到B）：**

```java
e = B
next = e.next = A        // B的下一个节点是A
// 无限循环开始...
```

最终形成死循环。

<img src="juc并发编程 .assets/屏幕截图 2026-04-18 000743.png" style="zoom:67%;" />

总结：

HashMap 是一个线程不安全的集合，内部的方法为了保证 HashMap 的并发性，没有使用任何的原子操作来保证数据的安全，也没有使用锁等手段来保证原子性。

虽然在 JDK 8 的时候对数据插入到链表的方式做了修改，将头插法改为了尾插法，防止了并发死链的问题，但是依然不能够防止在多线程模式下进行扩容可能造成数据丢失的问题。



## 2.ConcurrentHashMap

### 重要的属性及内部类

```java
/**
 用于数组扩容，默认为 0
 当初始化时，为 -1。ConcurrentHashMap 是懒惰初始化，初始数组长度为 0，只有第一次使用的时候才初始化创建舒数组
 当扩容时，为 -(1 + 扩容线程数)
 当初始化或扩容完成后，为 下一次的扩容的阈值大小，也就是容量的 0.75
 **/
private transient volatile int sizeCtl;

// 整个 ConcurrentHashMap 中存储的就是一个个 Node
static class Node<K,V> implements Map.Entry<K,V> {}

// hash 表，就是 Node 数组
transient volatile Node<K,V>[] table;

// 扩容时的 新 hash 表
private transient volatile Node<K,V>[] nextTable;
```

- #### 以下均是 Node 的子类

ForwardingNode 转发节点

他的 hash = **MOVED = -1**

```java
static final class ForwardingNode<K,V> extends Node<K,V> {}
```

用途一：

- 扩容时，会创建新的数组，然后将原数组中的节点进行迁移：
- 但是 ConcurrentHashMap 它是线程安全的，所以他得防止在搬迁的过程中其他线程对其产生影响
- 它是这样做的：
- 首先从后往前遍历原数组，一个一个桶位进行数据迁移
- 当某个桶位迁移处理完了，他就会在该桶位链表的头节点添加一个 ForwardingNode 作为头节点
- 当其他线程来了，发现头节点已经变成了 ForwardingNode，他就知道该链表已经处理过了，那他就不会再进行处理了

用途二：

- 如果在扩容的过程中，有其他线程调用 get 获取 key 时候，他发现是 ForwardingNode ，那他就会去新数组中获取 key，而不是在旧数组

<img src="juc并发编程 .assets/屏幕截图 2026-04-18 193604.png" style="zoom:67%;" />



TreeBin

作为红黑树的头节点

```java
// 作为 treebin 的头节点，存储 root 和 first
static final class TreeBin<K,V> extends Node<K,V> {}
```

TreeNode

作为红黑树的节点

```java
// 作为 treebin 的节点，存储 parent, left, right
static final class TreeNode<K,V> extends Node<K,V> {}
```

### 重要方法

```java
// 获取 Node[] 中第 i 个 Node，只是获得链表头部的 Node
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i)

// 使用 cas 操作修改 Node[] 中第 i 个 Node 的值, c 为旧值, v 为新值
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i, Node<K,V> c, Node<K,V> v)

// 直接修改 Node[] 中第 i 个 Node 的值, v 为新值
static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v)
```



## 3.ConcurrentHashMap （JDK 8）实现原理



### 构造器

1. JDK 8 中的 ConcurrentHashMap 采用懒惰初始化，构造方法当中仅仅只是率先计算了 table 数组的容量大小，而未初始化创建数组。只有当第一次使用 ConcurrentHashMap 的时候，才会初始化创建 table，而 JDK 7中是一上来就直接创建，不管使用没使用。

2. 计算 table 容量的时候会使用到传入的初始容量，用初始容量 / 负载因子 + 1，之后调用`tableSizeFor`方法，该方法会将计算出的容量转换为 **2^n**，因为哈希表的容量需要一直都是**2^n**。所以即使设置了初始大小，最终也有可能不是我们传入的初始值大小。计算出之后就将容量赋值给 sizeCtl。供下次初始化创建的时候使用。


```java
public ConcurrentHashMap(int initialCapacity,
                         float loadFactor, int concurrencyLevel) {
    // 1. 参数合法性校验：负载因子>0、初始容量≥0、并发级别>0
    if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    
    // 2. 初始容量至少 ≥ 并发级别（保证桶数量 ≥ 预估线程数）
    if (initialCapacity < concurrencyLevel)
        initialCapacity = concurrencyLevel;
    
    // 3. 根据初始容量和负载因子计算需要的 table 容量
    long size = (long)(1.0 + (long)initialCapacity / loadFactor);
    
    // 4. 转成 2 的幂次方容量（tableSizeFor 保证是 ≥size 的最小2的幂）
    int cap = (size >= (long)MAXIMUM_CAPACITY) ?
        MAXIMUM_CAPACITY : tableSizeFor((int)size);
    
    // 5. 赋值给 sizeCtl：初始化完成前代表【初始化容量】，初始化后代表【扩容阈值】
    this.sizeCtl = cap;
}
```

该构造方法可以接收三个参数：

initialCapacity：初始容量，可以给 Map 一个初始的大小。初始容量至少要 >= 并发度，否则会将初始容量的大小设置为并发数的大小。

loadFactor：负载因子，用于扩容阈值

concurrencyLevel：并发度



### get( ) 原理

1. 当调用了 key 的 hashCode 之后，得到初始的哈希值，接着调用 `spread`哈希扰动函数，该方法的作用的是对初始哈希值进行扰动，目的是为了让 key 的哈希值分布的更加均匀，从而更多的减少哈希冲突，让 key 在哈希表中分布的更加均匀，并且保证其哈希值是一个正整数。

2. 之后会判断 table 数组不为空，并且容量 > 0，接下来会调用 `tabAt(tab, (n - 1) & h))`方法，会根据桶下标来定位链表的头节点，（n - 1）& h 相当于取模操作，来找到桶下标，但是效率更高。之后会判断该头节点是否为空。

3. 如果不为空，接着比较头节点的哈希值是否等于 key 的哈希值，如果哈希值也相同，进一步判断这个 key 是不是和需要查找的 key 是一样的，如果一样，就把这个值返回了。不一样也没关系，接着会调用 `equals`方法进一步判断是不是相同的 key，这是根据内容的值判断的，因为有可能是两个不同的对象，但是他们的内部属性一致，这时候也会认定他们是相同的 key，比如：


```java
String k1 = new String("name");
String k2 = new String("name");

k1 == k2        → false （不是同一个对象）
k1.equals(k2)   → true  （内容一样）
```

​	HashMap 必须认为它们是同一个 **key**，之后也会返回，这是直接命中的情况。

4. 如果头节点的哈希值是负数，说明这个头节点是特殊节点，有两种情况：

- 当前的哈希表正在扩容，此头节点已经完成了迁移，变成了 ForwardingNode，这时取值就为 -1，之后就会调用 ForwardingNode 的`find`方法，会到新 table 数组当中寻找这个 key

- 这个节点是 TreeBin 树的头节点，就会调用 TreeBin  的`find`方法，到红黑树当中去寻找这个 key 

5. 最后，如果既不是负数，也不是头节点，那他就会去遍历这个节点的下边的链表，查找下一个节点，去比较 key 相不相同，相同直接返回，不同，再进一步调用 equala 方法判断内部属性是否相同，是不是相同的 key。

   可以看到，整个 get 的流程当中，**没有使用到任何的锁**。

6. 核心流程总结：

   完全无锁，依赖volatile保证可见性
   通过两次hash定位到具体元素
   如果在桶上直接返回
   如果是红黑树则按树方式查找
   否则按链表遍历查找

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    
    // 1. 计算 key 的哈希值（spread = 二次哈希，减少碰撞）
    int h = spread(key.hashCode());

    // 2. 判断：table不为空 && 长度>0 && 计算索引位置，拿到桶的头节点 e
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {

        // 3. 如果头节点 hash 匹配，直接检查是不是要找的 key
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val; // 命中！直接返回 value
        }
        
        // 4. 如果头节点 hash < 0 → 说明是特殊节点（正在扩容/红黑树）
        // 调用节点自身的 find() 方法查找（TreeBin/ForwardingNode）
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;

        // 5. 普通链表：遍历查找
        while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    // 没找到，返回 null
    return null;
}
```



### put( ) 原理

1. putVal 接收一个参数 onlyIfAbsent ，布尔值 onlyIfAbsent 如果为 true，代表：如果第二次 put 的元素 key 相同，那他就不会做任何操作，也就是不会用新的 key 的 value 覆盖掉旧的 key 的 value。但是此时传入的参数为false，也就是说即使 key 相同，也会用新值覆盖掉旧值。

2. 首先判断 key 和 value 是否为空，如果为空，则抛出空指针异常。这也是和 HashMap 的区别，HashMap 允许 key 最多一个为null，value 允许多个为 null，而 ConcurrentHashMap 的 key、value都不允许为空。

3. 之后也是调用 hashCode 和 spread 来计算 key 的原始哈希值并进行扰动，得到最终的哈希值。

4. 开始进入一个死循环（自旋），首先将原数组 table 赋值给一个临时的 Node 数组 tab，之后判断 tab 是否为空，或者长度是否为0，只要有一个条件满足，就代表哈希表还未创建，这时就会进入`initTable()`方法中，内部会初始化创建哈希表，采用懒加载模式，底层也是采用 **CAS 操作** 防止多个线程重复创建哈希表，确保只能有一个线程 **CAS** 成功，CAS 失败的线程就会进入下一轮循环，再一检查哈希表不为空了，也就不创建了。这里在初始化创建哈希表的时候没使用 synchronized 来上锁，这也是性能提升的关键。

5. 之后进入下一个分支，`(f = tabAt(tab, i = (n - 1) & hash)`也是先计算出桶下标，然后获取该桶位上的头节点 f，如果**头节点为空**，表示这个桶位置还没有被占用，那当前的元素就直接占用即可，使用 CAS 操作创建一个新的 Node 节点，并且将当前键值对元素的 key、value、hash这些值传如 Node 当中，也是使用 CAS 操作，无需 synchronized。这时新的 Node 添加就完毕了。

6. 如果发现头节点不为空，就会进入下一个分支，`else if ((fh = f.hash) == MOVED)`，这里的 MOVED 是等于 -1的，并判断如果当前头节点的哈希值是不是为 -1，因为 ForwardingNode 的哈希值是等于 -1 的，这一步就是在判断当前的头节点是不是 ForwardingNode 节点，如果是，就证明，当前有其他的线程正在对哈希表进行扩容，正因为有线程正在进行扩容嘛，所以我当前的线程肯定不能再读了，要不就会出现问题，那按理来说他应该会被阻塞，等其他线程扩容完成之后再读数据。但是这里比较有意思，他实际上没有被阻塞，而是帮忙去进行扩容，他就会进入`helpTransfer(tab, f)`去帮忙扩容。

7. 扩容的时候是以桶为单位的，一个桶一个桶处理。线程帮忙进行扩容的时候，会按**数组区间**分配任务，每个线程都会被分配一段桶的下标区间，线程从数组的高位往低位处理桶，处理每个桶前，先使用**synchronized 锁住桶的头节点**，再进行数据迁移，迁移完成后将旧桶标记为 MOVED，保证其他线程不会重复操作，就保证了该桶的线程安全。

8. 如果此时**哈希表既不在初始化，查找的 key 也不是头节点，而且哈希表也不在扩容的过程中**，那此时肯定是发生了哈希冲突，那就会进入最后一个 else 分支。桶下标冲突了，此时使用 synchronized 锁住头节点，这也是设计的巧妙的地方，前面的分支都没有加锁，只有当桶下标发生冲突的时候才会加锁，而且锁住的也不是整个 table，而是只锁住当前链表的头节点。

9. 之后再次调用`tabAt(tab, i) == f`获得头节点，并判断头节点是不是被移动过。没有，就可以放心的进行 put 操作了。

   put 操作又分为两种，先判断头节点的哈希值是不是大于等于0：

- 大于等于0，说明这个头节点就是一个普通的头节点。接下来就会去遍历这个链表，遍历的过程中如果发现已经存在了 key 相同的键值对，那新的 key 的 value 就会覆盖掉旧的 key 的 value。如果不存在，那就会在链表的尾部追加一个新的 Node，底层会创建一个新的 Node，并把元素的 key、value、hash这些值添加到 Node当中。

- 小于0，说明这个头节点可能是 红黑树 的头节点，或者是 ForwardingNode 。之后会进一步判断该节点是不是红黑树的头结点，如果是，就会调用红黑树的`putTreeVal`方法，往红黑树当中添加节点

10. putVal 当中有一个变量 binCount，用来存储当前桶位的链表的长度，初始值为 0。释放锁之后，会根据 binCount 来判断是不是需要对链表进行优化，当 binCount 达到了树化阈值（8），会尝试将链表升级为红黑树，当然不是立马就能升级为红黑树，会先扩容，等整个哈希表的长度达到了64之后，如果链表的长度还是8或者更高，才会升级为红黑树。

11. 最后执行`addCount`将 size 的个数+1。

12. 核心流程总结：

    计算key的hash值
    如果表未初始化，则初始化表
    如果定位到的桶为空，尝试CAS插入新节点
    如果桶正在迁移(ForwardingNode)，帮助迁移
    否则对桶的头节点加synchronized锁：
    如果是链表，遍历查找并插入/更新
    如果是树，按照红黑树方式插入
    判断是否需要树化或扩容

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}

/**
 * ConcurrentHashMap 核心插入方法
 * @param key 键
 * @param value 值
 * @param onlyIfAbsent true：不存在才插入；false：覆盖旧值
 * @return 旧值，没有则返回null
 */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    // ConcurrentHashMap 不允许 key/value 为 null
    if (key == null || value == null) throw new NullPointerException();

    // 计算 key 的哈希值（二次扰动，减少冲突）
    int hash = spread(key.hashCode());

    // 记录当前桶的元素个数（用于判断是否树化）
    int binCount = 0;

    // 自旋循环：CAS 失败时重试插入
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f;           // 桶的头节点
        int n, i, fh;          // n=数组长度 i=数组下标 fh=头节点hash
        K fk; V fv;

        // 1. 容器未初始化 → 初始化数组（懒加载）
        if (tab == null || (n = tab.length) == 0)
            // 初始化创建哈希表
            tab = initTable();

        // 2. 目标桶为空 → CAS 无锁插入新节点
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            // CAS 原子更新，成功就退出循环
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value)))
                break;                   
        }

        // 3. 当前桶正在扩容（hash == MOVED）→ 协助迁移数据
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);

        // 4. onlyIfAbsent 模式：key 已存在，直接返回旧值（不加锁）
        else if (onlyIfAbsent 
                 && fh == hash
                 && ((fk = f.key) == key || (fk != null && key.equals(fk)))
                 && (fv = f.val) != null)
            return fv;

        // 5. 桶非空、不在扩容 → 加锁插入/覆盖
        else {
            V oldVal = null;

            // 锁 头节点 f → 分段锁，高并发核心
            synchronized (f) {
                // 再次确认头节点没有被其他线程修改
                if (tabAt(tab, i) == f) {

                    // 5.1 头节点是普通链表节点（fh >= 0）
                    if (fh >= 0) {
                        binCount = 1;
                        // 遍历链表
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;

                            // key 已存在 → 覆盖 value
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }

                            // 到链表尾部 → 追加新节点
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key, value);
                                break;
                            }
                        }
                    }

                    // 5.2 桶已经树化 → 红黑树插入
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }

                    // 5.3 保留节点异常（递归更新错误）
                    else if (f instanceof ReservationNode)
                        throw new IllegalStateException("Recursive update");
                }
            }

            // 插入完成后的处理
            if (binCount != 0) {
                // 链表长度 ≥ 8 → 转为红黑树
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);

                // 存在旧值 → 返回
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }

    // 元素数量 +1，检查是否需要扩容
    addCount(1L, binCount);
    return null;
}
```



### initTable( ) 原理

1. 当向哈希表当中 put 元素的时候，如果判断哈希表还未被初始化创建，此时会调用`initTable`方法初始化创建哈希表，这是采用的懒惰初始化。
2. 初始化的时候保证只能有一个线程完成初始化的操作，防止多个线程重复操作。
3. 首先也是进入一个循环，先判断哈希表是否为空，或者长度是否为0，满足，就代表哈希表还未被初始化创建，就会进入循环。
4. 首先进入第一个 else if 当中，使用 CAS 操作尝试将 sizeCtl 的值从 0 改为 -1，因为 sizeCtl 的值为 -1 时，代表正在执行初始化创建哈希表。CAS 成功的线程就会去创建哈希表，没有成功的线程就会进入下一轮循环，一检查，sizeCtl 已经变为了负数，代表已经有其他线程正在执行创建的工作了，那他就会调用 yield 让出 CPU，此时是忙等待，并没有被阻塞住，因为是让出 CPU，所以过一会他还可能会分到 CPU 再去执行。
5. 之后线程创建哈希表，再次检查哈希表确实还没有被创建，如果预设的 sc 容量大于 0 ，那他就会根据预设的初始容量长度去创建 Node 数组。如果没有给预设的初始容量，就会根据默认的初始容量 DEFAULT_CAPACITY（16）创建一个 Node 数组。
6. 创建完成一个会再次计算 sc 的值，但此时 sc 已经不作为初始容量了，而是作为下一次扩容时的阈值。之后将 sc 赋值给 sizeCtl ，此时 sizeCtl 已经是正数了。之后其他忙等待的线程下次循环，发现哈希表已经创建好了，那他就直接退出循环了。

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; 
    int sc;
    // 自旋循环：table为null或长度为0时，持续尝试初始化
    while ((tab = table) == null || tab.length == 0) {
        // sizeCtl < 0：说明有其他线程正在初始化table
        if ((sc = sizeCtl) < 0)
            // 当前线程让出CPU，自旋等待，不参与竞争
            Thread.yield(); 
        // CAS 尝试将 sizeCtl 设置为 -1：标记当前线程开始独占初始化
        // 成功：当前线程获得初始化权限；失败：其他线程已抢占
        else if (U.compareAndSetInt(this, SIZECTL, sc, -1)) {
            try {
                // 二次检查：防止多线程下重复初始化
                if ((tab = table) == null || tab.length == 0) {
                    // 确定数组容量：sc>0用指定容量，否则用默认容量16
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    // 泛型数组创建：强制类型转换，压制未检查转换警告
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    // 赋值给全局table和局部变量tab
                    table = tab = nt;
                    // 计算扩容阈值：n - n/4 = 0.75n（负载因子0.75）
                    sc = n - (n >>> 2);
                }
            } finally {
                // 无论是否异常，最终将sizeCtl设置为扩容阈值
                // 释放初始化锁，其他线程可以正常使用table
                sizeCtl = sc;
            }
            // 初始化完成，退出循环
            break;
        }
    }
    // 返回初始化好的哈希表数组
    return tab;
}
```



### addCount( ) 原理

1. 在 put 的流程中，最后一步会调用这个 `addCount`方法。该方法的作用是：更新 **元素计数 + 触发扩容。**
2. 该方法类似 LongAdder 原子累加器，也是设置了多个累加单元，从而保证多个线程做累加操作的时候 CAS 冲突减少，从而提升性能。
3. 首先判断，累加单元数组 counterCells 是否为空，如果为空，说明还没有竞争，就会使用 CAS 操作向一个基础数值 BASECOUNT 进行累加操作 +1。
4. 如果不为空，说明已经有竞争了，或者 CAS 失败了，这时就会进入 if 逻辑内。
5. 该逻辑内部会再次判断累加单元数组是不是为空，或者长度小于等于0，如果是就会进入` fullAddCount(x, uncontended)`方法内部，创建累加单元数组。但是如果累加单元数组有了，而当前线程对应的累加单元还没创建，也会进入`fullAddCount(x, uncontended)`内，去创建累加单元。
6. 如果都有了，就可以进行累加了，也是使用 CAS 操作，先取得累加单元的上一个值，然后在上一个值的基础上加1。但是如果这个 CAS 失败了，也还是会进入到 `fullAddCount(x, uncontended)`当中，里面也是死循环，做不断的尝试。
7. 完成以上这些，会检查一下链表长度，如果长度 <=1，就直接返回。但是如果链表的长度 >1，那就代表当前的链表有可能需要扩容，触发扩容，那就会接着向下走。
8. 首先获得当前哈希表的所有元素的个数，并判断元素个数是否大于扩容阈值，如果成立就会进入扩容的流程。
9. 扩容时，先使用 CAS 操作将 sizeCtl 设置为负数，表示要进入扩容的流程了，之后进入`transfer(tab, null)`进行扩容。那其他 CAS 操作失败的线程就进入下一次循环，发现 sizeCtl 已经为负数了，说明扩容的时候其他线程已经创建好了新的 table 数组，那他也会去帮忙扩容。

```java
private final void addCount(long x, int check) {
    CounterCell[] cs;
    long b, s;

    // 第一步：尝试用最轻快的方式更新计数
    // 逻辑：如果 counterCells 不为空（已经用过分段计数）
    // 或者 CAS 更新 baseCount 失败（竞争冲突）
    // 就进入分段计数逻辑
    if ((cs = counterCells) != null ||
        !U.compareAndSetLong(this, BASECOUNT, b = baseCount, s = b + x)) {

        CounterCell c;
        long v;
        int m;
        boolean uncontended = true; // 标记是否无竞争

        // 第二层判断：任意一个条件满足，就进入完整计数逻辑 fullAddCount
        // 1. cs == null：分段数组未初始化
        // 2. 数组长度 <=0
        // 3. 当前线程对应的分段Cell为null
        // 4. CAS 更新当前分段Cell的值失败（发生竞争）
        if (cs == null || (m = cs.length - 1) < 0 ||
            (c = cs[ThreadLocalRandom.getProbe() & m]) == null ||
            !(uncontended =
              U.compareAndSetLong(c, CELLVALUE, v = c.value, v + x))) {

            // 竞争激烈，调用完整方法处理计数（会初始化/扩容CounterCell数组）
            fullAddCount(x, uncontended);
            return;
        }

        // 如果不需要检查扩容，直接返回
        if (check <= 1)
            return;

        // 计算当前总元素个数（baseCount + 所有CounterCell之和）
        s = sumCount();
    }

    // 第二步：检查是否需要扩容（check >= 0 才执行）
    if (check >= 0) {
        Node<K,V>[] tab, nt;
        int n, sc;

        // 循环判断：
        // 1. 元素总数 s >= 扩容阈值 sizeCtl
        // 2. 哈希表不为空
        // 3. 当前长度未达到最大容量
        // 满足 → 触发扩容
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
               (n = tab.length) < MAXIMUM_CAPACITY) {

            // 生成扩容标识戳：用于标记当前扩容的容量，防止错误扩容
            int rs = resizeStamp(n) << RESIZE_STAMP_SHIFT;

            // sc < 0：说明**已经有线程正在扩容**
            if (sc < 0) {
                // 这些条件表示扩容已经结束、或者达到最大协助线程数，直接退出
                if (sc == rs + MAX_RESIZERS || sc == rs + 1 ||
                    (nt = nextTable) == null || transferIndex <= 0)
                    break;

                // CAS 尝试将扩容线程数 +1 → 当前线程**协助扩容**
                if (U.compareAndSetInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt); // 协助迁移数据
            }
            // sc >= 0：没有线程在扩容，当前线程**发起扩容**
            // CAS 将 sizeCtl 设置为 rs + 2 → 标记扩容开始，初始化扩容线程数=1
            else if (U.compareAndSetInt(this, SIZECTL, sc, rs + 2))
                transfer(tab, null); // 真正执行扩容、数据迁移

            // 重新计算元素总数，继续循环判断是否需要继续扩容
            s = sumCount();
        }
    }
}
```



### size( ) 原理

1. 通过 size 方法可以获得哈希表当中元素的个数。size 计算实际发生在 put 、remove 这些改变集合中元素的操作当中。
2. 当没有竞争发生的时候，直接向 baseCount 做累加操作。
3. 有竞争发生的时候，会创建累加单元数组，向其中一个累加单元做累加操作。
4. 累加单元初始的时候，只有两个累加单元，当竞争非常激烈的时候，会创建更多的累加单元。
5. 首先获取 baseCount ，然后去遍历累加单元数组，看看每个累加单元的值是否为空，不为空，就和 baseCount 做一个相加的操作，然后将该值返回。
6. size( ）它是弱一致性的，方法返回的是近似值，不是精确值，因为是通过每个累加单元统计的，不加全局锁，所以很有可能在统计的过程中，其他线程又添加或者删除了元素，导致累计单元发生变化。只有当 size 的时候，没有任何的线程修改了哈希表，才会返回精确值。

```java
/**
 * 返回当前哈希表中【键值对的总数量】
 * 注意：sumCount() 是统计总数，返回的是 long，这里转成 int 并做边界处理
 */
public int size() {
    // 1. 调用 sumCount() 拿到【真实、完整的元素总数】（long类型，防止溢出）
    long n = sumCount();
    
    // 2. 安全转换为 int 类型返回
    return ((n < 0L) ? 0 :        // 总数不可能 < 0，出现直接返回 0
            (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE :  // 超过int最大值，返回最大值
            (int)n);              // 正常情况，强转int返回
}

/**
 * 【核心统计方法】
 * 计算整个 Map 的真实元素总数：baseCount + 所有 CounterCell 计数
 * 高并发下可能是【弱一致性】的值（统计过程中可能有线程增删元素）
 */
final long sumCount() {
    CounterCell[] cs = counterCells; // 分段计数数组（高并发用）
    long sum = baseCount;            // 第一步：先加【基础计数】
    
    // 第二步：如果分段计数数组不为空，遍历所有分段，把计数累加到 sum
    if (cs != null) {
        for (CounterCell c : cs)
            if (c != null)          // 过滤掉空的分段（避免空指针）
                sum += c.value;     // 累加每个分段的计数值
    }
    
    // 返回最终总和 = 整个 Map 的真实元素个数
    return sum;
}
```



### transfer 扩容原理

1. 该方法接收两个参数，原始的数组，和新的数组，因为新数组也采用延迟初始化，所以此时为 null。

2. 之后判断，如果新数组为 null，就会将新数组创建出来。原始容量翻倍，创建一个新的数组，然后赋值给新数组。

3. 创建完新数组之后，就会进行节点的迁移工作，将旧数组中的节点搬迁到新数组当中。迁移的时候以桶为单位，一个桶一个桶的进行迁移。

4. 他会判断，链表的头结点是不是 null，如果是，代表这个桶已经完成了搬迁，他就会在头结点添加一个 ForwardingNode。如果其他的线程发现头结点已经是 ForwardingNode 了，那他就会去迁移下一个桶位。

5. 但是如果发现该桶位的链表头节点有元素，那他就会使用 synchronized 将该头节点锁住，然后迁移数据，保证了线程安全。迁移的时候也是根据节点的哈希值来判断节点属于哪种类型的节点，普通节点还是红黑树节点，迁移的方式也就有所不同。

6. 核心流程总结：

   当元素数量超过容量*负载因子时触发扩容
   扩容时创建新表(大小为原表2倍)
   采用多线程协助扩容机制，提高扩容效率
   扩容期间可以同时进行查询操作
   通过 ForwardingNode 标记已迁移的桶

```java
    /**
     * Moves and/or copies the nodes in each bin to new table. See
     * above for explanation.
     */
    private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        int n = tab.length, stride;
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; // subdivide range
        if (nextTab == null) {            // initiating
            try {
                @SuppressWarnings("unchecked")
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
            transferIndex = n;
        }
        int nextn = nextTab.length;
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
        boolean advance = true;
        boolean finishing = false; // to ensure sweep before committing nextTab
        for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh;
            while (advance) {
                int nextIndex, nextBound;
                if (--i >= bound || finishing)
                    advance = false;
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;
                    advance = false;
                }
                else if (U.compareAndSetInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                if (finishing) {
                    nextTable = null;
                    table = nextTab;
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }
                if (U.compareAndSetInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    finishing = advance = true;
                    i = n; // recheck before commit
                }
            }
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
            else if ((fh = f.hash) == MOVED)
                advance = true; // already processed
            else {
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        Node<K,V> ln, hn;
                        if (fh >= 0) {
                            int runBit = fh & n;
                            Node<K,V> lastRun = f;
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                        else if (f instanceof TreeBin) {
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                }
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                        else if (f instanceof ReservationNode)
                            throw new IllegalStateException("Recursive update");
                    }
                }
            }
        }
    }
```





## 4.ConcurrentHashMap （JDK 7）实现原理

JDK 7 中的 ConcurrentHashMap 维护了一个 segment 数组，每个 segment 都是一把锁，其继承于 ReentrantLock

- 优点：如果多个线程访问不同的 segment，线程访问的时候，只对自己访问的 segment 加锁，那就不会有冲突。
- 缺点：segment 数组的大小默认为 16，而且这个容量大小指定之后就不能变了，而且数组也不是懒惰初始化，只要构造方法一执行，就会被创建出来。

### 构造器

在构造器内直接创建了 segment 数组，而不是用到的时候才创建，所以对内存的占用是比较大的。

此外，数组一旦被创建，紧接着会创建下标为 0 的元素，内部是一个 HashEntry，对应着一个真正的哈希表，结构是数组加链表的组合。每个 segment 对应着一个小的哈希表，这就是分段锁，将来不同的线程访问不同的 segment，锁住的 segment 也就不同。

![](juc并发编程 .assets/屏幕截图 2026-04-19 230904.png)

其中 this.segmentShift 和 this.segmentMask 的作用是决定将 key 的 hash 结果匹配到哪个 segment

例如，根据某一 hash 值求 segment 位置，先将高位向低位移动 this.segmentShift 位

![](juc并发编程 .assets/屏幕截图 2026-04-19 231222.png)

结果再与 this.segmentMask 做位与运算，最终得到 1010 即下标为 10 的 segment

![](juc并发编程 .assets/屏幕截图 2026-04-19 231227.png)

```java
@SuppressWarnings("unchecked")
public ConcurrentHashMap(int initialCapacity,
                         float loadFactor, int concurrencyLevel) {
    // 1. 参数合法性校验
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    // 并发级别不超过最大值65536
    if (concurrencyLevel > MAX_SEGMENTS)
        concurrencyLevel = MAX_SEGMENTS;

    // 2. 计算Segment数组长度ssize（≥concurrencyLevel的最小2的幂）
    int sshift = 0;
    int ssize = 1;
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1; // 左移1位=×2
    }
    // 定位Segment用：hash >>> segmentShift & segmentMask
    this.segmentShift = 32 - sshift;
    this.segmentMask = ssize - 1;

    // 3. 总容量不超过最大值
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;

    // 4. 计算每个Segment的HashEntry数组容量c
    int c = initialCapacity / ssize;
    if (c * ssize < initialCapacity) // 向上取整
        ++c;
    // 每个Segment的table最小为2，且必须是2的幂
    int cap = MIN_SEGMENT_TABLE_CAPACITY;
    while (cap < c)
        cap <<= 1;

    // 5. 初始化Segment数组，仅创建第0个Segment（其余懒加载）
    Segment<K,V> s0 =
        new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
                         (HashEntry<K,V>[])new HashEntry[cap]);
    Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];;
    // UNSAFE.putOrderedObject：原子写入，保证可见性
    UNSAFE.putOrderedObject(ss, SBASE, s0);
    this.segments = ss;
}
```



### put( ) 原理

当调用 put 添加元素的时候，先是调用的 ConcurrentHashMap 的 put 方法。

1. 首先计算 key 的哈希值，然后根据哈希值进行移位以及掩码运算，最终得出 segment 数组的下标。
2. 之后会判断该下标处的 segment 是否为空，如果为空，就会调用`ensureSegment`方法创建 segment 对象。
3. 创建完成之后，会调用 Segment 的 put 方法添加元素，添加的时候，会将元素的 key、value、hash这些值传入进去。

```java
public V put(K key, V value) {
    Segment<K,V> s;
    if (value == null)
        throw new NullPointerException();
    int hash = hash(key);
    // 计算出 segment 下标
    int j = (hash >>> segmentShift) & segmentMask;

    // 获得 segment 对象, 判断是否为 null, 是则创建该 segment
    if ((s = (Segment<K,V>)UNSAFE.getObject
        (segments, (j << SSHIFT) + SBASE)) == null) {
        // 这时不能确定是否真的为 null, 因为其它线程也发现该 segment 为 null,
        // 因此在 ensureSegment 里用 cas 方式保证该 segment 安全性
        s = ensureSegment(j);
    }
    // 进入 segment 的put 流程
    return s.put(key, hash, value, false);
}
```

Segment 继承了 ReentrantLock。

1. 首先尝试进行一次 tryLock 加锁操作，如果失败了，就会进入`scanAndLockForPut`方法内部，循环的进行尝试加锁的操作，最多尝试 64 次，如果还是失败了们就会进入 lock 流程，最终大概率会被阻塞住。
2. 如果加锁成功，就接着向下执行，此时已经加锁了，所以对该 segment 访问的时候，肯定是线程安全的。
3. 进入 segment ，每个 segment 内部都是一个 HashEntry，对应着一个小的哈希表。之后根据元素的哈希值，来确定桶下标，然后找到链表的头节点，遍历链表。
4. 遍历的过程中，如果存在与新添加的元素的 key 相同的 key，那新的 value 就会覆盖掉旧的 value。
5. 如果没有，就走新增的流程。创建一个新的 Node 节点，然后将元素的 key、value、hash这些值传入进去。
6. 之后会会判断以下哈希表中的元素个数是否达到了扩容阈值，如果是，就走扩容流程，进入`rehash`方法内。
7. 如果不是，最终将节点设置为链表的头节点。没错，JDK 7 的ConcurrentHashMap 也是采用了头插法，但是由于是线程安全的，不会产生并发死链的问题。
8. 最终将锁释放掉。

```java
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    // 1. 获取锁：尝试非阻塞获取锁，失败则进入循环重试加锁（lock()）
    HashEntry<K,V> node = tryLock() ? null : scanAndLockForPut(key, hash, value);
    
    V oldValue;
    try {
        // 当前 Segment 内部的哈希表
        HashEntry<K,V>[] tab = table;
        // 计算在 Segment 内部的数组下标
        int index = (tab.length - 1) & hash;
        // 获取该下标下的链表头节点
        HashEntry<K,V> first = entryAt(tab, index);

        // 2. 遍历链表
        for (HashEntry<K,V> e = first;;) {
            if (e != null) {
                // 节点存在：key 相同，覆盖 value
                if (e.hash == hash && 
                    ((k = e.key) == key || key.equals(k))) {
                    oldValue = e.value;
                    // 不覆盖（putIfAbsent）则直接返回旧值
                    if (!onlyIfAbsent || oldValue == null) {
                        e.value = value;
                        modCount++;
                    }
                    break;
                }
                e = e.next;
            } 
            // 3. 没找到节点：执行插入
            else {
                if (node != null)
                    node.setNext(first); // 头插法
                else
                    node = new HashEntry<K,V>(hash, key, value, first);
                
                int c = count + 1;
                // 4. 超过阈值，扩容
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    rehash(node);
                else
                    setEntryAt(tab, index, node); // 替换头节点
                
                modCount++;
                count = c;
                oldValue = null;
                break;
            }
        }
    } finally {
        // 5. 必须释放锁
        unlock();
    }
    return oldValue;
}
```



### rehash( ) 扩容原理

1. rehash 发生在 put 的过程当中，外部已经加了锁，所以无需担心线程安全问题。
2. 首先获取旧数组的原始容量，之后进行翻倍，根据新的容量创建一个新的哈希表，容量为旧数组的2倍。
3. 之后从低位向高位遍历原哈希表，对每个桶位的数据进行迁移，迁移的的方式会根据桶位上的数据的形式有所不同，分为三种形式：
4. 首先获取当前桶位的头节点元素 e，以及头节点的下一个 next 节点，如果 next = null，则说明当前桶位只有 e 一个节点，那很简单，直    接重新计算桶下标，将该节点赋值给新数组中对应桶位的头节点 ，newTable[idx] = e，相当于直接平移过去。
5. 如果 next != null，则说明该桶位上已经形成链表了，而链表的迁移就有学问了：他的宗旨是尽量对链表进行平移搬迁，而不是重新创建节点

- 首先他会遍历该链表，从头节点开始，例如：头节点是 A，计算新的桶下标，然后获取 next 节点，比如：B，他会计算 B 在新的哈希表当中的桶下标，之后判断，B 在新的桶的下标位置和他的上一个节点 A 的新的桶下标位置是否相同，如果不同发生了改变，那他就会将该 B 节点（lastRun），以及新的桶下标记录下来（lastIdx ）。
- 之后再遍历 B 的 next 节点，比如 C，如果发现 C 和 B一样，桶下标也改变了，并且两者的新的桶下标还相同，之后是 D 以此类推，那他就会将这连续的一批节点，以 B 为头节点，直接从原来的链表断开，整体平移到新的哈希表的桶位上。相当于新数组当中的链表是复用了以 B 为头节点原来的部分链表。
- 而剩余的节点就需要重建了，也是从头节点开始遍历，遍历到 lastRun，创建一个新的 HashEntry，将该节点的 key、value、hash等传进去，之后将这个新的 HashEntry，然后采用头插法插入到新数组的链表当中。而原先的节点则留在了旧的哈希表当中。
- 扩容完成之后，也不要忘了之前传入的新增节点，因为 rehash 发生新增的时候嘛，因为刚才正在扩容，所以还没有新增节点，接下来就要新增节点。
- 也是计算节点在新的哈希表当中的桶下标，然后采用头插法将节点插入链表头。

6. 最终将新的哈希表替换了旧的哈希表。

```java
/**
 * 扩容方法：仅在当前 Segment 持有锁时调用
 * 扩容逻辑：容量翻倍 -> 重新哈希所有元素 -> 新数组赋值
 */
void rehash(HashEntry<K,V> node) {
    HashEntry<K,V>[] oldTable = table;        // 旧数组
    int oldCapacity = oldTable.length;        // 旧容量
    int newCapacity = oldCapacity << 1;       // 新容量 = 旧容量 × 2（必须是2的幂）
    threshold = (int)(newCapacity * loadFactor); // 新扩容阈值
    HashEntry<K,V>[] newTable = (HashEntry<K,V>[]) new HashEntry[newCapacity]; // 新建数组
    int sizeMask = newCapacity - 1;           // 新掩码，用于重新计算下标

    // 遍历旧数组的每一个位置
    for (int i = 0; i < oldCapacity; i++) {
        HashEntry<K,V> e = oldTable[i];       // 拿到链表头 e
        if (e != null) {
            HashEntry<K,V> next = e.next;
            int idx = e.hash & sizeMask;      // 计算在新数组中的下标

            if (next == null) { 
                // 链表只有一个节点，直接赋值
                newTable[idx] = e;
            } else { 
                // 链表有多个节点：重新拆分
                HashEntry<K,V> lastRun = e;
                int lastIdx = idx;

                // 找到后面连续一批 新下标相同 的节点
                for (HashEntry<K,V> last = next; last != null; last = last.next) {
                    int k = last.hash & sizeMask;
                    if (k != lastIdx) {
                        lastIdx = k;
                        lastRun = last;
                    }
                }
                
                // 把这批节点直接整体迁移，不用复制
                newTable[lastIdx] = lastRun;

                // 剩下的节点，逐个复制、头插法迁移
                for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
                    V v = p.value;
                    int h = p.hash;
                    int k = h & sizeMask;
                    HashEntry<K,V> n = newTable[k];
                    // 头插法！！！
                    newTable[k] = new HashEntry<K,V>(h, p.key, v, n);
                }
            }
        }
    }

    // 把新加入的 node 也插入到新数组
    int nodeIndex = node.hash & sizeMask;
    node.setNext(newTable[nodeIndex]);
    newTable[nodeIndex] = node;

    // 替换旧数组为新数组
    table = newTable;
}
```



### get( ) 原理

与 JDK 8 的 ConcurrentHashMap 一样，get 也是完全未加锁的，调用了 UNSAFE 的 `getObjectVolatile` 来保证了可见习性。

首先也是先定位 segment，然后定位桶下标，之后判断 key 相不相同，最终返回。

```java
public V get(Object key) {
    Segment<K,V> s;
    HashEntry<K,V>[] tab;

    // 1. 计算 hash
    int h = hash(key);
    // 2. 根据 hash 定位 segment 索引
    long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;

    // 3. UNSAFE 原子获取对应的 Segment
    if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
        (tab = s.table) != null) {
        // 4. 遍历当前 Segment 里的 HashEntry 链表
        for (HashEntry<K,V> e = (HashEntry<K,V>)UNSAFE.getObjectVolatile
                 (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
             e != null; e = e.next) {
            K k;
            // 找到 key 就返回 value
            if ((k = e.key) == key || (e.hash == h && key.equals(k)))
                return e.value;
        }
    }
    return null;
}
```



### size( ) 原理

首先进入一个循环，不断的尝试，尝试的过程中，去遍历每一个 segment，只要 segment 不为空，就会去计数。

内部有两个属性，seg.count：当前 segment 内部元素的个数，seg.modCount：修改的次数计数，比如执行 put、remove 都会增加其大小。

**无锁重试机制**：

先通过 2 轮无锁遍历（`retries < 2`），累加每个 Segment 的 `count`（元素数）和 `modCount`（修改次数）。若两轮 `modCount` 总和一致，说明统计期间无并发修改，直接返回结果 ——**高性能优先**。

**加锁兜底**：

若 2 次无锁重试仍不稳定（`retries == 2`），则**一次性锁住所有 Segment**，再遍历统计，确保结果绝对精准 ——**准确性兜底**。

**溢出处理**：

当元素总数超过 `Integer.MAX_VALUE` 时，`overflow` 标记为 `true`，最终返回 `Integer.MAX_VALUE`（符合 Map 规范）。

```java
public int size() {
    final Segment<K, V>[] segments = this.segments;
    int size;                  // 最终元素总数
    boolean overflow;         // 溢出标记：超过 Integer.MAX_VALUE 时为 true
    long sum;                  // 本轮所有 Segment 的 modCount 总和（修改次数）
    long last = 0L;           // 上一轮 modCount 总和（用于校验无并发修改）
    int retries = -1;         // 重试次数（首次循环不算重试，从 0 开始）

    try {
        // 死循环：直到获取稳定计数或成功加锁
        for (; ; ) {
            // 重试次数达到阈值（RETRIES_BEFORE_LOCK = 2），强制给所有 Segment 加锁
            if (retries++ == RETRIES_BEFORE_LOCK) {
                for (int j = 0; j < segments.length; ++j) {
                    ensureSegment(j).lock(); // 确保 Segment 存在并加锁，阻塞写操作
                }
            }

            sum = 0L;
            size = 0;
            overflow = false;

            // 遍历所有 Segment，累加 count 和 modCount
            for (int j = 0; j < segments.length; ++j) {
                Segment<K, V> seg = segmentAt(segments, j);
                if (seg != null) {
                    sum += seg.modCount; // 累加修改次数（用于检测并发变动）
                    int c = seg.count;   // 当前 Segment 的元素个数
                    // 处理溢出：count 为负或累加后溢出，标记 overflow
                    if (c < 0 || (size += c) < 0) {
                        overflow = true;
                    }
                }
            }

            // 校验：本轮与上一轮 modCount 总和一致 → 统计期间无并发修改，直接退出
            if (sum == last) {
                break;
            }
            // 不一致 → 更新 last 为当前 sum，继续下一轮重试
            last = sum;
        }
    } finally {
        // 若曾加锁（重试次数超过阈值），释放所有 Segment 的锁
        if (retries > RETRIES_BEFORE_LOCK) {
            for (int j = 0; j < segments.length; ++j) {
                segmentAt(segments, j).unlock();
            }
        }
    }

    // 溢出返回 Integer.MAX_VALUE，否则返回实际 size
    return overflow ? Integer.MAX_VALUE : size;
}
```



## 5.CopyOnWriteArrayList

ArrayList 本身是线程不安全的，，内部的所有操作都没有使用同步机制，简单来说没加锁，多线程操作的话可能会出现线程安全问题。

CopyOnWriteArrayList（COW）是 JUC 包下的线程安全的 LIst，适用于读多写少的场景，通过“写时赋值”的策略实现了读写分离。读操作不加任何的锁，读时获取数组的引用快照，基于旧容器读，多线程之间完全并发读；写操作，先加锁，复制原数组，创建新副本，基于新数组写。读写基于不同的容器，体现了读写分离的思想。

特点：

- 读操作不加锁，完全并发读
- 写操作复制，加锁拷贝原数组，线程基于新的副本做增删改操作
- 读-读、读-写均并发，写-写互斥



### add( ) 原理

1. 首先 synchronized 加锁，保证线程之间互斥执行。
2. 之后获取旧数组，以及旧数组的长度，底层创建一个新的数组，并将原数组的数据拷贝到新数组，然后新数组的长度加1，新的元素添加到数组的尾部。
3. 最终将新数组替换掉旧数组。

```java
public boolean add(E e) {
    synchronized (lock) {          // 加锁（同一时间只有一个线程能执行）
        Object[] es = getArray();   // 获取当前底层数组
        int len = es.length;        // 拿到数组长度
        // 重点：创建一个新数组，长度+1，复制旧数据
        es = Arrays.copyOf(es, len + 1);
        es[len] = e;                // 把新元素放到最后
        setArray(es);               // 替换底层数组
        return true;
    }
}
```

读操作：例如 foreach 遍历：

基于旧数组读，不加任何锁

```java
public void forEach(Consumer<? super E> action) {
    Objects.requireNonNull(action);  // 判空，防止空指针
    for (Object x : getArray()) {    // 重点：遍历的是【当前获取的数组快照】
        @SuppressWarnings("unchecked") E e = (E) x;
        action.accept(e);            // 执行遍历逻辑
    }
}
```



### get( ) 弱一致性

get( ) 是弱一致性的，当前读到的元素可能不是数组当中最新的元素。

比如：t1 先读，由于是基于旧数组读，他先获取到旧数组的引用，找到了内存当中的数组。

但是与此同时 t2 又进行了写操作，并将旧的数组替换成新的数组，而且这一步快于了 t1 的读，此时集合当中的数组肯定是最新的。

但是 t1 此时拿到的还是旧数组的引用，读时还是基于旧数组度的，读到的数据就不是最新的了。

![](juc并发编程 .assets/屏幕截图 2026-04-21 163618.png)
