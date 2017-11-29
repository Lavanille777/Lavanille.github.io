---
layout: post
title: 初识GCD其一
category: iOS开发
---
> GCD 全称 Grand Central Dispatch，目前似乎没有特别好的中文翻译，这个名字我个人认为取自纽约中央火车站（Grand Central Terminal），如果把线程比作一列一列的火车，那么GCD是个非常合适的名称。不管在什么语言中，多线程编程似乎都是进阶的难点，对于我这个还谈不上初级的iOS开发选手，也只能在这里简单归纳一下GCD的用法了。

## 认识线程（Thread）

首先要认识到，一个CPU一次只能执行一个命令，它是决不能“同时”执行多个命令的，我们将“CPU执行的命令列”称为线程，但是否一个CPU执行的命令列就是一条线程呢，我们往下看。

## 多线程处理（Multithreading）

在实际的程序中，任务的流水线不止一条，而是需要多条互相独立的线程并行工作，因此也就诞生了多线程编程技术，但在很多情况下计算机只有一个CPU，这种情况下的处理是将每个线程的CPU寄存器等信息（简称为“上下文”，即线程执行时的环境）复制一份存储在一个内存块中，当我们需要多条线程同时工作时，由于内存中保存了不同线程的上下文，因此可以系统可以通过每隔一段时间切换线程的方式来达到多线程的效果，当线程切换时，CPU也能正确还原该线程原本的环境，尽管CPU执行的依旧是一个命令列，但看上去就像是有多条线程一般。不过从根本上来讲，单个CPU的多线程仅仅只是在空间上并行了，时间上依然不是并行的，并且切换线程会增加时间开销，这种情况我们也称之为**“并发”**，不过在多个CPU环境下，就能达到真正的**“并行”**了。

## 多线程编程容易引起的问题

### 数据竞争

举个例子，当线程A读取对象A的同时，线程B正在对对象A进行写入，那么线程A最后得到的对象A的数据可能并不是预先期望的，因为它中途被线程B改变了。

### 死锁

死锁就是程序逻辑中的矛盾，就好比“只有吃饱了的饿汉才能来领面包”，正因为没有吃饱才希望能领到面包，结果领面包的条件是必须先吃饱，这样一来你永远也领不到面包，关于死锁的例子下面会以GCD为例详细讲解。

### 内存消耗

我们知道在一个核心需要处理多个线程时需要将上下文存放在内存中，当线程数过多时，会造成不必要的内存浪费。

## GCD的API

> 尽管多线程技术在底层实现中非常复杂，但apple为我们提供了一些简洁的API来进行多线程编程，下面就来一一介绍。

## Dispatch Queue

Dispatch Queue 可以理解为“线程调度队列”，它遵循队列的FIFO（先进先出）顺序，先追加的任务先执行，在GCD中有两种 Dispatch Queue ：

+ Serial Dispatch Queue ：当前任务处理结束才开始下一个任务。
+ Concurrent Dispatch Queue ：当前任务处理开始后立即开始下一个任务。

我们可以称之为串行队列和并行队列，其中并行队列需要同时处理多个任务，这就用到了多线程技术。

尽管用到了多线程，但也并不代表并行队列中的每个任务都独占一个线程，线程数是系统根据当前的状态决定的，并非是无限的，因此并不是所有的任务都会同时起步。

## 任务队列的获取方法其一：dispatch_queue_create

比如说我们想创建一个串行队列：

```
dispatch_queue_t serialDispatchQueue = dispatch_queue_create("com.example.gcd.serialDispatchQueue", NULL);
```

我们可以看到这个函数包含两个参数，其中第一个参数是一个非OC的字符串，它代表创建队列的名称，这个名称当然是随便取的，即使为NULL也是可以的，不过建议是像我们为自己的项目ID命名一样用逆序全程域名（ FQDN，Fully Qualified Domain Name ）为其命名，线程的名称会出现在程序的崩溃报告中，这有助于我们debug。

第二个参数则可以指定我们要生成的队列类型，如上，我们要得到一个串行队列，只需填入 NULL ，如果要得到并行队列，则需填入 DISPATCH_QUEUE_CONCURRENT 。

> **注意事项**
> 
> 在上面我们提到过在并行队列中，线程的数量由系统决定，因此往并行队列中加入大量任务也是安全的，并不会造成线程数量过多消耗内存的问题，但串行队列是独占一个线程的，也就是说我们生成多少个串行队列就会生成多少个线程，这时线程的数量是没有限制的，因此容易造成内存消耗过大的问题，我们尽可能将任务安排在并行队列中，串行队列只在为了避免数据竞争的情况下使用，并且要严格控制它的数量。

尽管 dispatch_queue_t 类型并不是一种OC对象类型，但iOS 6.0以后ARC也开始支持GCD了，可能是这本书编写的时间原因，书中提到我们需要手动释放队列，不过现在并不需要了。

## 任务队列的获取方法其二：Main Dispatch Queue/Global Dispatch Queue

事实上系统已经为我们准备了两个队列，Main Dispatch Queue和Global Dispatch Queue。

