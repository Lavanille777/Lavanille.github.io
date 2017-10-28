---
layout: post
title: 选择排序法
category: 算法
---

# 思路

选择排序法将一个待排数组分为已排序和未排序两部分，每次从待排序的部分中选出一个最小的数放在已排序部分的末尾，最终可以将整个数组排序完毕。

# 实现

因为比较简单，想了想也实在没什么好说的，先贴代码。

```
//选择排序
void selectionSort(int arr[], int n){

    for (int i = 0; i < n; i ++) {
        //默认最小值为当前的元素
        int minIndex = i;
        for (int j = i+1; j < n ; j ++) {
            //找到当前元素之后的数组中最小的元素
            if (arr[j] < arr[minIndex])
            {
                minIndex = j;
            }

        }
        //与当前元素交换位置
        swap( arr[i],arr[minIndex] );
    }

}

``` 

# 测试结果

在对10000个在[0，10000]的范围中的随机数进行排序

```
int main() {
    int n = 10000;
    int* arr = SortHelper::generateRandomArray(n, 0, 100);
    SortHelper::sortTimeCounter("selection sort", selectionSort, arr ,n );
    return 0;
}
```
打印出

```
selection sort : 0.109314s
```

## 本篇到此为止，希望这对你有帮助，如果有错误或是有需要补充的地方，望告知。


