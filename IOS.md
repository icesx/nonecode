Operate System
======

## PV

### 信号量（Semaphore）：

由一个值和一个指针组成，指针指向等待该信号量的进程。信号量的值表示相应资源的使用情况。信号量S≥0时，S表示可用资源的数量。

S初始数量代表可用资源数量。

当信号量S小于0时，其绝对值表示系统中因请求该类资源而被阻塞的进程数目.S大于0时表示可用的临界资源数。注意在不同情况下所表达的含义不一样。当等于0时，表示刚好用完。

 ### P操作（-1）

意味着请求分配一个资源，因此S的值减1；当S<0时，表示已经没有可用资源，S的绝对值表示当前等待该资源的进程数。请求者必须等待其他进程释放该类资源，才能继续运行。

###  V操作（+1）

意味着释放一个资源，因此S的值加1；若S<0，表示有某些进程正在等待该资源，因此要唤醒一个等待状态的进程，使之运行下去。
 *注意：信号量的值只能由PV操作来改变。*

### 解释

1. S大于0那就表示有临界资源可供使用，为什么不唤醒进程？
    S大于0的确表示有临界资源可供使用，也就是说这个时候没有进程被阻塞在这个资源上，所以不需要唤醒。
2. S小于0应该是说没有临界资源可供使用，为什么还要唤醒进程？

## 前驱图

