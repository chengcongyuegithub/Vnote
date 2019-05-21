# InterruptedException
## 常用类
抛InterruptedException的代表方法有：

java.lang.Object 类的 wait 方法

java.lang.Thread 类的 sleep 方法

java.lang.Thread 类的 join 方法

```
，对于常见的ABC都有说明，这里也不多解释了，对于D答案我看没有说明，
我简单说一下，CyclicBarrier是一个屏障类，它的await方法可以简单的理解为：等待多个线程同时到达之后才能继续进行，在此之前它就是这些线程的屏障，线程不能继续进行，而对于失败的同步尝试，CyclicBarrier 使用了一种要么全部要么全不 (all-or-none) 的破坏模式：如果因为中断、失败或者超时等原因，导致线程过早地离开了屏障点，那么在该屏障点等待的其他所有线程也将通过 BrokenBarrierException（如果它们几乎同时被中断，则用 interruptedException）以反常的方式离开。
因此它被中断也是可以抛出interruptedException的，如果还是不清楚，查看一下JavaAPI，对于这个类介绍的清清楚楚。
```