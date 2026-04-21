

# JUC 并发编程学习笔记

> JUC 并发编程个人学习笔记，知识内容来源：个人理解，博客内容，推文，视频内容，源码级别分析，Ai讲解。

## 笔记内容:

- 同步与异步，进程与线程，并发与并行
- 线程安全与线程不安全原理分析
- synchronized 底层原理，锁升级流程（JVM 层面）
- ReentrantLock 底层原理，包含源码分析
- 各种线程安全的原子类、累加器（LongAdder/DoubleAdder）
- CAS 原理，使用方式，以及 ABA 问题解决方案
- AQS 原理，及各同步器基于 AQS 的实现方式
- 线程池使用方式、核心参数、工作原理，内含**手写线程池实现**
- 线程安全集合（ConcurrentHashMap），JDK 7 / JDK 8 底层原理对比
- ArrayList 线程不安全分析 & CopyOnWriteArrayList 写时复制原理
- 线程间通信、等待唤醒机制
- 等等......

## 核心笔记要点

- 线程安全本质：可见性、原子性、有序性
- synchronized 锁升级：无锁 → 偏向锁 → 轻量级锁 → 重量级锁
- ReentrantLock 可重入、可中断、公平 / 非公平、基于 AQS 实现
- CAS 无锁编程原理 + 原子类底层实现
- AQS 核心：state 状态 + 阻塞队列 + 模板方法模式
- 线程池：核心参数、执行流程、拒绝策略、自定义线程池
- ConcurrentHashMap：JDK7 分段锁 / JDK8 CAS + synchronized 锁头节点
- CopyOnWriteArrayList：写时复制、读写分离、弱一致性、无锁读

## 适用人群

- Java 后端开发者
- 准备面试的后端求职者
- 想要深入理解 JUC 并发容器原理的学习者