## PV 操作实现多任务同步

一篇 naive 的短文。

PV 操作实际上指的是 [信号量](https://www.cnblogs.com/sinkinben/p/14087750.html) 的 2 种基本操作，在 POSIX 的实现中，P 操作对应 `sem_wait`，而 V 操作对应 `sem_post` 。

假设有 5 个进程/线程，其运行的先后次序必须要满足以下依赖关系：

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210201141613.png" style="width:40%;" />

问题：如何使用 PV 操作实现多线程/多进程的同步？

解决办法十分简单。首先给每条边定义相应的信号量 `s`，初始值为 0 ，然后边的出口需要 `V(s)`，边的入口需要 `P(s)` ，如下图所示。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210201142226.png" style="width:40%;" />

显然，上述过程就是一个简单的 BFS （或者说是拓扑排序？），最终可以轻易翻译成代码：

```c
sem_t a,b,c,d,e,f,g;
sem_init(a, 0, 0);
...
void S1()
{
    // do S1 job
    sem_post(&a);
}
void S2()
{
    sem_wait(&a);
    // do S2 job
    sem_post(&b, &c, &d);
}
// the other processes/threads is similar
```

以上基本知识足够做一道题了：https://leetcode-cn.com/problems/print-in-order/

