---
layout: post
title: 归并排序法（自顶向下）
category: 算法
---
> 从这里起的算法比之前的算法难度会有明显提升，我个人的学习方法是：
> 首先看视频/文章/书籍理解一遍。
> 照着别人的代码敲一遍。
> 用自己的理解写一遍博客。
> 再把代码默写一遍。
> 最好是没事的时候在脑子里再过一遍它的流程。
> 如此一来，尽管不能算是吃透了个算法，但至少能写的出来，并能说出整体的思想，各个部分的含义，我认为目前阶段有这种程度就足够了。

> 约翰·冯·诺伊曼在1945年首次提出了归并排序(merge sort)，自此以后，算法在科学领域的重要作用不断体现出来。



## 思路

> 归并排序法是一种O(nlogn)级别的高级排序算法，思路对于之前的O(n²)的算法而言要复杂很多，归并这个名称，在我个人的理解中，就是“在归来的过程中合并”，因此归并排序的过程并没有交换，而是合并，而且是发生在递归回来的过程中。

### 一句话解释

用一句话描述归并排序的过程。

首先将待排序数组进行不断进行二分，当分到每个部分都只剩一个元素时，那么每个子序列(此时只包含一个元素)就是有序的，继续保持有序不断合并，最后即可得到完整的有序数组。

### 初步解释

这么说可能并不容易理解，我们看一个实际的例子。

比如说我们有一个四个元素的待排数组8、6、5、4。

```
8654
```

将这个序列不断二分。

```
86 54

8 6 5 4
```

从左往右看，二分到最后，8和6这两个子序列都是有序的，现在将它们**在维持有序的情况下**合并为从小到大排序的有序数组即为"6 8",同样的，5和4将合并为"4 5"。

那么我们得到了

```
68 45
```

68和45两个子序列是有序的，现在我们将这两个子序列**在维持有序的情况下**合并，最终我们得到了

```
4568
```

### 进一步解释

这个过程中我们需要重点注意的两个问题就是

+ 如何二分。
+ 如何在维持有序的情况下合并。

那么第一个问题，二分，可以很容易的想到“递归”这个手段。

因为递归就是“递出，归来”这样一个过程，我们在递出的过程中二分，在归来的过程中合并，这样就是“归并”。

第二个问题，合并，用语言来描述这个过程会显得过于艰涩，我们不妨具体在实例中看看这个过程。

比如说现在有两组有序的子数组，我们要将它们合并成一个有序数组

> 待合并：**5**6 **4**8

这里使用了加粗表示当前选中的元素，可以看到我们选中了两个子数组的首个元素，然后将这两者进行比较。

4小于5，因此我们将4挑出来，放到新数组的首位，接着左边的子数组选中元素不变，右边的子数组选中下一个元素，注意这里我们使用了额外的数组来存放待排序元素。

> 已合并：4
> 
> 待合并：**5**6 4**8**

5小于8，因此我们挑出5，放到新数组的第二位，接着选中左边第二位，右边不变

> 已合并：4 5
> 
> 待合并：5**6** 4**8**

6小于8，挑出6，放在新数组第三位，左侧选中下一个元素（没有下一个元素了，但这是我们下一轮才要处理的事情），右侧不变。

> 已合并：4 5 6
> 
> 待合并：56 4**8**

发现左侧的标定点超出了子数组范围，我们知道左侧数组已经归并完成，因此直接将右侧元素按顺序全部填入数组即可。

> 已合并：4 5 6 8
> 
> 待合并：56 48

这样一来我们就大致的了解了归并排序的过程，接下来看具体实现。

## 实现

先贴代码