前者是在主线程中执行的队列，它是唯一的，当然也属于串行队列，我们通常将必须在主线程中处理的任务（必须实时反映给用户的，比如更新UI）放在这个队列里。

后者是系统提供的并行队列，当我们使用多个全局队列时可以为它们定义优先级，这样可以大致划分它们的执行顺序，它的优先级从高往低有以下四种：

+ High Priority (高优先级)
+ Default Priority (默认优先级)
+ Low Priority (低优先级)
+ Background Priority (后台优先级)

其中默认优先级也是我们通过dispatch_queue_create生成的队列的优先级。

毕竟是并行队列，无论哪一种优先级都不是实时的。

```

//获取主线程队列
        dispatch_queue_t mainDispatchQueue = dispatch_get_main_queue();
        
//获取全局队列
        dispatch_queue_t globalDispatchQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
        
```

也许有人会对全局队列的第二个参数感到疑惑，但它只是apple的保留参数，目前只能是0或NULL。

## 让任务在队列中异步调度

```
dispatch_async(mainDispatchQueue, ^{});
```
async代表异步，即在追加并调度任务时，（如果需要）允许系统另开一条线程进行处理。

第二个参数便是我们添加的任务，是一个block对象。

## 为队列变更优先级

我们知道使用dispatch_queue_create生成的队列的优先级均为默认优先级，当我们希望变更它的优先级时，就可以使用：

```
dispatch_set_target_queue(mySerialDispatchQueue, globalDispatchQueue);
```

这个函数将第一个参数的队列加入到了第二个队列中，可以看到前者是一个串行队列，后者是一个全局队列，如此一来，前者的优先级就会变更为后者的优先级。

我们知道当生成多个串行队列时，它们是并行执行的，这样就不利于管理，我们希望多个串行队列串行执行时，就可以使用：

```
dispatch_set_target_queue(mySerialDispatchQueue, serialDispatchQueue);
```

我们可以这样将多个串行队列加入到了一个串行队列中，这样它们就能按顺序执行。

## 延迟执行


```
dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW, 3ull * NSEC_PER_SEC);

dispatch_after(time, queue, ^{});
```

通过这个函数添加到队列的任务可以延迟执行，第一个参数为 dispatch_time_t 类型的变量，我们看它的创建方法，第一个参数指时间的起点，DISPATCH_TIME_NOW就是从当前开始，第二个参数则是一个 int64_t（无符号长整型）类型的数乘以 NSEC_PER_SEC ，这个数代表持续多少秒。

如上所示，当queue执行到这个任务时会延迟三秒再处理。

## 结束处理

有时我们希望在一个队列的全部任务结束时做一些处理，如果在串行队列中，我们只需将结束处理的任务加在队列的最后就行，但在并行队列中，任务结束的时间无从知晓，GCD依然为我们提供了一个用于处理队列结束的API：Dispatch Group

使用方法很简单：

```
dispatch_group_t group = dispatch_group_create();
dispatch_group_async(group, mainDispatchQueue, ^{});
```

如前，dispatch_group_async可以向group中添加一个在队列结束时执行的任务。

Dispatch Group 亦可进行group的等待计时：

```
long result = dispatch_group_wait(group, time);
        if (result == 0) {
            NSLog(@"group已结束");
        }
        else
        {
            NSLog(@"group未结束")
        }
//时间设为DISPATCH_TIME_FOREVER时dispatch_group_wait会在group结束前永久等待，因此result恒为0
long result = dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
        if (result == 0) {
            NSLog(@"group已结束");
        }
```

## 将串行任务插入并行队列

我们知道读写操作在并行队列中执行会产生各种意想不到的后果，小到数据错误，大到程序崩溃。

主要原因是写入操作发生在了不该发生的时候，如果仅仅只是读取操作，放在并行队列中是完全安全的。

但如果将所有的写入操作都放在串行队列中，未免会拖累程序的响应，因此apple提供了一种将写入操作安全追加到并行队列中的方法

```
dispatch_barrier_async(globalDispatchQueue, ^{});
```

已该种方式加入的任务，会等待在它之前加入的任务全部执行完后再执行，并且在它执行时，整个并行队列会进行串行处理，直到这个任务结束，并行处理恢复。

## 同步调度

在上面提到async方式是异步调度，那么同步调度理所当然就是

```
dispatch_sync(myDispatchQueue, ^{});
```

那么相应的，dispatch_sync只会在当前线程中进行追加操作，并且会等待当前任务结束才会进行下一次追加，因此使用dispatch_sync追加的任务无论是在串行队列还是在并行队列中都无法并发执行。

假如这里的myDispatchQueue是主线程队列，那么将发生死锁，原因是主线程执行到sync操作时会阻塞，等待sync操作结束。

而此时

+ sync结束的前提条件是block被追加到主队列中并执行结束。
+ block被追加到主队列并执行的前提条件是当前线程队列中的任务(也就是主线程中的sync)结束

二者无法互相满足，如此一来主线程发生死锁。

> GCD的内容还是比较多的，即使只是总结它的使用方法也需要两篇文章，以上所写的内容里也许还有模糊不清的地方，其实是因为我自己也没有完全弄清楚，但我已经尽量保证传达的意思是正确的，以后会逐渐完善。




