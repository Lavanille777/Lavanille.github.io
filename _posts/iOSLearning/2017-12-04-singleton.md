---
layout: post
title: 单例模式
category: iOS开发
--- 

> 所谓模式（pattern）就是编写代码时的套路，使用模式可以在各个方面改良你的的程序。

## 单例模式

使用场景：

+ 某个对象只需要生成一次
+ 它需要大量复用

例如：数据源连接对象

## 如何实现

### 重写allocWithZone

我们的目的是让一个对象的实例生成从始至终都只执行一次，我们知道一个实例的生成方法是：

```
[[MyObject alloc]init];
```


其中alloc方法就是申请内存空间创建实例的方法，因此我们要重写这个方法。。。。吗？

不是。

alloc方法实际上还默认调用了allocWithZone，还记得ARC中我们提到过的NSZone吗，尽管这个方法已经被弃用了，但仍然可以调用，如果有人直接调用allocWithZone，那么我们重写alloc的目的就无法达到，因此我们要重写的是allocWithZone。

### 重写copy

要使用拷贝必须要重写copy方法，此时我们也要保证返回的是唯一创建的实例。

### 提供一个供外部获取实例的方法

它必须是一个类方法，只需在这个方法中创建实例并返回。

## 实现

```
+ (instancetype)allocWithZone:(struct _NSZone *)zone
{
    //只会运行一次，且是线程安全的
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
            _myObject = [super allocWithZone:zone];
            //这里可以写初始化方法
    });
    return _myObject;
}

- (id)copyWithZone:(NSZone *)zone{
    return _myObject;
}

- (id)mutableCopyWithZone:(NSZone *)zone
{
    return _myObject;
}

+ (MyObject *)shareMyObject
{
	return [[self alloc]init];
}
```

网上的有些代码示例中看到了在dispatch_once中用懒加载的情况，我实在是想不出这里使用懒加载的理由，于是就没有加上，大部分的写法是不用懒加载的，如果是用互斥锁的写法就需要懒加载保证只执行一次，而dispatch_once原本就有这些作用。

## 本篇到此为止，希望这对你有帮助，如果有错误或是有需要补充的地方，望告知。