```
//合并过程
void __merge(int arr[], int l, int mid, int r) {

    //新建一个空数组。
    int temp[r-l+1];
    //将待合并的两个子数组装在里面
    for (int i = l; i <= r; i ++) {
        temp[i-l] = arr[i];
    }
    int i = l, j = mid + 1;
    for (int k = l; k <= r; k ++) {
        //如果左侧的子数组超过右边界，证明已经合并完成，将右侧剩余元素合并
        if(i > mid){
            arr[k] = temp[j-l];
            j ++;
        }//如果右侧的子数组超过右边界，证明已经合并完成，将左侧剩余元素合并
        else if(j > r){
            arr[k] = temp[i-l];
            i ++;
        }//如果左侧选中元素小于右侧选中元素，将左侧元素填入数组，选中下一个元素
        else if(temp[i-l] < temp[j-l]){
            arr[k] = temp[i-l];
            i ++;
        }//如果右侧选中元素小于左侧选中元素，将右侧元素填入数组，选中下一个元素
        else{
            arr[k] = temp[j-l];
            j ++;
        }

    }
}
//归并过程
void __mergeSort(int arr[], int l, int r) {
    //当左边界碰到右边界时，即代表二分到只剩一个元素了，递出过程结束
    if(l >= r){
        return;
    }
    int mid = (l + r)/2;
    //将左侧部分不断二分
    __mergeSort(arr, l, mid);
    //将右侧部分不断二分
    __mergeSort(arr, mid+1, r);
    //合并
    __merge(arr, l, mid, r);

}
//暴露给外界调用的方法
void mergeSort(int arr[], int n) {
    //由于归并排序在递归时左右边界是在变化的，因此这里我们将真正实现函数的参数列表改为带左右边界的。
    __mergeSort(arr, 0, n-1);

}

```

相信合并的过程并不难理解，大部分人（其实也包括我自己）可能还是不太好理解递归这个过程，并且这个过程实在是不容易用语言描述，还是要靠各位意会。

## 测试

### 随机数组

```
selection sort : 2.722508s
insertion sort2 : 1.744580s
merge sort : 0.010182s
```

对50000个数进行排序，可以看到归并排序的速度是远远快于上面两个O(n²)级别的算法的。

### 近乎有序的数组

```
selection sort : 2.726213s
insertion sort2 : 0.001114s
merge sort : 0.005417s
```

在这种情况下，归并排序依旧很快，但插入排序更快，这是因为插入排序在这种条件下复杂度是O(n)级别，而归并排序是O(nlogn)级别。

## 第一步优化

首先是合并这个位置，我们可以想象一下，并非所有的情况下都需要对子数组进行合并，比如说：

> 35 78

可以看到，左侧的最后一个元素比右侧的第一个元素要小，因此它们原本就是一个有序的数组，不需要比较合并，因此，我们可以在代码中添加一行判断。

```
//只有当左侧最后一个元素大于右侧第一个元素时才进行二分
if (arr[mid] > arr[mid+1]){
    __merge(arr, l, mid, r);
}
```

## 第二步优化

在和插入排序比较的过程中我们了解到，当数组近乎有序时，插入排序的效率高于归并排序，受这一点的启发，我们可以想象，当待排子数组的长度比较小的时候，它接近有序的概率就比较大一些，因此我们在递归到某种程度时不使用归并而改为插入排序，也可以提高归并排序的效率。

我们将这个较小的长度拟定为16（也许会有更好的数字）。

将递归出口修改为

```
//        //当左边界碰到右边界时，即代表二分到只剩一个元素了，递出过程结束
//        if (l >= r) {
//            return;
//        }
        //当待排子数组长度小于16时，改用插入排序
        if (r - l <= 15) {
            insertionSort2(arr, l, r);
            return;
        }
```

这里额外写了一个对具体范围进行的插入排序。

结合上一步优化，理论上可以有效提高归并排序的效率。

## 测试

### 随机数组

```
selection sort : 2.722539s
insertion sort2 : 1.770827s
merge sort : 0.008555s
```

可以看到在随机数组的排序中，优化已经得到一定体现。

### 近乎有序的数组

```
selection sort : 2.722905s
insertion sort2 : 0.001452s
merge sort : 0.001803s
```

而在近乎有序的数组排序中，优化后的归并排序已经可以与插入排序媲美，尽管理论上它无法达到O(n)级别。

## 本篇到此为止，希望这对你有帮助，如果有错误或是有需要补充的地方，望告知。














