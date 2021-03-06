---
title: Operating System 05 - 进程通信
date: 2019-04-26 21:42:12
categories: Operating System
---
# 进程通信

<!--more-->

## 进程通信范式

1. 持锁共享内存
    
    在这种范式下, 两个或者多个进程可以同时读写一块或者多块常规内存区域. 有时进程需要在这些内存区域上执行一些具有原子性的操作序列, 其他进程在操作完成前不得访问这些区域, 这就需要一种令该进程阻止其他进程访问这些区域的方法, 也就是锁.
    
    锁的实现需要内存系统的支持, 一般是由硬件以特殊指令的形式提供支持. 使用锁的进程之间必须通力合作: 所有进程必须先获取锁才能访问共享内存区域, 访问结束后还需要将锁归还给其他进程使用. 操作系统或编程语言分别以系统调用或语言构件的形式提供了信号量, 监视器和互斥量等以基础锁为基础的高级构件, 用以确保锁的请求和释放的正确性. 尽管借助这些构件我们可以规避最棘手的问题, 但仍然难以克服诸多锁的缺点:

     1. 锁的开销过大.
     2. 锁是内存系统中的竞争热点.
     3. 出错的进程可能正处于加锁状态, 无法释放锁.
     4. 难以调试锁的问题.
     5. 随着程序规模和进程数量的增长, 潜在的死锁问题很难避免.

    这种通信范式最好只出现在底层编程场合, 例如操作系统内核中. 但是在目前流行的大部分编程语言中都能看到锁的身影, 因为其本身实现并不复杂, 也不会对编程模型产生影响.

2. 软件事务性内存(STM, Software Transactional Memory)

    我们在Haskell的GHC的实现和基于JVM的Clojure语言中看到这种机制. STM将内存当作传统数据库, 用事务决定何时写入什么内容. 这种实现是用乐观方式来规避锁: 将一组读写访问视作单个操作, 如果两个进程试图同时访问共享区域, 则各自启动一个事务, 最终只会有一个事务成功. 另一个进程会得知事务的失败, 并应该在检查共享区域的新内容后重试. 该模型较为直观, 不需要通过锁的占用释放来访问共享区域.

    STM的缺点在于:

    1. 需要对失败事务进行处理.
    2. 事物本身有显著开销, 对于数量较多的并发访问性能差.
    3. 在确定进程成功前, 需要额外的内存来存放试图写入的数据.

    从编程人员角度而言, STM对可控性高于使用锁, 其本质上是持有锁的共享内存的变体, 他在操作系统层面的操作要甚于编程层面. 理想情况下, 系统应该像支持虚拟内存一样对STM提供硬件支持.

3. Future, Promise及同类机制

    所谓future或promise, 这个概念还有其他形式: 在E和MultiLisp等语言都可以看到他对身影. 类似对还有Id和Glasgow Haskell中的I-var和M-var, Concurrent Prolog中的
并发逻辑变量, 以及Oz中的数据流变量.

    其基本思路是: 每个future代表一个被外包到其他进程的计算结果, 该进程可能跑到别的CPU甚至是别的主机上. Future可以像其他对象一样被传递, 但是无法在计算完成之前读取结果, 必须等待计算完成. 这种方法虽然概念简单, 简化了并发系统中的数据传递, 但也使程序在远端进程故障和网络故障面前v变得脆弱, 计算结果尚未就绪而连接又不幸断开, 试图访问promise的值的代码便会无所适从.

4. 消息传递

    消息传递意味这接收进程实际上获取了一份独立的数据副本, 发送方感知不到接收方对副本所做的任何操作. 向发送方回传信息的唯一途径就是反向发送另一条消息. 因此我们可知, 无论收发双方是在一台及其还是受网络隔离, 都能以相同的方式通信.

## 通信方式

1. 管道
   
   管道是通过pipe函数创建, fd[0]用于读, fd[1]用于写.

   ```c
   #include <unistd.h>
   int pipe(int fd[2]);
   ```

   具有以下限制:

   1. 只支持半双工通信(单向交替传输)
   2. 只能在父子进程中使用

2. FIFO

    也被称为命名管道, 可以不止在父子进程中使用.

    ```c
    #include <sys/stat.h>
    int mkfifo(const char* path, mode_t mode);
    int mkfifioat(int fd, const char* path, mode_t mode);
    ```

    FIFO常用于C/S模型, FIFO作为汇聚点, 在C/S进程时间传递消息.

3. 消息队列

    相比较FIFO, 消息队列有以下优点:

    1. 消息队列可以独立于读写进程存在, 从而避免了FIFO中同步管道的打开和关闭时可能产生的困难.
    2. 避免了FIFO的同步阻塞, 不需要进程提供同步方法.
    3. 读进程可以根据消息类型有选择的接收消息, 而不像FIFO那样只能默认的接收.

4. 信号量

    他是一个计数器, 用于为多个进程提供共享数据对象的访问.

5. 共享存储

    允许多个进程共享一个给定的存储区, 因为数据不需要在进程之间复制, 所以这是一种最快的IPC.

    需要使用信号量来互斥的对共享存储进行访问.

    多个进程可以将同一个文件映射到他们的地址空间从而实现共享内存. 另外XSI共享内存不是使用文件, 而是使用内存的匿名段.

6. 套接字

    和其他通信机制不同的是, 他可用于不同机器间的进程通信.
