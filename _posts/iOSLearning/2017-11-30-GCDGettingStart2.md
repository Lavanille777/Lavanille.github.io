---
layout: post
title: 初识GCD其二
category: iOS开发
---


## 循环异步调度任务

当我们想要用多线程遍历处理一个数组时，希望数组中的每个元素可以并发处理，这时我们可以使用for循环+dispatch_async函数来达到目的。

但苹果为我们提供了更方便的解决方案，即:

```
        dispatch_apply(5, myDispatchQueue, ^(size_t index){
            NSLog(@"%zu", index);
        });
```

如此，便可以简单地用并发的方式遍历数组，第一个参数是要循环的次数。

用这种方式调度任务，如果是将任务加入并行队列，则是异步调度，如果加入的是串行队列，实际测试中系统并不会多开一条线程，相当于同步调度。

另外要注意的是dispatch_apply本身会阻塞当前线程等待任务全部执行完，因此推荐异步执行这个函数。

## 队列的挂起与恢复

```
//挂起队列  任务不再执行
dispatch_suspend(myDispatchQueue);
//恢复队列  从挂起的位置继续执行
dispatch_resume(myDispatchQueue);
```

## 使用信号量进行任务控制

当我们使用并发的方式对一个可变数组写数据时，很容易就会出现错误。

当然为了保证安全我们可以用串行队列来操作，但如果我们一定要并发，也要保证每次只有一个线程对可变数组写数据，这时我们就可以使用信号量来进行控制:

```
//生成一个值为1的信号量
        dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);
        
        dispatch_async(myDispatchQueue, ^{
           
            //当信号量为0时阻塞当前线程，大于等于1时就会返回并使信号量减1
            dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
            
            //我们要锁住的操作
            
            //使信号量加1
            dispatch_semaphore_signal(semaphore);
            
        });
```

从上述代码中我们可以想到，无论此时有多少个线程要进行操作，由于信号量初始化时值为1，因此只要第一个线程开始操作，信号量就会减为0，此时其他所有线程都会阻塞，而只有等这个线程操作结束时，信号量才会加1，其他的线程当中就会有一个得以继续，如此往复，便能达到每次只有一个线程能进入这段代码的目的。

## 使代码只执行一次

```
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^{
            //这一部分代码在程序中只被执行一次
        });
```

使用方法十分简单粗暴，多用于单例模式。

大家可能觉得

```
static BOOL initialized = NO;

if (!initialized){
	//初始化方法
}
```

这一段代码也能达到同样的目的。

但在多线程环境中，使用dispatch_once更加安全。
 



