---
title: CUDA 04 - 同步
date: 2019-05-19 12:01:28
categories: CUDA
---
# 同步

<!--more-->

栅栏同步是一个原语, 在很多并行编程语言中都很常见. 在CUDA中, 同步可以在两个级别执行:

- 系统级: 等待主机和设备完成所有工作.
- 块级: 在设备执行过程中等待一个线程块中所有线程到达同一点.

对于主机来说, 由于需要CUDA API调用和所有点的内核启动不是同步的, cudaDeviceSynchonize函数可以用来阻塞主机应用程序, 直到所有CUDA操作(复制, 核函数等)完成:

```c
cudaError_t cudaDeviceSynchronize(void);
```

这个函数可能会从先前的异步CUDA操作返回错误, 因为在一个线程块中线程束以一个为定义的顺序被执行, CUDA提供了一个使用块局部栅栏来同步他们的执行的功能, 使用下述函数在内核中标记同步点:

```c
__device__ void __syncthreads(void);
```

当__syncthreads被调用时, 在同一个线程块中每个线程都必须等待直至该线程块中所有其他线程都已经达到这个同步点. 在栅栏之前所有线程产生的所有全局内存和共享内存访问, 将会在栅栏后对线程块中所有其他的线程可见. 该函数可以协调一个块中线程之间的通信, 但他强制线程束空闲, 从而可能对性能产生负面影响.

线程块中对线程可以通过共享内存和寄存器来共享数据. 当线程之间共享数据时, 要避免竞争条件. 竞争条件或危险, 是指多个线程无序地访问相同的内存位置. 例如, 当一个位置的无序读发生在写操作之后, 写后读竞争条件发生. 因为读写之间没有顺序, 所以读应该在写前还是在写后加载值是为定义的. 其他竞争条件的例子有读后写或写后写. 当线程块中的线程在逻辑上并行运行时, 在物理上并不是所有的线程都可以在同一时间上执行. 如果线程A试图读取由线程B在同步的线程数中写的数据, 若使用了适当的同步, 只需要知道线程B已经写完就可以了.

在不同块之间没有线程同步. 块间同步, 唯一安全的方法就是在每个内核执行结束端使用全局同步点, 也就是说, 在全局同步后, 终止当前的核函数, 开始执行新的核函数.

不同块中的线程不允许相互同步, 因此GPU可以以任意顺序执行块. 这使得CUDA程序在大规模并行GPU上是可扩展的.