---
layout: post
title: 排序基础——准备
category: 算法
---

## 开始之前的话

边学习边记录，算法的基础是从排序开始，这一部分内容是我从慕课网上的liuyubobobo老师的视频中学习到的，基本上也就是学习他的视频内容，转化为自己的文字。代码使用C++。

## 排序学习的工具准备

在开始学习排序之前需要先准备一些辅助用的工具，新建一个sorthelper.h的头文件，将我们需要的工具都写在这里面。

### 需要用到的头文件

头文件与下面用到的方法的对应关系大家可以自己去查。。

```
#include <iostream>
#include <ctime>
#include <cassert>
#include <cstring>
```

### 随机数组生成器

排序学习的后期可能会测试一些非常长的数组，因此编写一个随机生成数组的方法很有必要。

先贴代码

```
//随机数组生成，n为数组长度，L与R分别代表随机数的最小值和最大值，返回数组的指针
int* generateRandomArray(int n, int L, int R){
        //断言，左边界必须小于等于右边界
        assert( L <= R );
        int *arr = new int[n];
        srand(time(NULL));

        for (int i = 0; i < n; ++i) {
        //用求模的方式确保随机数的范围
            arr[i] = rand() % (R - L + 1) + L;
        }

        return arr;
    }
```

经过查阅，说明一下以上代码中的细节。

+ 首先是time()函数，调用它会返回 自1970年1月1日00:00:00至系统现在这一刻所经过的秒数 ，返回值类型是time_t，是一个正整数，time()有两种调用方式，第一种是将一个变量地址填入括号，这样这个变量就会获得返回值，另一种方法是填入NULL，那么可以用a = time(NULL)的方式获得返回值。
+ srand()与rand()函数，这两个函数需要配合使用，调用前者需要传入一个参数，称为“种子”，这个函数会根据“种子”生成伪随机数序列，调用rand()则会从srand()生成的伪随机数序列中取出一个伪随机数，换言之，如果srand填入的种子不变，那么随机数每次都会按照固定的序列生成，因此要达到真正的随机，必须要使srand的种子是随机的，那么刚好time(NULL)返回的是一个无时无刻不在变化的数字，使得srand的种子不断变化。

### 数组有序判断

在排过序之后还需检查数组是否有序以证明排序是否成功，比较简单，就不做说明了。

```
bool isSorted(int arr[], int n){

        for (int i = 0; i < n - 1; i ++) {
            if (arr[i] > arr[i + 1])
                return false;
        }

        return true;
    }
```

### 排序时间测试

记录排序所用的时间是对排序算法性能衡量的重要指标。

先贴代码

```
//排序用时测试 sortname为排序方法的名称，sort是排序方法,arr为待排数组，n为数组长度
    void sortTimeCounter(string sortname, void(*sort)(int[], int), int arr[], int n) {
        //开始时间
        clock_t startTime = clock();
        sort(arr, n);
        //结束时间
        clock_t endTime = clock();
        //有序判断
        assert(isSorted(arr, n));
        //将时间单位转化为秒并打印
        cout << std::fixed << sortname << " : " << double(endTime - startTime) / CLOCKS_PER_SEC << "s" << endl;
    }
```

也并没有太多需要说明的地方，通过传入要使用的排序方法的函数指针，以及待排数组，得到排序所用时间。

+ 唯一的小细节是clock()函数返回的是clock_t类型，这是当前的时间，单位是计算机的时钟计时单元，并不是秒，需要除以CLOCKS_PER_SEC这个常量将单位转换为秒。

+ 另外，发现直接输出double类型控制台会自动使用科学计数法表示输出，很不直观，加入std::fixed可以转化为十进制输出。

### 数组打印

这个是我自己写的方法，liuyubobobo老师并没有写，因为我担心在自己写的时候出了什么问题，方便通过打印排除问题。

```
    //数组打印
    void printArray(int arr[], int n){
        for (int i = 0; i < n; i ++) {
            cout << arr[i] << " " ;
        }
    }
```



 


