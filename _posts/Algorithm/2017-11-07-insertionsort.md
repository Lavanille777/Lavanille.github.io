---
layout: post
title: 插入排序法
category: 算法
---
## 思路

插入排序，打个比方就像是我们在抽扑克牌时，不断维护手中牌序的过程。

为了让手中的牌保持从小到大排序，我们会在已经排好序的牌中寻找合适的位置，然后将它插入。

而为了将这个过程写成程序，初步的思路是这样的。

如同选择排序，我们将整个数组分为已排序和待排序两个部分。

两层循环是必须的，外层循环负责扫描一遍整个数组，内层循环负责将扫描到的每个元素放到已排序部分中正确的位置。

1. 第一个元素默认为有序，因此我们从第二个元素开始扫描。
2. 将第二个元素与前一个进行比较，如果比前一个大，则进行位置交换。
3. 外层循环继续扫描下一个元素，以此类推，在内层循环中我们将不断的与前面的元素比较，交换，直到前一个元素比自己小，这样便可以不断的将待排部分中的元素加入到有序部分。

## 实现

思路已经充分明确，先贴代码

```
void insertionSort(int arr[], int n){

    for (int i = 1; i < n; i ++) {
        //从当前元素起不断向后交换直至当前元素挪到了正确的位置
        for (int j = i; j > 0 && arr[j] < arr[j-1] ; j --) {
            swap(arr[j], arr[j-1]);
        }
    }

}
```

## 测试结果

在10000个0到10000的元素下，与选择排序比较。

```
int main() {
    int n = 10000;
    int* arr = SortHelper::generateRandomArray(n, 0, 100);
    int* arr2 = SortHelper::copyIntArray(arr, n);
    SortHelper::sortTimeCounter("selection sort", selectionSort, arr ,n );
    SortHelper::sortTimeCounter("insertion sort", insertionSort, arr2 ,n );
    return 0;
}
```

输出:

```
selection sort : 0.110093s
insertion sort : 0.153060s
```

时间相近。

## 优化

注意在选择排序中，我们的交换是在外层循环中进行的，因为每一次交换都是将元素放到了正确的位置，因此只需要进行n次交换便可完成排序。

而在上面的插入排序中，交换是在内层循环中的，这显然是不必要的，我们可以在这个过程中做优化。

具体方法是，我们只需要将前面的元素覆盖当前元素，而不是交换，唯一的问题就是，需要事先保存待插入元素的值。

## 实现

```
void insertionSort2(int arr[], int n){

    for (int i = 1; i < n; i ++) {
        //使用临时变量保存当前的元素
        int t = arr[i];
        //j需要在循环后使用
        int j;
        //注意是和保存的元素比较，而不是和当前元素比较了
        for (j = i; j > 0 && arr[j-1] > t ; j --) {
            //从当前元素起不断向后覆盖直至找到了正确的位置
            arr[j] = arr[j-1];
        }
        //将保存的当前元素插入到正确位置
        arr[j] = t;
    }

}
```

## 测试结果

```
selection sort : 0.112045s
insertion sort : 0.151370s
insertion sort2 : 0.075784s
```

可以看到优化后的插入排序速度已经有了很大的提升，相比选择排序，它更快的原因是在内层循环中可以提前结束扫描。

这一点在对近乎有序的数组的排序中能体现出更大的优势。

## 对近乎有序的数组排序

```
int main() {
    int n = 10000;
    int* arr = SortHelper::generateNearlyOrderedArray(n, 10);
    int* arr2 = SortHelper::copyIntArray(arr, n);
    int* arr3 = SortHelper::copyIntArray(arr, n);
    SortHelper::sortTimeCounter("selection sort", selectionSort, arr ,n );
    SortHelper::sortTimeCounter("insertion sort", insertionSort, arr2 ,n );
    SortHelper::sortTimeCounter("insertion2 sort", insertionSort2, arr3 ,n );
    return 0;
}
```

输出

```
selection sort : 0.111469s
insertion sort : 0.000058s
insertion sort2 : 0.000051s
```

可以看出在近乎有序的情况下插入排序的速度远远快于选择排序，事实上在完全有序的情况下插入排序的时间复杂度是O(1)。

## 本篇到此为止，希望这对你有帮助，如果有错误或是有需要补充的地方，望告知。




